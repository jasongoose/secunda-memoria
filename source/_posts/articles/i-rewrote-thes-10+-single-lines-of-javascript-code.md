---
title: I Re-Wrote These 10+ Single Lines of JavaScript Code, the Team Lead Praised the Code for Being Elegant
date: 2024-11-01 22:14:44
categories:
  - Articles
#tags:
---
## Array Deduplication

```js
// Normal
const uniqueArr = (arr) => [...new Set(arr)];
```

```js
// Better
const removeDuplicateStrings = (array) => {
  const uniqueValues = [];
  const seenMap = {};

  for (const item of array) {
    if (seenMap[item]) continue;
    seenMap[item] = true;
    uniqueValues.push(item);
  }

  return uniqueValues;
};
```

## Get Parameters from a URL and Convert Them to Object

```js
// Normal
const getParameters = URL => JSON.parse(`{"${decodeURI(URL.split("?")[1]).replace(/"/g, '\\"').replace(/&/g, '","').replace(/=/g, '":"')}"}`
```

```js
// Better
const getURLParameters = (url) => {
  const { searchParams } = new URL(url);
  return Object.fromEntries(searchParams);
};
```

## Check Whether the Object is Empty

```js
// Normal
const isEmpty = (obj) =>
  Reflect.ownKeys(obj).length === 0 && obj.constructor === Object;
```

```js
// Better
const isObjectEmpty = (object) => {
  if (object.constructor !== Object) return false;
  // Iterates over the keys of an object, if
  // any exist, return false.
  for (_ in object) return false;
  return true;
};
```

## Reverse the String

```js
// Normal
const reverse = (str) => str.split("").reverse().join("");
```

```js
// Better
const reverseString = (string) => {
  let reversedString = "";

  for (let i = string.length - 1; i >= 0; i--)
    reversedString += string[i];

  return reversedString;
};
```

## Generate Random Hex

```js
// Normal
const randomHexColor = () =>
  `#${Math.floor(Math.random() * 0xffffff)
    .toString(16)
    .padEnd(6, "0")}`;
```

```js
// Better
const getRandomHexColor = () => {
  const randomValue = Math.floor(Math.random() * 0xffffff);
  const hexValue = randomValue.toString(16);
  return hexValue.padStart(6, "0");
};
```

## 출처

[I Re-Wrote These 10+ Single Lines of JavaScript Code, the Team Lead Praised the Code for Being Elegant](https://levelup.gitconnected.com/i-re-wrote-these-10-single-lines-of-javascript-code-the-team-lead-praised-the-code-for-being-668ade4fea71)
