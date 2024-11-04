---
title: How to debounce and throttle in React without losing your mind
date: 2024-11-04 21:16:00
categories:
  - Studies
  - React
  - Developer Way React
#tags:
---
debounce와 throttle은 짧은 시간동안 빈번하게 발생하는 함수의 실행을 일부 생략하기 위한 기법들입니다.

특정 콜백함수를 정해진 시점에 실행하기 위해서 내부에 타이머가 있는데, 이 타이머를 유지해야 리렌더링에 걸쳐서 debounce, throttle을 수행할 수 있습니다.

## Debounced callback in React: dealing with re-renders

input에 값을 입력하는 동안 상태 업데이트로 컴포넌트가 리렌더링되어도 처음에 생성한 timer를 유지하려면 그냥 컴포넌트 외부에 정의하는 것도 방법입니다.

```jsx
const sendRequest = (value) => {
  // send value to the backend
};
const debouncedSendRequest = debounce(sendRequest, 500);

const Input = () => {
  const [value, setValue] = useState();

  const onChange = (e) => {
    const value = e.target.value;
    setValue(value);

    // debouncedSendRequest is created once, so state caused re-renders won't affect it anymore
    debouncedSendRequest(value);
  };

  return <input onChange={onChange} value={value} />;
};
```

하지만 위 방식은 debounce 콜백인 sendRequest에서 컴포넌트의 state나 props를 사용하지 않는 경우에만 적용할 수 있습니다.

더 유연한(?) 구현을 위해 아래와 같이 memoization을 적용할 수 있습니다.

```jsx
const Input = () => {
  const [value, setValue] = useState("initial");

  // memoize the callback with useCallback
  // we need it since it's a dependency in useMemo below
  const sendRequest = useCallback((value: string) => {
    console.log("Changed value:", value);
  }, []);

  // memoize the debounce call with useMemo
  const debouncedSendRequest = useMemo(() => {
    return debounce(sendRequest, 1000);
  }, [sendRequest]);

  const onChange = (e) => {
    const value = e.target.value;
    setValue(value);
    debouncedSendRequest(value);
  };

  return <input onChange={onChange} value={value} />;
};
```

## Debounced callback in React: dealing with state inside

만일 아래와 같이 debounce 콜백에서 컴포넌트 상태값을 직접 사용한다면 dependency인 value의 끊임없는 업데이트로 sendRequest와 debouncedSendRequest 함수가 재정의되어 debounce 처리가 되지 않습니다.

```jsx
const Input = () => {
  const [value, setValue] = useState("initial");

  const sendRequest = useCallback(() => {
    // value is now coming from state
    console.log("Changed value:", value);

    // adding it to dependencies
  }, [value]);

  // ...이후는 위와 동일
};
```

컴포넌트 리렌더링에도 상관없이 timer를 유지하려면 ref를 이용하면 됩니다.

```jsx
const ref = useRef(
  debounce(() => {
    // this value is coming from state
    console.log(value);
  }, 500)
);
```

하지만 위와 같이 value를 그대로 사용하면 closure 현상에 의해서 최초 mount에 의한 렌더링으로 초기화된 value값을 참조하기 때문에 최근 상태값을 사용할 수 없습니다.

그렇다고 해서 매 리렌더링마다 `ref.current` 에 새로운 debounce 함수를 업데이트하면 첫번째 예시와 동일하게 debounce 되지 않습니다.

```jsx
const Input = () => {
  const [value, setValue] = useState();

  // creating ref and initializing it with the debounced backend call
  const ref = useRef(
    debounce(() => {
      // send request to the backend here
    }, 500)
  );

  useEffect(() => {
    // updating ref when state changes
    ref.current = debounce(() => {
      // send request to the backend here
    }, 500);
  }, [value]);

  const onChange = (e) => {
    const value = e.target.value;
    setValue(value);

    // calling the debounced function
    ref.current();
  };

  // ...
};
```

위 상황을 해결하려면 (1) `ref.current` 에는 debounce 콜백을 저장하고, (2) timer를 유지하기 위해 debouce 함수를 `useMemo` 로 memoise하면 됩니다.

```jsx
const Input = () => {
  const [value, setValue] = useState();

  const sendRequest = () => {
    // send request to the backend here
    // value is coming from state
    console.log(value);
  };

  // creating ref and initializing it with the sendRequest function
  const ref = useRef(sendRequest);

  useEffect(() => {
    // updating ref when state changes
    // now, ref.current will have the latest sendRequest with access to the latest state
    ref.current = sendRequest;
  }, [value]);

  // creating debounced callback only once - on mount
  const debouncedCallback = useMemo(() => {
    // func will be created only once - on mount
    const func = () => {
      // ref is mutable! ref.current is a reference to the latest sendRequest
      ref.current?.();
    };
    // debounce the func that was created once, but has access to the latest sendRequest
    return debounce(func, 1000);
    // no dependencies! never gets updated
  }, []);

  const onChange = (e) => {
    const value = e.target.value;

    // calling the debounced function
    debouncedCallback();
  };
};
```

위와 같이 구현하면 debouncedCallback은 memoise되어 첫 번째 렌더링 이후로 timer가 유지됩니다.

또한 commit 이후 onChange 이벤트 핸들러가 실행될 시점에는 `ref.current`가 참조하는 sendRequest는 현재 컴포넌트의 상태값인 value를 사용하기 때문에 제대로 동작하게 됩니다.

이제 debounce 관련 로직들을 하나의 hook으로 만들어 사용하면 간결한 코드를 작성할 수 있게 됩니다.

```jsx
const useDebounce = (callback) => {
  const ref = useRef();

  useEffect(() => {
    ref.current = callback;
  }, [callback]);

  const debouncedCallback = useMemo(() => {
    const func = () => {
      ref.current?.();
    };
    return debounce(func, 1000);
  }, []);

  return debouncedCallback;
};
```

```jsx
const Input = () => {
  const [value, setValue] = useState();

  const debouncedRequest = useDebounce(() => {
    // send request to the backend
    // access to latest state here
    console.log(value);
  });

  const onChange = (e) => {
    const value = e.target.value;
    setValue(value);
    debouncedRequest();
  };

  return <input onChange={onChange} value={value} />;
};
```
