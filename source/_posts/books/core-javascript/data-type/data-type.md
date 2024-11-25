---
title: 데이터 타입 종류
date: 2024-11-01 00:37:26
categories:
  - Books
  - 코어 자바스크립트
#tags:
---

![데이터 타입 종류](/images/data_type.png)

## null

“없음”을 의미하는 primitive value로, 임의의 변수가 참조할 데이터가 없음을 나타낼 때 인위적으로 할당하는 값입니다.

`typeof` 연산자를 적용하면 object가 나오고 동등연산자(`==`)로 `undefined`와 같다고 나오므로 특정 변수의 값이 `null`인지 판별할 때는 일치연산자(`===`)를 사용해야 합니다.

```js
var n = null;
console.log(typeof n); // 'object'

console.log(n == undefined);
console.log(n == null);

console.log(n === undefined);
console.log(n === null);
```

## undefined

JS 엔진에 의해서 `undefined`가 반환되는 경우는 다음과 같습니다.

1. 선언만 하고 할당하지 않은(데이터 영역의 주소를 가지지 않은) 변수에 접근할 때

```js
var a;

console.log(a);
```

2. 객체 내부에 존재하지 않는 속성에 접근할 때

```js
var arr1 = [];
arr1.length = 3;
console.log(arr1); // [ <3 empty items> ]

var arr2 = new Array(3);
console.log(arr2); // [ <3 empty items> ]

console.log(arr1[2]); // undefined
console.log(arr2[0]); // undefined
```

3. return 문이 없거나 호출되지 않는 함수의 실행결과

```js
var func = function () {};
var c = func();

console.log(c);
```

아래 `arr3`처럼 인위적으로 명시하면 마치 `undefined`가 변수에 할당된 듯이 보이지만 실제로는 JS 엔진에 의해서 그냥 반환된 겁니다.

```js
var arr3 = [undefined, undefined, undefined];
console.log(arr3); // [ undefined, undefined, undefined ]
```

또한 인위적으로 할당된 `undefined`는 순회의 대상이 되기 때문에 `forEach`, `map`, `filter`, `reduce` 등 실제 배열의 순회를 기반으로 하는 메서드들을 적용하면 결과는 예상과 다르게 나옵니다.

```js
var arr1 = [undefined, 1];

arr1.forEach(function (v, i) {
  console.log(v, i);
}); // undefined 0

arr1.map(function (v, i) {
  return v + i;
}); // [ NaN, 2 ]

arr1.filter(function (v) {
  return !v;
}); // [ undefined ]

arr1.reduce(function (p, c, i) {
  return p + c + i;
}, ""); // 'undefined011'
```

```js
var arr2 = [];
arr2[1] = 1;

arr2.forEach(function (v, i) {
  console.log(v, i);
}); // 1 1

arr2.map(function (v, i) {
  return v + i;
}); // [ <1 empty item>, 2 ]

arr2.filter(function (v) {
  return !v;
}); // []

arr2.reduce(function (p, c, i) {
  return p + c + i;
}, ""); // '11'
```
