---
title: Syntatic Sugar
date: 2024-11-03 17:53:24
categories:
  - Studies
  - JavaScript
  - Syntatic Sugar
#tags:
---
## -1 bit

`String.prototype.indexOf` 메서드는 인자로 전달한 문자가 존재하지 않으면 -1을 반환합니다.

-1일 때 `false`가 나오도록 하고 싶다면 `bitwise NOT`(~) 연산자로 -1(all bit 1)을 숫자 0(all bit 0)으로 바꾸면 됩니다.

```js
return `${url}${~url.indexOf("?") ? "&" : "?"}${params}`;
```

## Array Intersection

두 배열 arrX, arrY가 있을 때 arrX를 기준으로 arrY에도 있는 요소들을 필터링 하는 함수를 다음과 같이 구현할 수 있습니다.

```js
function intersect(arrX, arrY) {
  return arrX.filter(Set.prototype.has, new Set(arrY));
}
// filter의 2번째 인자는 filter callbackFn의 this로 참조된다.
```

위 코드는 아래 코드로도 작성할 수 있습니다.

```js
function intersect(arrX, arrY) {
  const setY = new Set(arrY);
  return arrX.filter((el) => setY.has(el));
}
```

## async, await

`async` 함수는 `Promise` 인스턴스를 반환합니다.

`async` 함수를 호출하여 연산 중간에 `await` 키워드를 만나면 `await` 뒤에 있는 `Promise` 인스턴스가 `settle` 될 때까지 해당 함수의 실행은 중지되는데,

**중지된다고 해서 전체 코드의 실행이 멈춘다는 의미가 절대 아닙니다!!**

`async` 함수가 중지되면 해당 함수를 호출했던 context로 pending Promise를 반환하여 기존 로직을 다시 실행합니다.

그러다가 중간에 `async` 함수 내의 `await` 문에서 `settle` 된다면 다시 `async` 함수의 실행을 재개합니다.

즉, `await` 문에서 코드 실행 flow가 2가지로 나뉘는 것으로도 이해할 수 있습니다.

## Computed Property

객체의 속성으로 접근하기 위한 key값은 굳이 상수일 필요는 없습니다.

```js
const user = {
  userName: "ehco",
  avatar: "echo.png",
  setUserName(userName) {
    this.userName = userName;
    return this;
  },
};

const key = "avatar";
console.log(user[key]); // "echo.png"
```

또한 객체를 생성하기 위한 key값도 상수일 필요는 없습니다.

```js
const arrToObj = ([key, value]) => ({ [key]: value });

console.log(arrToObj(["foo", "bar"])); // { "foo": "bar" }
```

## Default Parameters

ES6 이후부터 함수의 인자를 전달하지 않았을 때, 기본적으로 지정할 값을 전달할 수 있습니다.

여기서 인자의 타입이 객체인 경우에는 다음과 같이 빈 객체를 전달해야 작성할 수 있습니다.

```js
const createUser = ({ userName = "Anonymous", avatar = "anon.png" } = {}) => ({
  userName,
  avatar,
});

console.log(
  // { userName: "echo", avatar: 'anon.png' }
  createUser({ userName: "echo" }),
  // { userName: "Anonymous", avatar: 'anon.png' }
  createUser()
);
```

## every, some

집합이론을 기반으로 두 메서드의 관계를 정리하면 다음과 같습니다.

```js
!array.some((x) => ... === ...)  <=>  array.every((x) => ... !== ...)
```

```js
array.some((x) => ... !== ...)  <=>  !array.every((x) => ... === ...)
```

## Filter Falsy Values

JS 배열 내에서 `null`, `undefined` 같은 falsy value들을 필터링할 때는 항상 아래와 같이 써왔습니다.

```js
const filtered = array.filter((x) => !!x);
```

하지만 `Boolean` 생성자를 함수로 사용하면 훨씬 더 간결하게 작성할 수 있습니다.

```js
const array = [{ good }, null, { great }, undefined];

const truthyArray = array.filter(Boolean);
// truthyArray = [{ good }, { great }]
```

## How to check empty array?

변수에 할당된 값이 배열이 빈 배열인지 여부를 판별할 때는 2가지를 고려해야 합니다.

- `length` 속성이 존재하면서 값이 0인가?
- 확인할 대상이 유사배열객체가 아닌 `Array`의 인스턴스인가?

2가지 사항을 고려한 조건문은 다음과 같이 정리할 수 있습니다.

```js
const sampleArr = [ ... ];
if (sampleArr.length === 0 && Array.isArray(sampleArr)) {
	// ...
}
```

## String To Array

```js
const str = "apple";
console.log([...str]);
// [ 'a', 'p', 'p', 'l', 'e' ]
```

## Truncating, Expanding Array

어떤 배열이 있을 때 앞에서 몇번째 원소까지만 필요한 경우, 보통 `Array.prototype.slice` 메서드로 shallow-copy를 만들면 됩니다.

원본 배열을 직접 접근하기에 권장되지는 않지만 더 간단한 방법으로 Array만이 가진 `length` 속성을 더 작은 값으로 수정하면 됩니다.

```js
const arr = [1, 2, 3, 4, 5];

arr.length = 3;

console.log(arr); // [1,2,3]
console.log(arr[4]); // undefined
```

반대로 기존 배열의 크기를 늘리고 싶다면 `length` 속성을 더 크게 수정하면 됩니다.

```js
const arr = [1, 2];

arr.length = 4;

console.log(arr); // [1, 2, empty * 2] -> empty slot 2개 추가
console.log(arr[4]); // undefined
```

## 참고자료

[await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await)

[How to check if an array is empty or exists?](https://stackoverflow.com/questions/11743392/how-to-check-if-an-array-is-empty-or-exists)
