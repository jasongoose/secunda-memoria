---
title: Updating Objects in State
date: 2024-11-04 08:21:02
categories:
  - Studies
  - React
  - Learn React
#tags:
---
`useState` hook으로 객체형 state를 관리할 때도 동일한 syntax를 사용하면 됩니다.

```jsx
const [position, setPosition] = useState({ x: 0, y: 0 });
```

여기서 object mutation으로 객체의 속성을 수정해도 원본 객체의 속성값은 변하지만 이에 맞춰 컴포넌트의 리렌더링은 trigger되지 않습니다!

```jsx
position.x = 8; // 🤮
```

따라서 리렌더링을 trigger하려면 아래와 같이 새로운 객체로 대체해야 합니다😇

```jsx
// 기존 속성들을 유지한 shallow-copy으로 업데이트
setPosition((curr) => ({
  ...curr,
  x: valueX,
  y: valueY,
}));

// or

// 완전히 새로운 객체로 덮어쓰기
setPosition({
  x: newValueX,
  y: newValueY,
});
```

중첩된 객체의 state를 업데이트하는 방법도 마찬가집니다.

```jsx
const [person, setPerson] = useState({
  name: "Niki de Saint Phalle",
  artwork: {
    title: "Blue Nana",
    city: "Hamburg",
    image: "https://i.imgur.com/Sd1AgUOm.jpg",
  },
});
```

```jsx
setPerson({
  ...person, // Copy other fields
  artwork: {
    // but replace the artwork
    ...person.artwork, // with the same one
    city: "New Delhi", // but in New Delhi!
  },
});
```

위와 같이 중첩된 객체의 깊은 속성을 업데이트하는 코드는 꽤 복잡해질 수 있는데, 여기서 [ImmerJS](https://github.com/immerjs/use-immer)를 사용하면 object mutation syntax로 중첩된 state를 업데이트할 수 있습니다.

```jsx
import { useImmer } from "use-immer";

const [person, setPerson] = useImmer({
  name: "Niki de Saint Phalle",
  artwork: {
    title: "Blue Nana",
    city: "Hamburg",
    image: "https://i.imgur.com/Sd1AgUOm.jpg",
  },
});

setPerson((draft) => {
  draft.artwork.city = "New Delhi";
});
```

## 참고자료

[Learn React](https://react.dev/learn)