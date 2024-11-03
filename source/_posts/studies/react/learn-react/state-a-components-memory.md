---
title: State, A Component’s Memory
date: 2024-11-04 08:19:21
categories:
  - Studies
  - React
  - Learn React
#tags:
---
컴포넌트가 가지는 state는 다음 2가지 성질을 가집니다.

- 렌더링 전과 후에 값이 유지된다.
- 값이 업데이트되면 컴포넌트가 리렌더링된다.

state는 `useState` hook을 사용하여 생성합니다.

```jsx
const [state, setState] = useState("initial value");
```

React에서 제공하는 특별한 함수로, 컴포넌트 렌더링 중에 사용되어야 한다는 조건에 의해서 컴포넌트 또는 custom hook의 root-level + block 최상단에서만 호출되어야 합니다.

state는 컴포넌트 내부에서만 유효범위를 가집니다.

## 참고자료

[Learn React](https://react.dev/learn)