---
title: Updating Arrays in State
date: 2024-11-04 08:14:13
categories:
  - Studies
  - React
  - Learn React
#tags:
---
`useState` hook으로 배열형 state를 관리하는 방식은 객체형 state를 관리하는 방식과 동일합니다.

배열형 state를 업데이트할 때 아래와 같이 요소에 대한 mutation을 적용해도 리렌더링이 trigger되지 않습니다.

```jsx
const arr = useState(["d", "e", "f"]);
arr[0] = "bird"; // ❌
```

대신 non-mutating 메서드로 새로운 배열을 생성해서 `setState` 인자에 넘겨주면 됩니다.

![Array State](/images/array_state.png)

[ImmerJS](https://github.com/immerjs/immer) 라이브러리를 사용한다면 mutating 메서드로도 리렌더링을 trigger할 수 있습니다.
