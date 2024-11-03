---
title: Responding to Events
date: 2024-11-04 08:16:30
categories:
  - Studies
  - React
  - Learn React
#tags:
---
React는 Vue와는 다르게 DOM native event만 사용하기 때문에 [event propagation](../../../browser/web_api/dom.md)이 일어납니다.

```jsx
export default function Toolbar() {
  return (
    <div
      className="Toolbar"
      onClick={() => {
        alert("You clicked on the toolbar!"); // 여기서도 한번
      }}
    >
      <button onClick={() => alert("Playing!") /* 여기서 한번 */}>
        Play Movie
      </button>
      <button onClick={() => alert("Uploading!")}>Upload Image</button>
    </div>
  );
}
```

만일 propagation(대부분 bubbling)을 중간에 멈출려면 event handler에서 `stopPropagation` 메서드를 호출하면 됩니다.

```jsx
function Button({ onClick, children }) {
  return (
    <button
      onClick={(e) => {
        e.stopPropagation();
        onClick();
      }}
    >
      {children}
    </button>
  );
}

export default function Toolbar() {
  return (
    <div
      className="Toolbar"
      onClick={() => {
        alert("You clicked on the toolbar!");
      }}
    >
      <Button onClick={() => alert("Playing!")}>Play Movie</Button>
      <Button onClick={() => alert("Uploading!")}>Upload Image</Button>
    </div>
  );
}
```

event handler는 렌더링 이후에 실행되므로 state를 업데이트하는 등의 side-effect 관련 로직들을 위한 최적의 위치입니다!

## 참고자료

[Learn React](https://react.dev/learn)