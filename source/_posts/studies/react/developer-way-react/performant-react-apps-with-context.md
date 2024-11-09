---
title: How to write performant React apps with Context
date: 2024-11-04 21:18:32
categories:
  - Studies
  - React
  - Developer Way React
#tags:
---
자식 컴포넌트로 부모 컴포넌트의 상태를 조작할 수 있는 함수를 props로 전달하면 전혀 상관없는 자식 컴포넌트들의 리렌더링을 일으킬 수 있습니다.

![Props Re-renders Flow.png](/images/props_re_renders_flow.png)

위와 같은 form에 context를 적용하면 상위 컴포넌트에서 form 상태를 관리하면서 props없이 필요한 상태값과 api 함수들을 전달할 수 있습니다.

```tsx
type State = {
  name: string;
  country: Country;
  discount: number;
};

type Context = {
  state: State;
  onNameChange: (name: string) => void;
  onCountryChange: (name: Country) => void;
  onDiscountChange: (price: number) => void;
  onSave: () => void;
};

const FormContext = createContext<Context>({} as Context);
```

```tsx
export const FormDataProvider = ({ children }: { children: ReactNode }) => {
  const [state, setState] = useState<State>({} as State);

  const value = useMemo(() => {
    const onSave = () => {
      // send the request to the backend here
    };

    const onDiscountChange = (discount: number) => {
      setState({ ...state, discount });
    };

    const onNameChange = (name: string) => {
      setState({ ...state, name });
    };

    const onCountryChange = (country: Country) => {
      setState({ ...state, country });
    };

    return {
      state,
      onSave,
      onDiscountChange,
      onNameChange,
      onCountryChange,
    };
  }, [state]);

  return <FormContext.Provider value={value}>{children}</FormContext.Provider>;
};
```

![Context Data Flow.png](/images/context_data_flow.png)

하지만 위와 같이 구현하면 onDiscountChange, onNameChange, onCountryChange 함수만 사용하는 context consumer의 경우, state를 사용하는 context consumer에 의해서 context value가 변했을 때 덩달아 리렌더링이 일어날 수 있습니다.

그래서 입력필드별 상태값을 전달하는 context와 api 함수를 전달하는 context를 따로 분리할 필요가 있습니다. 여기서 api의 정의는 state에 의존적이므로 state를 기준으로 memoization으로 처리합니다.

```tsx
type State = {
  name: string;
  country: Country;
  discount: number;
};

type API = {
  onNameChange: (name: string) => void;
  onCountryChange: (name: Country) => void;
  onDiscountChange: (price: number) => void;
  onSave: () => void;
};

const FormDataContext = createContext<State>({} as State);
const FormAPIContext = createContext<API>({} as API);
```

```jsx
const FormProvider = () => {
  // state logic

  const api = useMemo(() => {
    const onDiscountChange = (discount: number) => {
      // this is why we still need state here - in order to update it
      setState({ ...state, discount });
    };

    // all other callbacks

    return { onSave, onDiscountChange, onNameChange, onCountryChange };
    // still have state as a dependency
  }, [state]);

  return (
    <FormAPIContext.Provider value={api}>
      <FormDataContext.Provider value={state}>
        {children}
      </FormDataContext.Provider>
    </FormAPIContext.Provider>
  );
};
```

위와 같은 context는 [reducer](../../learn-react/extracting-state-logic-into-a-reducer.md)를 사용해서도 구현할 수 있습니다.

```tsx
type Actions =
  | { type: "updateName"; name: string }
  | { type: "updateCountry"; country: Country }
  | { type: "updateDiscount"; discount: number };

const reducer = (state: State, action: Actions): State => {
  switch (action.type) {
    case "updateName":
      return { ...state, name: action.name };
    case "updateDiscount":
      return { ...state, discount: action.discount };
    case "updateCountry":
      return { ...state, country: action.country };
  }
};
```

```tsx
export const FormProvider = ({ children }: { children: ReactNode }) => {
  const [state, dispatch] = useReducer(reducer, {} as State);

  const api = useMemo(() => {
    const onSave = () => {
      // send the request to the backend here
    };

    const onDiscountChange = (discount: number) => {
      dispatch({ type: "updateDiscount", discount });
    };

    const onNameChange = (name: string) => {
      dispatch({ type: "updateName", name });
    };

    const onCountryChange = (country: Country) => {
      dispatch({ type: "updateCountry", country });
    };

    return { onSave, onDiscountChange, onNameChange, onCountryChange };
    // no more dependency on state! The api value will stay the same
  }, []);

  return (
    <FormAPIContext.Provider value={api}>
      <FormDataContext.Provider value={state}>
        {children}
      </FormDataContext.Provider>
    </FormAPIContext.Provider>
  );
};
```

위에서 state를 입력필드별로 더 세부적으로 나누면 다른 입력필드의 리렌더링을 막을 수 있습니다.
