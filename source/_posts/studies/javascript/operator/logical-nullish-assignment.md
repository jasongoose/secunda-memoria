---
title: Logical Nullish Assignment
date: 2024-11-03 17:34:51
categories:
  - Studies
  - JavaScript
  - Operator
#tags:
---
nullish coalescing을 활용한 특이한 short-circute입니다.

```js
x ??= y; // x ?? (x = y);
```

특정 객체에는 없는 속성을 추가하면서 default 값을 지정할 때 활용할 수 있습니다.

```js
function config(options) {
  options.duration ??= 100;
  options.speed ??= 25;
  return options;
}

config({ duration: 125 }); // { duration: 125, speed: 25 }
config({}); // { duration: 100, speed: 25 }
```
