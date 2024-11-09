---
title: Equality
date: 2024-11-03 17:34:19
categories:
  - Studies
  - JavaScript
  - Operator
#tags:
---
## ==

- 두 피연산자가 모두 객체일 때, 동일한 객체를 참조하는 경우 ⇒ `true`
- `null != undefined` ⇒ `true`
- 두 피연산자의 데이터 타입이 다른 경우, 다음과 같은 타입변환이 수행됩니다.
    - [`string`, `number`] ⇒ [`number`, `number`]
    - [`boolean`, non-boolean] ⇒ [1(`true`) | +0(`false`), non-boolean]
    - [`object`, `number`] ⇒ [`object.valueOf()`, `number`]
    - [`object`, `string`] ⇒ [`object.toString()`, `string`]

## ===

- 두 피연자의 데이터 타입이 다른 경우 ⇒ `false`
- 두 피연산자가 모두 객체일 때, 동일한 객체를 참조하는 경우 ⇒ `true`
- `null === null` ⇒ `true`
- `undefined === undefined` ⇒ `true`
- `NaN !== NaN` ⇒ `true`
- `+0 === -0` ⇒ `true`

신기하게도 `NaN` 은 임의의 값과 동등연산자(==)나 일치연산자(===)를 적용해도 항상 `false`가 나오므로, `NaN` 인지 여부는 전역함수인 `isNaN`을 사용하면 됩니다.

```js
console.log(NaN == NaN); // False
console.log(NaN === NaN); // False

console.log(isNaN(NaN)); // True
```