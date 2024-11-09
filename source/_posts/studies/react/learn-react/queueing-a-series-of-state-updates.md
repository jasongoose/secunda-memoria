---
title: Queueing a Series of State Updates
date: 2024-11-04 08:13:46
categories:
  - Studies
  - React
  - Learn React
#tags:
---
`setState` 함수에 인자를 전달하면 다음 snapshot에서 사용할 state 값을 지정할 수 있습니다.

```jsx
setState(nextValue);
```

React는 단일 event handler에서 발생한 다수의 `setState` 호출들을 queueing할 수 있는데, 호출된 순서대로 + 한꺼번에 state(들)을 업데이트하여 새로운 단일 snapshot을 만듭니다.

`setState`을 호출할 때마다 snapshot을 생성한다면 잦은 빈도 수의 리렌더링이 일어나서 불필요한 연산들이 수행되므로 비효율적입니다.

이러한 문제를 해결한 위와 같은 메커니즘을 batching이라고 합니다.

## updater function

`setState` 함수는 queueing에 의해 직전에 수정된 state 값을 인자로 받는 순수함수(updater function)를 콜백으로 받을 수 있습니다.

이 부분은 updater function들을 queueing하는 방식으로 구현됩니다.

queue에 저장된 하나의 updater를 호출하고 반환된 값은 다음 updater의 인자로 전달되어 처리되고 최종 반환된 값이 리렌더링에 사용할 state가 됩니다.

```jsx
export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button
        onClick={() => {
          setNumber((n) => n + 1); // 0 -> 1
          setNumber((n) => n + 1); // 1 -> 2
          setNumber((n) => n + 1); // 2 -> 3
        }}
      >
        +3
      </button>
    </>
  );
}
```
