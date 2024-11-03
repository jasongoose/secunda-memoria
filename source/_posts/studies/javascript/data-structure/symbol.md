---
title: Symbol
date: 2024-11-03 16:21:01
categories:
  - Studies
  - JavaScript
  - Data Structure
#tags:
---
ES6에서 추가한 새로운 primitive 값으로 매 생성할 때마다 고유한 즉, 중복되지 않는 값을 가진다는 특징이 있습니다.

이 값은 `number`도, `string`도 아닌 `symbol`이라는 타입을 가집니다.

## Using Symbol

symbol을 생성할 때는 `Symbol` 생성자를 사용하면 됩니다.

```js
let sym1 = Symbol();
let sym2 = Symbol("foo");
let sym3 = Symbol("foo");

Symbol("foo") === Symbol("foo"); // false
```

## Global Symbol Registry

Symbol 생성자로 만들어진 `symbol`은 일회성이기 때문에 한번 정의하면 다시 해당 값을 읽을 수 없습니다.

코드 상에서 사용된 symbol들을 전역으로 관리할 때는 Global Symbol Registry를 사용합니다.

해당 registry에 등록된 symbol들은 프로젝트 내 파일들 또는 고유의 global scope를 가진 realm(?)들 사이에서 공유됩니다.

symbol을 새로 등록하거나 조회할 때는 `Symbol.for` 메서드를 사용하면 됩니다.

```js
Symbol.for("foo"); // create a new global symbol
Symbol.for("foo"); // retrieve the already created symbol

// Same global symbol, but not locally
Symbol.for("bar") === Symbol.for("bar"); // true
Symbol("bar") === Symbol("bar"); // false

// The key is also used as the description
const sym = Symbol.for("mario");
sym.toString(); // "Symbol(mario)"
```

## Object Private Property

symbol은 객체의 key 값으로 사용할 수 있습니다.

```js
const obj = {};
const sym = Symbol();

obj[sym] = "foo";
obj.bar = "bar";

console.log(obj); // { bar: 'bar' }
console.log(sym in obj); // true
console.log(obj[sym]); // foo
console.log(Object.keys(obj)); // ['bar']
```

`Object.keys()`의 반환값에 symbol이 포함되지 않은 것은 ES6 이전 명세와의 호환성을 위해서인데, 이러한 성질을 이용하면 외부에 공개하지 않을 private 속성을 만들 수 있습니다.

`Reflect`의 메서드를 사용하면 symbol key를 읽을 수 있습니다.

```js
function tryToAddPrivate(o) {
  o[Symbol("Pseudo Private")] = 42;
}
const obj = { prop: "hello" };
tryToAddPrivate(obj);

console.log(Reflect.ownKeys(obj)); // [ 'prop', Symbol(Pseudo Private) ]
```

## Preventing Property Name Collisions

symbol은 라이브러리(패키지)에서 사용자가 생성한 객체에 metadata 속성을 append하면서 기존 key값의 충돌을 방지할 때 유용하게 사용됩니다.

```js
const obj = {};

obj[Symbol()] = 1;
Object.defineProperty(obj, "foo", {
  enumberable: false,
  value: 2,
});

console.log(Object.keys(obj)); // []
console.log(Reflect.ownKeys(obj)); // [ 'foo', Symbol() ]
console.log(JSON.stringify(obj)); // {}
```

사실 중복을 방지하기 위해서 uuid와 같이 timestamp에 기반한 128bit 값을 이용하거나 namespace를 사용할 수는 있지만 머리가 비상한 사용자에 의해서 중복이 발생할 수학적 확률이 다분합니다.

## 참고자료

[Symbol](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol)

[JavaScript Symbols: But Why?](https://medium.com/intrinsic-blog/javascript-symbols-but-why-6b02768f4a5c)
