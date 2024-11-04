---
title: Implementing advanced usePrevious hook with React useRef
date: 2024-11-04 21:22:13
categories:
  - Studies
  - React
  - Developer Way React
#tags:
---
React에서 `useRef` 의 반환값인 ref는 컴포넌트의 리렌더링을 일으키지 않지만 렌더링 간의 값이 유지하는 변수입니다.

ref의 값인 `ref.current` 를 수정하면 컴포넌트의 리렌더링은 일어나지 않지만 다른 원인에 의해서 컴포넌트가 리렌더링된다면 해당 snapshot에서는 `ref.current` 의 최신값을 확인할 수 있습니다.

ref를 이용한 예시 중 하나인 `usePrevious` hook은 직전 상태값에 접근할 때 사용하고 아래와 같이 구현할 수 있습니다.

```jsx
const usePrevious = (value) => {
  const ref = useRef();
  useEffect(() => {
    ref.current = value;
  });
  return ref.current;
};
```

컴포넌트 리렌더링이 일어날 때마다 `ref.current` 값은 직전 렌더링 때 값을 참조하지만 리렌더링 이후 DOM에 반영될 시점에 값이 업데이트됩니다.

하지만 위와 같이 구현하면 **직전 상태값이 아닌 직전 렌더링 때 값을 참조**하게 되어 `ref.current` 의 값 자체의 변화를 고려하지 않으므로 아래와 같은 수정이 필요합니다.

```jsx
export const usePrevious = (value) => {
  // initialize the ref with previous and current values
  const ref = useRef({
    value: value,
    prev: null,
  });

  const current = ref.current.value;

  // if the value passed into hook doesn't match what we store as "current"
  // move the "current" to the "previous"
  // and store the passed value as "current"
  if (value !== current) {
    ref.current = {
      value: value,
      prev: current,
    };
  }

  // return the previous value only
  return ref.current.prev;
};
```

기존의 `useEffect` 를 제거하고 컴포넌트 리렌더링 중에 이전 값과 value 인자로 들어온 값을 비교하여 다른 경우에만 `ref.current` 를 업데이트합니다.

만일 value 인자로 객체가 전달된다면 따로 matcher function을 따로 정의해서 `usePrevious` hook으로 전달하면 됩니다.

```jsx
export const usePrevious = (value, isEqualFunc) => {
  // ...
  if (isEqualFunc ? !isEqualFunc(value, current) : value !== current) {
    // ...
  }

  // ...
};
```

이제 `usePrevious` hook의 인자로 props와 같은 reactive value들을 전달하여 사용하면 됩니다.
