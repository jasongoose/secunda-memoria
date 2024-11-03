---
title: 5. ref
date: 2024-11-03 20:55:01
categories:
  - Studies
  - Vue3
  - Reactivity API
#tags:
---
activeEffect 페이지의 첫번째 effect를 다음과 같이 수정해보면 어떨까?

```js
let product = reactive({ price: 5, quantity: 2 });
let salePrice = 0;
let total = 0;

effect(() => {
  //total = product.price * product.quantity
  total = salePrice * product.quantity;
});

effect(() => {
  salePrice = product.price * 0.9;
});
```

위 코드를 실행하면 첫 번째 effect에 의해서 total 값이 0이 되고 두 번째 effect에 의해서 salePrice의 값은 4.5가 됩니다.

하지만 salePrice라는 변수 자체는 reactive하지 않기 때문에 두 번째 effect에 의해 salePrice 값이 변해도 첫 번째 effect가 저절로 다시 실행되는 trigger가 발생하지 않습니다.

> "변수 x는 reactive하다." : x의 값이 변할 때마다 x의 dep에 포함된 모든 effect들이 다시 계산된다.

Vue3에서는 primitive value에 대해서 dep을 정의할 수 있도록 ref라는 함수를 제공합니다.

공식문서만 읽으면 ref의 정의를 아래와 같이 상상할 수 있지만 이는 완전히 잘못된 정의입니다.

```js
function ref(initialValue) {
  return reactive({ value: initialValue });
}
```

실제 github 상에서 정의는 JS의 Object Accesssor를 사용합니다.

```js
function ref(raw) {
  const r = {
    get value() {
      track(r, "value"); // activeEffect만 실행합니다.
      return raw;
    },
    set value(newVal) {
      raw = newVal;
      trigger(r, "value");
    },
  };
  return r;
}

const R = ref(0);
// closure 현상으로 ref 함수를 종료해도 raw값은 계속 참조할 수 있습니다.

effect(() => {
	... = R.value + ...
});
// 1. activeEffect의 실행으로 track에 의해서 WeakMap에 key R 생성
// 2. R depsMap 생성
// 3. key인 'value'에 대한 dep 생성(이미 있다면 1~3은 생략)
// 4. 해당 dep에 effect 추가

R.value = newVal;
// 1. ref setter 수행
// 2. raw에 newVal 할당
// 3. trigger에 의해서 업데이트된 raw를 기반으로 R.value의 모든 effect들을 실행
```


## 참고자료

[Gregg Pollack free Vue3 Reactivity Workshop](https://www.youtube.com/watch?v=BfDQD4Y6W8c)