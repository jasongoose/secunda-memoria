---
title: Extracting State Logic into a Reducer
date: 2024-11-04 08:18:01
categories:
  - Studies
  - React
  - Learn React
#tags:
---
`useState` hook을 사용하면 컴포넌트 내부 event handler에서 state setter를 호출하여 사용자 반응에 맞춰 state를 업데이트할 수 있습니다.

하지만 event handler의 개수가 많아진다면 state setter가 코드 곳곳에 분산되어서 가독성이 떨어지고 어디서 state 업데이트가 일어나는지 추적이 어려워집니다🤮

## useReducer

React에서는 컴포넌트의 상태를 일관된 수단으로 업데이트할 수 있도록 `useReducer` hook을 제공합니다.

`useReducer`는 첫번째 인자로 reducer, 두번째 인자로 initialState를 전달합니다.

```jsx
const [state, dispatch] = useReducer(reducer, initialState);

// state : 컴포넌트 상태값
// dispatch : action 객체를 받아서 state를 업데이트하는 함수
```

위에서 reducer는 현재 상태값과 action 객체를 받아서 새로운 상태값을 반환하는 [순수함수](../../../books/composing_software/concepts/pure_function.md)입니다.

```jsx
const reducer = (currState, action) => newState;

// action : state에 적용할 업데이트 정보(type) + 필요한 인자를 가진 객체
```

`useReducer`를 활용하면 다음과 같은 이점들이 있습니다.

1. 컴포넌트 코드량 감소

    - 상태를 업데이트하는 로직을 개별 파일에 독립된 reducer로 구현할 수 있습니다.

2. 가독성 증가

    - 사용자 이벤트에 의해서 상태를 어떻게 업데이트할지가 아닌 어떤 업데이트를 적용할지를 선언적으로 명시할 수 있습니다.

3. 디버깅 용이성
    - 사용자 이벤트의 타입을 확인하여 상태가 어디서 업데이트 되었는지 확인할 수 있습니다.
    - action 객체 필드값 + reducer 코드를 확인하여 상태가 왜 업데이트 되는지 확인할 수 있습니다.
