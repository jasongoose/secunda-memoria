---
title: Set
date: 2024-11-03 16:18:29
categories:
  - Studies
  - JavaScript
  - Data Structure
#tags:
---
reference 타입 중 하나인 Set은 임의의 타입(primitive + reference)의 값들을 중복없이 가지는 object입니다.

Set의 요소들간의 중복여부는 일치연산자(===)의 알고리즘을 따릅니다.

순회순서는 Map처럼 요소 삽입순서를 따르지만 요소는 key-value pair로 가지지 않습니다.

Set에는 `null`, `NaN`, `undefined`도 저장할 수 있습니다.

여기서 `NaN` 같은 경우, 아래와 같은 성질이 있지만 중복될 수 없습니다.

```js
console.assert(NaN !== NaN);
```

## Using Set

```js
const mySet1 = new Set();

mySet1.add(1); // Set [ 1 ]
mySet1.add(5); // Set [ 1, 5 ]
mySet1.add(5); // Set [ 1, 5 ]
mySet1.add("some text"); // Set [ 1, 5, 'some text' ]

const o = { a: 1, b: 2 };
mySet1.add(o);

// 여기서 o와 다른 객체를 참조하기 때문에 그대로 insert할 수 있습니다.
mySet1.add({ a: 1, b: 2 });

mySet1.delete(5); // removes 5 from the set

console.log(mySet1); // Set(4) [ 1, "some text", {a: 1, b: 2}, {a: 1, b: 2} ]
```

Set 내의 특정 값이 존재하는지 여부를 알아낼 때는 `Set.prototype.has` 메서드를 사용하면 됩니다.

해당 메서드는 동일한 개수, 종류의 요소를 가지는 배열에 대해서 `Array.prototype.includes` 메서드를 적용하는 것보다 더 빠른 속도를 가집니다.

## Relation with String

```js
let text = "India";

const mySet = new Set(text); // Set(5) {'I', 'n', 'd', 'i', 'a'}
mySet.size; // 5

//case sensitive & duplicate omission
new Set("Firefox"); // Set(7) { "F", "i", "r", "e", "f", "o", "x" }
new Set("firefox"); // Set(6) { "f", "i", "r", "e", "o", "x" }
```

## Array

```js
let myArray = ['value1', 'value2', 'value3']

// Array -> Set
let mySet = new Set(myArray)
mySet.has('value1')     // returns true


// Set -> Array
[...mySet]
Array.from(mySet)
```

## Iterating Set

```js
// iterate over items in set
// logs the items in the order: 1, "some text", {"a": 1, "b": 2}, {"a": 1, "b": 2}
for (let item of mySet1) console.log(item);

// logs the items in the order: 1, "some text", {"a": 1, "b": 2}, {"a": 1, "b": 2}
for (let item of mySet1.keys()) console.log(item);

// logs the items in the order: 1, "some text", {"a": 1, "b": 2}, {"a": 1, "b": 2}
for (let item of mySet1.values()) console.log(item);

// logs the items in the order: 1, "some text", {"a": 1, "b": 2}, {"a": 1, "b": 2}
// (key and value are the same here)
for (let [key, value] of mySet1.entries()) console.log(key);

// Iterate set entries with forEach()
mySet1.forEach((value) => {
  console.log(value);
});
```

## Remove duplicates from the array

```js
const numbers = [2, 3, 4, 4, 2, 3, 3, 4, 4, 5, 5, 6, 6, 7, 5, 32, 3, 4, 5];

console.log([...new Set(numbers)]); // [2, 3, 4, 5, 6, 7, 32]
```

## Ensure the uniqueness of the array

```js
const array = Array.from(document.querySelectorAll("[id]")).map((e) => e.id);

const set = new Set(array);
console.assert(set.size == array.length);
```

## 참고자료

[Set](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set)