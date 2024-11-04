---
title: Higher Order Function
date: 2024-10-28 08:44:07
categories:
  - Books
  - Composing Software
  - Concepts
#tags:
---
JS에서 함수는 일급객체라서 일반변수나 함수의 인자로 전달하거나 반환값으로 사용할 수 있습니다.

고차함수로 `Array.prototype.reduce` 메서드를 구현하면 다음과 같습니다.

```js
const reduce = (reducer, initial, arr) => {
  // shared stuff
  let acc = initial;
  for (let i = 0, { length } = arr; i < length; i++) {
    // unique stuff in reducer() call
    acc = reducer(acc, arr[i]);
    // more shared stuff
  }
  return acc;
};

reduce((acc, curr) => acc + curr, 0, [1, 2, 3]); // 6
```

위에서 만든 reduce 함수로 `Array.prototype.filter` 메서드를 구현하면 다음과 같습니다.

```js
const filter = (fn, arr) => {
  return reduce((acc, curr) => (fn(curr) ? acc.concat(curr) : acc), [], arr);
};
```

immutability를 위해서 `push`가 아닌 `concat`로 새로운 배열을 만드는 것을 확인할 수 있습니다.
