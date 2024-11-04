---
title: Higher-Order Components in React Hooks era
date: 2024-11-04 21:16:49
categories:
  - Studies
  - React
  - Developer Way React
#tags:
---
HOC는 컴포넌트를 인자로 받아서 다른 버전의 컴포넌트를 반환하는 컴포넌트 함수입니다.

```tsx
// accept a Component as an argument
const withSomeLogic = (Component) => {
  // do something

  // return a component that renders the component from the argument
  return (props) => <Component {...props} />;
};
```

hook이 알려지기 전에 HOC는 context에 접근하거나 외부 데이터 subscription을 구현할 때 자주 사용되었습니다.

hook만으로 구현할 수 없는 HOC만의 3가지 기능들을 정리하면 다음과 같습니다😎

## Enhancing callbacks and React lifecycle events

컴포넌트는 원래 정의한대로 사용하되 플러그인을 설치하여 기능을 추가하는듯한 효과를 줄 수 있습니다.

```tsx
type Base = { onClick: () => void };

// just a function that accepts Component as an argument
export const withLoggingOnClick = <TProps extends Base>(
  Component: ComponentType<TProps>
) => {
  return (props: TProps & { logText: string }) => {
    const onClick = () => {
      console.log(props.logText);
      // don't forget to call onClick that is coming from props!
      // we're overriding it below
      props.onClick();
    };

    // return original component with all the props
    // and overriding onClick with our own callback
    return <Component {...props} onClick={onClick} />;
  };
};
```

```tsx
const ButtonWithLoggingOnClickWithProps = withLoggingOnClick(SimpleButton);

const Page = () => {
  return (
    <ButtonWithLoggingOnClickWithProps
      onClick={onClickCallback}
      logText="this is Page button"
    >
      Click me
    </ButtonWithLoggingOnClickWithProps>
  );
};
```

특정 컴포넌트가 mount되었을 때 수행할 로직도 HOC로 구현할 수 있습니다.

```tsx
export const withLoggingOnMount = <TProps extends unknown>(
  Component: ComponentType<TProps>
) => {
  return (props: TProps) => {
    // no more overriding onClick, just adding normal useEffect
    useEffect(() => {
      console.log("log on mount");
    }, []);

    // just passing props intact
    return <Component {...props} />;
  };
};
```

## Intercepting DOM Events

동일한 DOM Event의 핸들링 로직을 여러 컴포넌트에서 사용하려는 경우, 위의 경우와 비슷하게 구현할 수 있습니다.

```tsx
export const withSupressKeyPress = <TProps extends unknown>(
  Component: ComponentType<TProps>
) => {
  return (props: TProps) => {
    const onKeyPress = (event) => {
      event.stopPropagation();
    };

    return (
      <div onKeyPress={onKeyPress}>
        <Component {...props} />
      </div>
    );
  };
};
```

```tsx
const ModalWithSupressedKeyPress = withSupressKeyPress(Modal);
```

## Context selctors

임의의 컴포넌트로 하여금 특정 context의 value만을 받아서 사용할 수 있는 context selector 컴포넌트를 구현할 수 있습니다.

```tsx
export const withFormIdSelector = <TProps extends unknown>(
  Component: ComponentType<TProps>
) => {
  const MemoisedComponent = React.memo(Component) as ComponentType<
    TProps & { formId: string }
  >;

  return (props: TProps) => {
    const { id } = useFormContext();

    return <MemoisedComponent {...props} formId={id} />;
  };
};
```

위에서 MemoisedComponent는 context vaule 중 하나인 id만 변할 경우에만 리렌더링되도록 구현된 것을 알 수 있습니다.

```jsx
// formId prop here is injected by the higher-order component below
const CountriesWithFormId = ({ formId }: { formId: string }) => {
  console.log("Countries with selector re-render");
  return (
     <-- code is the same as before -->
  );
};
```

```jsx
const CountriesWithFormIdSelector = withFormIdSelector(CountriesWithFormId);

const Form = () => {
  return (
    <form css={pageCss}>
      <Name />
      <CountriesWithFormIdSelector />
    </form>
  );
};
```
