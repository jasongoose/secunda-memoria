---
title: For Statement
date: 2024-11-03 17:12:36
categories:
  - Studies
  - JavaScript
  - Loop
#tags:
---
## for

JS 반복문들 중 하나로 `;`로 구분되는 3개의 option expression으로 구성되고 loop 상에서 block statement에 있는 로직을 수행합니다.

```js
for ([initialization]; [condition]; [final_expression]) {
  statement;
}

// initialization
// loop를 실행하기 전에 var 또는 let으로 선언된 변수에 초기값을 할당합니다.

// condition
// loop를 실행할지 여부를 판별하기 위한 조건식이 위치합니다.
// (omitted), true -> statement 수행
// false -> statment는 skip하고 final expression으로 이동

// final_expression
// 현재 loop를 종료하고 나서 마무리로 수행하는 expression이 위치합니다.

// statement
// condition에서 true로 판정되었을 때 수행할 로직이 위치합니다.
```

`var`로 선언된 변수는 for 문과 동일한 scope를, `let`으로 선언된 변수는 for 문 내부 block scope를 가집니다.

```js
var i = 0;
for (; i < 9; i++) {
  console.log(i);
  // more statements
}
// let은 불가~
```

iterate 중간에 break가 가능한 것은 for 문만의 특징으로 볼 수 있습니다.

```js
for (let i = 0; ; i++) {
  console.log(i);
  if (i > 3) break;
  // more statements
}
```

for 문으로 infinite loop도 간단하게 만들 수 있습니다.

```js
for (;;) {
  // ...
}
```

## for await-of

`for-of` 문의 asynchronous 버전으로, async iterable의 value들을 순회하기 위한 반복문입니다.

async iterable의 `[Symbol.asyncIterable]` 메서드는 settled promise를 생성하는 async iterator를 반환합니다.

순회할 때 `[Symbol.asyncIterable]` 메서드가 없다면 `[Symbol.asyncIterable]`을 대신 사용하는데, sync iterator의 `next`, `return`, `throw`로 반환되는 값들을 settled promise로 wrapping합니다.

기존 `for-of` 문과의 차이점들을 정리하면 다음과 같습니다.

1. `for await-of` 문은 sync, async iterable에 모두 사용할 수 있습니다.
2. `for await-of` 문은 모듈과 `async` 함수 내부에서만 사용할 수 있습니다.
3. promise들을 yield하는 iterable이라면 sync iterator는 promise 그대로, async iterator는 promise의 awaited value를 생성합니다.
4. `for await-of` 문은 variable의 이름을 "async"로 지정해도 되는데, `for-of`문은 불가합니다.

```js
for await (async of foo) {
  // ...
}
```

[여기서 순회하는 중에 throw되거나 rejected promise가 yield된다면 순회를 멈춘다는 점을 유의해야 합니다.](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for-await...of#iterating_over_sync_iterables_and_generators)

```js
function* generatorWithRejectedPromises() {
  try {
    yield 0;
    throw 1; // <<
    yield 2;
    yield Promise.resolve(3);
    yield Promise.reject(4); // <<
    yield 5;
  } finally {
    console.log("called finally");
  }
}

(async () => {
  try {
    for await (const num of generatorWithRejectedPromises()) {
      console.log(num);
    }
  } catch (e) {
    console.log("caught", e);
  }
})();
// 0
// caught 1
```

## for-in

특정 객체의 enumerable + non-Symbol property key들을 순회합니다.

여기서 loop에 의한 iteration으로 **객체 자신뿐만 아니라 chain에 위치한 prototype 객체들의 key도 모두 접근합니다**.

객체의 prototype들이 아닌 오직 객체 자신만의 속성(own-property)만을 접근할 때는

- `getOwnPropertyNames` 메서드를 사용하거나
- `hasOwnProperty` 또는 `propertyIsEnumerable` 메서드로 분기점을 생성하면 됩니다.

보통 객체 + prototype들의 property key값을 확인하는 디버깅 용도로 많이 사용됩니다.

```js
const object = { a: 1, b: 2, c: 3 };

for (const property in object) {
  console.log(`${property}: ${object[property]}`);
}

// Expected output:
// "a: 1"
// "b: 2"
// "c: 3"
```

## for-of

[iterable](../protocols#iterable)의 property value들을 순회할 때 사용하는 반복문입니다.

![For Of](/images/for_of.png)
