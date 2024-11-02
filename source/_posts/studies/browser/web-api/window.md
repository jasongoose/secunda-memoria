---
title: Window
date: 2024-11-02 00:10:21
categories:
  - Studies
  - Browser
  - Web API
#tags:
---
전역객체인 `window`는 현재 JS 코드가 실행되고 있는 창이나 탭을 참조합니다.

MDN 공식문서를 보면 `Window` interface가 언급되는데 `window` 객체와의 관계를 다음과 같이 표현할 수 있습니다.

```js
console.assert(window instanceof Window);
```

하나의 창에 여러 개의 탭이 있는 경우, 각 탭에서 실행되는 JS 코드가 참조하는 `window` 객체는 모두 다릅니다.

## 참고자료

[📜 What is the difference between window and Window?](https://stackoverflow.com/questions/24008630/what-is-the-difference-between-window-and-window)

[📜 Window](https://developer.mozilla.org/en-US/docs/Web/API/Window)