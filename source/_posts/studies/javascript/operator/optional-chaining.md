---
title: Optional Chaining
date: 2024-11-03 17:35:38
categories:
  - Studies
  - JavaScript
  - Operator
#tags:
---
임의의 객체에 특정 속성이나 메서드가 없을 때, 에러를 발생시키지 않고 `undefined`만 조용히 반환합니다.

```js
const adventurer = {
  name: "Alice",
  cat: {
    name: "Dinah",
  },
};

const dogName = adventurer.dog?.name;
console.log(dogName);
// Expected output: undefined

console.log(adventurer.someNonExistentMethod?.());
// Expected output: undefined
```

## 참고자료

[Optional Chaining](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Optional_chaining)