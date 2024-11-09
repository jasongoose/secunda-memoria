---
title: How to handle errors in React, full guide
date: 2024-11-04 21:16:41
categories:
  - Studies
  - React
  - Developer Way React
#tags:
---
React에서 특히 에러 핸들링이 중요한 이유는 매우 간단합니다.

왜냐하면 React v16 이상부터는 React life cycle 중 문제가 생기면 앱 전체를 그냥 unmount하기 때문입니다😱

## Simple try/catch in React: how to and caveats

try/catch문에만 의존하는 컴포넌트 내부 에러 핸들링의 한계점들을 정리하면 다음과 같습니다.

### Limitation1: you will have trouble with useEffect hook

`useEffect`들에서 발생한 에러를 한번에 잡겠다고 아래와 같이 try 블럭 안에 `useEffect` 들을 나열하면 안됩니다.

```jsx
try {
  useEffect(() => {
    throw new Error("Hulk smash!");
  }, []);
} catch (e) {
  // useEffect throws, but this will never be called
}
```

`useEffect` 는 **컴포넌트 렌더링 중이 아닌 현재 상태가 DOM에 반영된 commit 이후 시점에 실행하는 비동기 연산**이기 때문에 위 코드에서 throw가 발생하지 않습니다.

따라서 try/catch문은 `useEffect` hook별로 적용해야 합니다.

```jsx
useEffect(() => {
  try {
    throw new Error("Hulk smash!");
  } catch (e) {
    // this one will be caught
  }
}, []);
```

### Limitation2: children components

부모 컴포넌트의 try/catch문에서는 자식 컴포넌트에서 throw된 에러를 처리할 수 없습니다.

```jsx
const Component = () => {
  try {
    return <Child />;
  } catch (e) {
    // still useless for catching errors inside Child component, won't be triggered
  }
};

// 또는

const Component = () => {
  let child;

  try {
    child = <Child />;
  } catch (e) {
    // useless for catching errors inside Child component, won't be triggered
  }

  return child;
};
```

위에서 `<Child />` 컴포넌트는 부모 컴포넌트에 의해서 return되어 렌더링된 것이 아니 그저 [ReactElement](../mystery)이기에 catch문으로 넘어가지 않습니다.

### Limitation3: setting state during render is a no-no

컴포넌트 렌더링 중에 try/catch문에서 상태를 업데이트하면 무한루프에 빠질 수 있습니다.

```jsx
const Component = () => {
  const [hasError, setHasError] = useState(false);

  try {
    doSomethingComplicated();
  } catch (e) {
    // don't do that! will cause infinite loop in case of an error
    setHasError(true);
  }
};
```

## React ErrorBoundary Component

React에서는 현재 컴포넌트와 자식 컴포넌트의 렌더링 중에 에러를 다룰 수 있는 API인 [ErrorBoundary](https://reactjs.org/docs/error-boundaries.html)를 제공합니다.

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    // initialize the error state
    this.state = { hasError: false };
  }

  // if an error happened, set the state to true
  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    // send error to somewhere here
    log(error, errorInfo);
  }

  render() {
    // if error happened, return a fallback component
    if (this.state.hasError) {
      return <>Oh no! Epic fail!</>;
    }

    return this.props.children;
  }
}
```

```jsx
const Component = () => {
  return (
    <ErrorBoundary fallback={<>Oh no! Do something!</>}>
      <SomeChildComponent />
      <AnotherChildComponent />
    </ErrorBoundary>
  );
};
```

단, `ErrorBoundary`는 컴포넌트 lifecycle 중에 발생한 에러만 처리할 수 있다는 한계점이 있습니다.

그래서 event handling, setTimeout, resolved promise 등의 콜백함수들의 에러 핸들링은 try/catch문을 사용하면 됩니다.

```jsx
const Component = () => {
  const [hasError, setHasError] = useState(false);

  // most of the errors in this component and in children will be caught by the ErrorBoundary

  const onClick = () => {
    try {
      // this error will be caught by catch
      throw new Error("Hulk smash!");
    } catch (e) {
      setHasError(true);
    }
  };

  if (hasError) return "something went wrong";

  return <button onClick={onClick}>click me</button>;
};

const ComponentWithBoundary = () => {
  return (
    <ErrorBoundary fallback={"Oh no! Something went wrong"}>
      <Component />
    </ErrorBoundary>
  );
};
```

## Catching async errors with ErrorBoundary

비동기 에러들을 try/catch문으로만 처리하려고 하면 별도의 상태로직들을 부모 컴포넌트에서 구현해야 하는 번거로움이 있습니다.

```jsx
const Component = ({ onError }) => {
  const onClick = () => {
    try {
      throw new Error("Hulk smash!");
    } catch (e) {
      // just call a prop instead of maintaining state here
      onError();
    }
  };

  return <button onClick={onClick}>click me</button>;
};

const ComponentWithBoundary = () => {
  const [hasError, setHasError] = useState();
  const fallback = "Oh no! Something went wrong";

  if (hasError) return fallback;

  return (
    <ErrorBoundary fallback={fallback}>
      <Component onError={() => setHasError(true)} />
    </ErrorBoundary>
  );
};
```

`ErrorBoundary`에서 비동기 에러들을 다룰려면 비동기 에러에 의해서 리렌더링이 일어나도록 만들면 됩니다!

```jsx
const Component = () => {
  // create some random state that we'll use to throw errors
  const [state, setState] = useState();

  const onClick = () => {
    try {
      // something bad happened
    } catch (e) {
      // trigger state update, with updater function as an argument
      setState(() => {
        // re-throw this error within the updater function
        // it will be triggered during state update
        throw e;
      });
    }
  };
};
```

위와 같이 구현하면 Component가 setState에 의해 리렌더링되면서 에러를 throw하여 `ErrorBoundary`에서 처리할 수 있습니다.

이제 위 로직을 hook으로 만들면 더 간결한 코드를 작성할 수 있습니다.

```jsx
const useThrowAsyncError = () => {
  const [state, setState] = useState();

  return (error) => {
    setState(() => {
      throw error;
    });
  };
};
```

```jsx
const Component = () => {
  const throwAsyncError = useThrowAsyncError();

  useEffect(() => {
    fetch("/bla")
      .then()
      .catch((e) => {
        // throw async error here!
        throwAsyncError(e);
      });
  });
};
```
