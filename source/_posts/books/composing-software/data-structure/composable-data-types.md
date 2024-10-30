---
title: Composable Data Types
date: 2024-10-29 22:11:44
categories:
  - Books
  - Composing Software
  - 3. Data Structure
#tags:
---
JS에서 함수는 객체이므로 [Object Composition](../../concepts/object-composition)으로 함수에 속성이나 메서드를 추가할 수 있습니다.

그래서 함수를 사용하면 data type(interface)과 다수 data type간의 연산을 정의할 수 있고, [compose](../../concepts/curry-function#compose)나 [pipe](../../concepts/curry-function#pipe)로 data type들을 합성하여 만들 수 있습니다.

```js
const t6 = pipe(t1, t2, t3);
```

예를 들어 아래와 같은 interface를 제공하는 data type `t`가 있다고 가정합시다.

## Identity

```js
t(x)(t(0)) === t(x);
```

## Associativity

```js
t(a)(t(b))(t(c)) === t(a)(t(b)(t(c))); // (a + b) + c === a + (b + c)
```

그럼 data type `t`는 아래와 같이 정의할 수 있습니다.

```js
const t = (value) => {
  const add = (n) => t(n + value);

  return Object.assign(add, {
    // 기존 Object.prototype 메서드 override
    toString: () => `t${value}`,
    valueOf: () => value,
  });
};
// 🙌 composable data type => FUNCTION w/ valueOf
```

```js
const pipe =
  (...fns) =>
  (x) =>
    fns.reduce((y, f) => f(y), x);

const addPl = (...fns) => pipe(...fns)(t(0));

const sum = addPl(t(2), t(4), t(-1)); // t(5)
```
