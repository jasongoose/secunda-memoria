---
title: 2-1장. 알아둬야 할 자바스크립트
date: 2024-10-29 23:53:18
categories:
  - Books
  - Node.js 교과서(개정 3판)
  - Chapter 2
#tags:
---
## Promise

Node.js v16부터는 reject된 Promise에 catch를 달지 않으면 `UnhandledPromiseRejection` 에러가 발생합니다.

```js
try {
  Promise.reject("에러");
} catch (err) {
  console.error(err); // UnhandledPromiseRejection 에러 발생
}
```

### async/await

Node.js v7.6부터 지원되는 기능으로 ES2017에 추가되었습니다.

for 문과 함께 async/await을 같이 써서 Promise를 순차적으로 실행할 수 있는데, Node.js v10부터 지원하는 ES2018 문법입니다.

```js
const promise1 = Promise.resolve("성공1");
const promise2 = Promise.resolve("성공2");

(async () => {
  for await (promise of [promise1, promise2]) {
    console.log(promise);
  }
})();
```

## Map

속성들 간의 순서를 보장하고 반복문을 사용할 수 있습니다.

속성명으로 문자열이 아닌 값도 사용할 수 있고 `size` 메서드를 통해 속성의 수를 쉽게 알 수 있습니다.

## Set

배열 자료구조를 사용하고 싶은데 중복을 허용하고 싶지 않을 때 유용합니다.

## nullish coalescing(??) / optional chaining(?.)

ES2020에서 추가된 `??` 연산자와 `?.` 연산자입니다.
