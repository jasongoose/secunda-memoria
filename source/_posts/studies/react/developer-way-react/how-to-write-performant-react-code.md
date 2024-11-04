---
title: How to write performant React code, rules, patterns, do's and don'ts
date: 2024-11-04 21:17:30
categories:
  - Studies
  - React
  - Developer Way React
#tags:
---
## 규칙 #1

> 자식 컴포넌트의 리렌더링을 막겠다고 props로 전달한 inline function들을 useCallback으로 wrapping하지 않는다.

React 성능문제 사례들의 90%는 “불필요하고 빈번한 리렌더링”이라고 합니다.

보통 자식 컴포넌트 props로 전달하는 inline function들은 부모 리렌더링으로 새로 생성되므로 자식 컴포넌트의 리렌더링을 방지하려면 아래와 같이 `useCallback` 으로 재정의를 막아야 한다는 인식이 있습니다.

```tsx
export const Page = ({ countries }: { countries: Country[] }) => {
  // ... same as before

  const onCountryChanged = useCallback((c) => setSelectedCountry(c), []);
  const onCountrySaved = useCallback(
    () => setSavedCountry(selectedCountry),
    []
  );

  return (
    <>
      ...
      <CountriesList onCountryChanged={onCountryChange} />
      <SelectedCountry onCountrySaved={onCountrySaved} />
      ...
    </>
  );
};
```

하지만 위와 같이 작성해도 onCountryChanged, onCountrySaved 함수는 그대로 유지되지만 부모인 Page 컴포넌트가 리렌더링되면 자식인 CountriesList, SelectedCountry 컴포넌트들도 같이 리렌더링됩니다.

## 규칙 #2

> 어떤 컴포넌트가 상태를 관리한다면 자식 컴포넌트의 불필요한 리렌더링을 방지하도록 props로 전달되는 값들로 memoization 처리를 해준다.

그래서 의도한대로 구현하려면 아래와 같이 자식 컴포넌트 자체를 `useMemo` 로 memoise 처리하면 됩니다.

```tsx
export const Page = ({ countries }: { countries: Country[] }) => {
  const [selectedCountry, setSelectedCountry] = useState<Country>(countries[0]);
  const [savedCountry, setSavedCountry] = useState<Country>(countries[0]);

  const list = useMemo(() => {
    return (
      <CountriesList
        countries={countries}
        onCountryChanged={setSelectedCountry}
        savedCountry={savedCountry}
      />
    );
  }, [savedCountry, countries]);

  const selected = useMemo(() => {
    return (
      <SelectedCountry
        country={selectedCountry}
        onCountrySaved={() => setSavedCountry(selectedCountry)}
      />
    );
  }, [selectedCountry]);

  return (
    <>
      <h1>Country settings</h1>
      <div css={contentCss}>
        {list}
        {selected}
      </div>
    </>
  );
};
```

`useMemo` hook은 보통 부모 컴포넌트의 리렌더링으로부터 벗어나도록 도와주는데, 2번째 인자인 dependency list에 부모 컴포넌트의 리렌더링을 일으키는 값을 전달하면 적용효과가 사라지므로 유의해야 합니다.

## 규칙 #3

> 절대로 컴포넌트 내부에 다른 컴포넌트를 정의하지 않는다.

list rendering을 구현할 때 item 컴포넌트를 list 컴포넌트 내부에 정의하면 list 컴포넌트의 리렌더링마다 item 컴포넌트는 새롭게 생성되어 mount됩니다.

```tsx
const CountriesList = ({ countries }: { countries: Country[] }) => {
  const Item = ({ country }: { country: Country }) => {
    useEffect(() => {
      console.log("Mounted!");
    }, []);
    console.log("Render");
    return <div>{country.name}</div>;
  };

  return (
    <>
      {countries.map((country) => (
        <Item country={country} />
      ))}
    </>
  );
};
```

그럼 item 컴포넌트는 list 컴포넌트 외부에 정의하면 됩니다.

```tsx
type ItemProps = {
  country: Country;
  savedCountry: Country;
  onItemClick: () => void;
};

// turned out savedCountry and onItemClick were also used
// but it was not obvious at all in the previous implementation
const Item = ({ country, savedCountry, onItemClick }: ItemProps) => {
  // ... same code
};

export const CountriesList = ({
  countries,
  onCountryChanged,
  savedCountry,
}: CountriesListProps) => {
  return (
    <div>
      {countries.map((country) => (
        <Item
          country={country}
          key={country.id}
          savedCountry={savedCountry}
          onItemClick={() => onCountryChanged(country)}
        />
      ))}
    </div>
  );
};
```

React에서 컴포넌트를 mount하는 연산은 일반적인 JS 연산에 비해서 방대한 리소스가 필요하기 때문에 위와 같은 현상은 거의 없도록 구현해야 합니다.

## 규칙 #4

> context를 사용할 때, context value가 객체라면 memoization 처리를 한다.

Theme을 적용하기 위해서 자주 활용되는 context의 경우, context provider에서 전달한 context value가 변할 때마다 해당 context consumer인 컴포넌트는 리렌더링되는 성질이 있습니다.

그래서 context value가 변해도 당장 리렌더링이 불필요한 consumer들도 강제로 리렌더링을 수행해야할 수도 있는데, 특히 context value가 객체인 경우에 신경써야 하는 부분입니다.

```tsx
export const Page = ({ countries }: { countries: Country[] }) => {
  // everything else stays the same

  // memoised list is still memoised
  const list = useMemo(
    () => <CountriesList countries={countries} />,
    [countries]
  );

  return (
    <ThemeContext.Provider value={{ mode }}>// same</ThemeContext.Provider>
  );
};
```

```tsx
const Item = ({ country }: { country: Country }) => {
  const theme = useTheme();
  console.log("render");
  return <div>{country.name}</div>;
};
```

그래서 context value가 객체인 경우, 아래와 같은 memoization이 필요합니다.

```jsx
export const Page = ({ countries }: { countries: Country[] }) => {
  // everything else stays the same

  // memoising the object!
  const theme = useMemo(() => ({ mode }), [mode]);

  return <ThemeContext.Provider value={theme}>// same</ThemeContext.Provider>;
};
```

React에서 컴포넌트가 리렌더링되는 경우는 총 3가지 입니다.

- 컴포넌트의 state가 변하는 경우
- 부모 컴포넌트가 리렌더링되는 경우
- context value가 변해서 context consumer가 리렌더링되는 경우

## 규칙 #5

마지막으로 `useCallback` 은 아래와 같은 상황에서 사용하면 됩니다.

- inline function을 props로 받는 컴포넌트가 `React.memo` 로 memoize된 경우

  ```jsx
  const MemoisedItem = React.memo(Item);

  const List = () => {
    // this HAS TO be memoised, otherwise `React.memo` for the Item is useless
    const onClick = () => {
      console.log("click!");
    };

    return <MemoisedItem onClick={onClick} country="Austria" />;
  };
  ```

- props로 전달한 inline function이 컴포넌트 내부에서 `useMemo` , `useCallback` , `useEffect` 의 dependency인 경우

  ```jsx
  const Item = ({ onClick }) => {
    useEffect(() => {
      // some heavy calculation here
      const data = ...
      onClick(data);

    // if onClick is not memoised, this will be triggered on every single render
    }, [onClick])
    return <div>something</div>
  }

  const List = () => {
    // this HAS TO be memoised, otherwise `useEffect` in Item above
    // will be triggered on every single re-render
    const onClick = () => {console.log('click!')};

    return <Item onClick={onClick} country="Austria" />
  }
  ```
