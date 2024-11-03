---
title: Object
date: 2024-11-03 16:18:23
categories:
  - Studies
  - JavaScript
  - Data Structure
#tags:
---
reference 타입 중 하나인 Object는 key-value pair로 entity들을 저장하기 위해서 사용되는 타입입니다.

대부분의 객체들을 모두 `Object`의 instance이기 때문에 prototype chain을 따라서 `Object.prototype`에 정의된 속성들과 메서드들을 사용할 수 있습니다.

object의 속성 자체를 삭제하는 메서드는 없기 때문에 `delete` 연산자를 사용합니다.

```js
const Employee = {
  firstname: "John",
  lastname: "Doe",
};

console.log(Employee.firstname);
// expected output: "John"

delete Employee.firstname;

console.log(Employee.firstname);
// expected output: undefined
```

## Object.keys

object의 enumerable **own-property들**의 이름들을 `string` 배열로 반환하는 static method로, 출력되는 순서는 `for-in` 문에 적용했을 때와 동일합니다.

```js
// simple array
const arr = ["a", "b", "c"];
console.log(Object.keys(arr)); // console: ['0', '1', '2']
```

```js
// array-like object
const obj = { 0: "a", 1: "b", 2: "c" };
console.log(Object.keys(obj)); // console: ['0', '1', '2']
```

```js
// array-like object with random key ordering
const anObj = { 100: "a", 2: "b", 7: "c" };
console.log(Object.keys(anObj)); // console: ['2', '7', '100']
```

```js
// getFoo is a property which isn't enumerable
const myObj = Object.create(
  {},
  {
    getFoo: {
      value: function () {
        return this.foo;
      },
    },
  }
);
myObj.foo = 1;
console.log(Object.keys(myObj)); // console: ['foo']
```

## Object.getOwnPropertyNames

주어진 object의 enumerable + symbol을 제외한 non-enumerable **own-property들**의 이름들을 `string` 배열로 반환하는 static method입니다.

반환되는 array 내의 enumerable property들의 순서는 `Object.keys()` 또는 `for-in` 문을 적용했을 때 접근하는 순서와 동일합니다.

ES6에서는 모든 non-integer property들은 삽입순서를 따릅니다.

```js
const arr = ["a", "b", "c"];
console.log(Object.getOwnPropertyNames(arr).sort());
// ["0", "1", "2", "length"]

// Array-like object
const obj = { 0: "a", 1: "b", 2: "c" };
console.log(Object.getOwnPropertyNames(obj).sort());
// ["0", "1", "2"]

Object.getOwnPropertyNames(obj).forEach((val, idx, array) => {
  console.log(`${val} -> ${obj[val]}`);
});
// 0 -> a
// 1 -> b
// 2 -> c

// non-enumerable property
const myObj = Object.create(
  {},
  {
    getFoo: {
      value() {
        return this.foo;
      },
      enumerable: false,
    },
  }
);
myObj.foo = 1;

console.log(Object.getOwnPropertyNames(myObj).sort()); // ["foo", "getFoo"]
```

## Object.getOwnPropertySymbols

object의 **own-symbol property들**만 추려서 array로 반환하는 static 메서드입니다.

```js
const obj = {};
const a = Symbol("a");
const b = Symbol.for("b");

obj[a] = "localSymbol";
obj[b] = "globalSymbol";

const objectSymbols = Object.getOwnPropertySymbols(obj);

console.log(objectSymbols.length); // 2
console.log(objectSymbols); // [Symbol(a), Symbol(b)]
console.log(objectSymbols[0]); // Symbol(a)
```

## Object Serialization

JS 객체를 네트워크를 통해 전송하거나 브라우저 스토리지 또는 로컬 디스크에 저장할 수 있는 format으로 변경하는 작업을 가리킵니다.

여기서 직렬화된 객체는 다시 원상복구가 되어야 합니다.

보통 `JSON.stringify` 메서드로 객체를 문자열 형식으로 바꾸고, 다시 복원할 때는 `JSON.parse` 메서드를 사용합니다.

## 참고자료

[Object.keys](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/keys)

[Object.getOwnPropertyNames](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyNames)

[Object.getOwnPropertySymbols](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertySymbols)