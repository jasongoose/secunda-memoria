---
title: Transducer
date: 2024-10-28 08:45:18
categories:
  - Series
  - Composing Software
  - Concepts
#tags:
---
## Why Transducers?

reducer(다수의 입력들을 일련의 누적과정을 거쳐서 하나의 출력을 만드는 함수)를 인자로 받아서 새로운 reducer를 반환하는 고차함수입니다.

```js
reducer = (accumulator, current) => accumulator;
transducer = (reducer) => newReducer;
```

transducer는 다음과 같은 사용이점들이 있습니다.

- 여러 함수들을 합성해서 만들 수 있다.
- 중간에 새로운 객체를 생성할 필요없이 대량의 연산을 요소별로 + 한번에 + 연속적으로 처리할 수 있다.
- array, stream 등의 enumerable 데이터에 대해서 적용할 수 있다.
- 서로 다른 타입의 데이터 사이의 mapping, filtering, insertion 등을 구현할 수 있다.

## Iterate 연산

JS에서 Iterable 객체의 요소별로 변환하는 연산은 2가지로 나눌 수 있습니다.

### Push API

`Array.prototype.reduce`와 같이 요소별로 모든 변환을 거쳐서 최종 결과값이 나올 때마다 누적시키는 eager evaluation 방식입니다.

### Pull API

Generator, Iterable 객체와 같이 요소의 변환이 필요할 때마다 consumer가 next 요청을 전달해야 하는 lazy evaluation 방식입니다.

transducer를 통해 입력으로부터 출력을 생성하는 과정은 마치 신호 변환기의 원리와 유사합니다.

그래서 reducer의 입출력을 signal 또는 stream(시간에 따라 나타나는 일련의 값들)으로도 불립니다.

![Transducer](/images/transducer.png)

## Implementation

transducer는 다음과 같은 [partial function](../curry-function)으로 구현합니다.

```js
const td = (transform) => (step) => (acc, curr) => step(acc, transform(curr));

// transform : 요소에 대한 최초 변환함수

// step : 다음 stage에서 수행할 reducer로,
// 현재까지 누적값 acc과 변환된 curr값을 인자로 받아 새로운 누적값을 만듭니다.
```

다만 실무에서 transducer를 적용할 때는 [Ramda](https://ramdajs.com/), [Transducers-JS](https://github.com/cognitect-labs/transducers-js), [RxJS](https://rxjs.dev/guide/overview) 등의 외부 라이브러리를 사용하는 것을 권장합니다.

## Practice

mapping과 filter 기능을 가진 transducer는 각각 다음과 같이 구현할 수 있습니다.

```js
const map = (f) => (step) => (a, c) => step(a, f(c));

const filter = (predicate) => (step) => (a, c) => predicate(c) ? step(a, c) : a;
// 필터링 조건을 만족해야 step reducer에 의한 변환이 적용되고,
// 충족하지 않으면 그대로 이전 acc가 그대로 반환됩니다.
```

두 transducer를 [compose](../curry-function#compose)로 합성하면 배열에서 짝수들만 골라서 2를 곱하는 reducer를 구현할 수 있습니다.

```js
const isEven = (n) => n % 2 === 0;
const double = (n) => n * 2;

const compose =
  (...fns) =>
  (x) =>
    fns.reduceRight((y, f) => f(y), x);

const arrayConcat = (a, c) => a.concat([c]); // 최초 step fn
```

```js
const doubleEvens = compose(filter(isEven), map(double));

const xform = doubleEvens(arrayConcat); // 최종 reducer
const result = [1, 2, 3, 4, 5, 6].reduce(xform, []); // [4, 8, 12]
```

합성함수를 안에서부터 하나씩 실행할 때마다 반환되는 reducer를 구하면 다음과 같습니다.

```js
// 1. map(double)
(acc, curr) => acc.concat([double(curr)]);

// 2. filter(isEven)
(acc, curr) => isEven(curr) ? acc.concat([double(curr)]) : acc; // <= xform
```

transducer들을 조합할 때는 [pipe](../curry-function.md#pipe)가 아닌 [compose](../curry-function#compose)로 합치지만 실제 최종 결과함수는 pipe 순서대로 적용된다는 특징이 있습니다.

```js
const final = compose(transducer1, transducer2, /*...*/ transducerN)(reducer);
const ans = iteratble.reduce(final);

// reducer는 compose의 정의에 따라 transducerN에서 transducer1 방향으로 적용되지만
// ans를 구하는 과정에서 transducer1에서 transducerN 방향으로 연산을 수행합니다.
```

## Transducer Rules

transducer가 따라야하는 규칙들입니다.

### Initialization

transducer에서 첫 번째 reducer의 초기 누적값은 다음 reducer인 step에 의해서 지정되어야 합니다.

```js
const map =
  (f) =>
  (step) =>
  (acc = step(), curr) =>
    step(acc, f(curr));
```

reducer를 인자없이 호출할 때, 초기 누적값을 반환하도록 작성하는건 좋은 습관입니다.

### Early Termination

[compose](../curry-function#compose)에 의해서 반환된 reducer를 reduce 콜백으로 전달하면 배열 요소별로 transducer들을 거쳐서 연산이 수행됩니다.

여기서 transducer는 특수한 reduced value를 반환하여 중간에 변환하는 연산을 중단하여 배열 요소 자체를 acc로 반환하는 로직도 포함해야 합니다.

이와 같은 조기종료를 구현하려면 중간에 반환되는 reduced value가 특정 타입을 가져야만 합니다.

```js
const reduced = (v) => ({
  get isReduced() {
    return true;
  },
  valueOf: () => v,
  toString: () => `Reduced(${JSON.stringify(v)})`,
});
```
