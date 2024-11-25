---
title: Curry Function
date: 2024-10-28 08:42:28
categories:
  - Series
  - Composing Software
  - Concepts
#tags:
---
## 커리함수란?

다수의 인자를 한번에 하나씩 받는 함수를 가리키는데 arrow function으로 간결하게 표현할 수 있습니다.

```js
// add = a => b => Number
const add = (a) => (b) => a + b;
```

## Point-free Style

커리함수로 인자를 참조하지 않는 정의를 가지는 point-free style 함수를 구현할 수 있습니다.

```js
const inc = add(1); // point-free style

inc(3); // => 4
```

point-free style 함수는 다음과 같은 사용이점들이 있습니다.

- generalization
    - 다양한 타입의 데이터를 인자로 전달할 수 있습니다.
- specialization
    - 일부 인자를 고정하여 원하는 용도로 범위를 좁힐 수 있어서 function composition을 구현할 수 있습니다.

## Utils

커리함수로 구현할 수 있는 util들은 다음과 같습니다.

### compose

여러 함수들을 인자로 받아서 대수학의 합성함수를 반환합니다.

```js
const compose =
  (...fns) =>
  (x) =>
    fns.reduceRight((y, f) => f(y), x);
// compose(f, g, h)(x) => f(g(h(x)))
```

### pipe

여러 함수들을 인자로 받아서 순서대로 합성하는 함수를 반환합니다.

```js
const pipe =
  (...fns) =>
  (x) =>
    fns.reduce((y, f) => f(y), x);
// pipe(f, g, h)(x) => h(g(f(x)))
```

### asyncPipe

일련의 비동기 작업들을 순서대로 처리하는 pipe 함수입니다.

```js
const asyncPipe = (...fns) => x => (
	fns.reduce(async (y, f) => f(await y), x)
);
```

### trace

compose, pipe 함수 내에서 중간에 연산된 값을 고정된 label과 함께 전달합니다.

```js
const trace = (label) => (value) => {
  console.log(`${label}: ${value}`);
  return value;
};
```

```js
const g = (n) => n + 1;
const f = (n) => n * 2;

pipe(g, trace("after g"), f, trace("after f"))(20);
/*
after g: 21
after f: 42
*/
compose(trace("after f"), f, trace("after g"), g)(20);
/*
after g: 21
after f: 42
*/
```

## 설계방식

커리함수를 정의할 때 “data last” 즉, 최종함수가 받는 인자를 가장 마지막에 전달하는 방식을 권장합니다.

```js
const map = (fn) => (mappable) => mappable.map(fn);

const pipe =
  (...fns) =>
  (x) =>
    fns.reduce((y, f) => f(y), x);

const log = (...args) => console.log(...args);

const isEven = (n) => n % 2 === 0;

const stripe = (n) => (isEven(n) ? "dark" : "light");
```

```js
const arr = [1, 2, 3, 4];

const stripeAll = map(stripe);
const striped = stripeAll(arr);

log(striped);
// => ["light", "dark", "light", "dark"]
```

```js
const double = (n) => n * 2;

const doubleAll = map(double);
const doubled = doubleAll(arr);

log(doubled);
// => [2, 4, 6, 8]
```
