---
title: State as a Snapshot
date: 2024-11-04 08:19:53
categories:
  - Studies
  - React
  - Learn React
#tags:
---
React에서 컴포넌트를 렌더링하는 것은 컴포넌트를 정의한 함수를 호출하여 현재 state 기반의 UI snapshot(JSX)을 얻는 것과 같습니다.

컴포넌트 내부의 event handler에 의해서(아니면 다른 함수의 호출로) 일련의 `setState`들을 호출하면 최종 state들을 반영한 snapshot이 생성되어 실제 DOM에 반영됩니다.

## setState

`setState`는 다음 렌더링 때 사용할 state 값을 지정하는 역할만 가질 뿐, `setState`의 호출에 의해서 즉각적으로 + 동기적으로 리렌더링이 일어나지 않습니다.

아래와 같이 특정 event handler에서 `setState`를 3번 호출한다고 해도 현재 state는 아직 0이므로 다음 state는 3이 아니라 1이 됩니다.

```jsx
export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button
        onClick={() => {
          setNumber(number + 1); // setNumber(1)
          setNumber(number + 1); // setNumber(1)
          setNumber(number + 1); // setNumber(1)
          // 이 함수가 종료될 때까지 각 setState는 리렌더링을 위해서 queueing됩니다!
        }}
      >
        +3
      </button>
    </>
  );
}
```

실제 queueing을 거쳐서 새로운 state가 지정되는 코드는 바로 `useState`를 호출하는 부분입니다.

## 참고자료

[Learn React](https://react.dev/learn)