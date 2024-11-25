---
title: Virtual DOM
date: 2024-11-03 20:55:52
categories:
  - Studies
  - Vue3
  - Rendering Mechanism
#tags:
---
![Virtual DOM](/images/vdom.jpeg)

React에서 처음 소개한 개념으로 메모리 상에 DOM 트리(Virtual DOM 트리, Vtree)를 구현한 JS 객체를 저장한 상태에서 Real DOM 트리(Rtree)와 동기화시키는 메커니즘입니다.

임의의 Node의 변화를 반영한 DOM tree를 렌더링할 때, 새로고침으로 처음부터 다시 Rtree를 구축하지 않습니다.

대신 변경된 Vnode를 포함한 새로운 Vtree를 생성한 뒤 기존의 Vtree와 비교했을 때, 다른 부분만 찾아내어 그에 맞게 Rnode들만 수정합니다.

Vue3에서 Vnode는 다음과 같이 저장합니다.

```js
const vnode = {
  type: "div",
  props: {
    id: "hello",
  },
  children: [
    /* more vnodes */
  ],
};
```
