---
title: The mystery of React Element, children, parents and re-renders
date: 2024-11-04 21:17:46
categories:
  - Studies
  - React
  - Developer Way React
#tags:
---
React로 개발하면서 생길 수 있는 여러 의문들과 그 답들을 정리해봅니다🤓

## 1. children은 왜 리렌더링이 되지 않는가?

```jsx
const MovingComponent = ({ children }) => {
  const [state, setState] = useState({ x: 100, y: 100 });

  return (
    <div
      onMouseMove={(e) => setState({ x: e.clientX - 20, y: e.clientY - 20 })}
      style={{ left: state.x, top: state.y }}
    >
      // children now will not be re-rendered
      {children}
    </div>
  );
};
```

```jsx
const SomeOutsideComponent = () => {
  return (
    <MovingComponent>
      <ChildComponent />
    </MovingComponent>
  );
};
```

`children` 자체는 props이기에 MovingComponent의 상태변화로 리렌더링되어도 props 자체는 변하지 않으므로 ChildComponent는 리렌더링되지 않습니다.

## 2. render Fn으로 전달된 children은 왜 리렌더링이 되는가?

```jsx
const MovingComponent = ({ children }) => {
  ...
  return (
    <div ...// callbacks same as before
    >
      // children as render function with some data
      // data doesn't depend on the changed state!
      {children({ data: 'something' })}
    </div>
  );
};

const SomeOutsideComponent = () => {
  return (
    <MovingComponent>
      // ChildComponent re-renders when state in MovingComponent changes!
      // even if it doesn't use the data that is passed from it
      {() => <ChildComponent />}
    </MovingComponent>
  )
}
```

마찬가지로 `children`으로 전달된 [render Fn](./react_component_as_prop.md) 자체는 변하지 않지만 반환되는 ReactElement 객체는 새로 생성되므로 MovingComponent 렌더링 시, `children`이 위치한 slot에 ChildComponent가 리렌더링됩니다.

ReactElement의 재생성도 컴포넌트의 리렌더링으로 이어집니다.

컴포넌트로부터 반환된 ReactElement 자체는 그저 Render Tree에서 존재할 node 정보만을 담는 객체이고, 컴포넌트 life cycle이 시작하려면 ReactElement가 return문에 포함되어 반환되어야 합니다.

```jsx
const Parent = () => {
  // will just sit there idly
  const child = <Child />;

  return <div />;
};

// or

const Parent = () => {
  // render of Child will be triggered when Parent re-renders
  // since it's included in the return
  const child = <Child />;

  return <div>{child}</div>;
};
```

## 3. React.memo를 적용한 컴포넌트가 왜 리렌더링되는가?

```jsx
// wrapping MovingComponent in memo to prevent it from re-rendering
const MovingComponentMemo = React.memo(MovingComponent);

const SomeOutsideComponent = () => {
  // trigger re-renders here with state
  const [state, setState] = useState();

  return (
    <MovingComponentMemo>
      <!-- ChildComponent will still re-render when SomeOutsideComponent re-renders -->
      <ChildComponent />
    </MovingComponentMemo>
  )
}
```

MovingComponentMemo는 `children` props가 변하면 리렌더링됩니다.

위에서 SomeOutsideComponent가 리렌더링되면 ChildComponent가 새로 생성되므로 MovingComponentMemo가 리렌더링됩니다.

## 4. children으로 useCallback을 적용한 render Fn을 전달하면 왜 리렌더링되는가?

```jsx
const SomeOutsideComponent = () => {
  // trigger re-renders here with state
  const [state, setState] = useState();

  // trying to prevent ChildComponent from re-rendering by memoising render function. Won't work!
  const child = useCallback(() => <ChildComponent />, []);

  return (
    <MovingComponent>
      <!-- Memoized render function. Didn't help with re-renders though -->
      {child}
    </MovingComponent>
  )
}
```

render Fn은 자체는 memoise되었지만 반환되는 ChildComponent라는 ReactElement는 새로 생성되므로 리렌더링이 일어납니다.

ChildComponent의 리렌더링을 방지하려면 (1) MovingComponent를 `React.memo` 로 memoise하거나, (2) useCallback을 제거하고 ChildComponent 자체를 `React.memo` 로 memoise하면 됩니다.
