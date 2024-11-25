---
title: Saving Code To Run Later
date: 2024-11-03 20:53:49
categories:
  - Studies
  - Vue3
  - Reactivity API
#tags:
---
Vue3의 반응성은 3가지 함수와 dependency(dep)에 의해서 구현됩니다.

![Saving Code To Run Later](/images/saving_code_to_run_later.png)

## effect

특정 데이터 x가 변했을 때 다시 실행시켜야 코드 즉, x에 의존하는 연산을 수행하는 wrapper function입니다.

data fetching을 위해 비동기 작업이 위치하기도 합니다.

## dependency(dep)

특정 데이터 x의 effect들을 저장하는 Set 자료구조입니다.

Set을 사용하는 이유는 가리키는 effect들 간의 중복을 방지하기 위해서입니다.

## track

데이터 x의 effect를 dep에 추가하는 함수입니다.

## trigger

업데이트된 x를 기준으로 데이터 x의 dep에 저장된 모든 effect들을 실행하는 함수입니다.

위 내용을 바탕으로 dep을 사용하여 track, trigger를 구현하면 다음과 같습니다.

```js
let product = { price: 5, quantity: 2 };
let total = 0;
let dep = new Set();

function track() {
  dep.add(effect);
}

function trigger() {
  dep.forEach((effect) => effect());
}

let effect = () => {
  total = product.price * product.quantity;
};
```

```js
track(); // 먼저 effect를 dep에 추가하여 추적을 시작합니다.
effect(); // 첫 번째 total을 구합니다.

product.price = 20;
console.log(total); // 10 - total 값은 price에 의존하지만 아직 변하지 않았습니다.

trigger(); // price 값의 변화를 반영하기 위해서 dep에 있는 effect를 실행합니다.
console.log(total); // 40 - 그럼 price 값이 반영되어 total 값도 변합니다.
```
