---
title: 4. activeEffect
date: 2024-11-03 20:53:49
categories:
  - Studies
  - Vue3
  - Reactivity API
#tags:
---
앞에서 살펴보았듯이 effect를 실행하면 reactive object의 속성들을 읽어내고 해당 속성의 dep에 실행한 effect가 추가됩니다.

하지만 reactive object의 속성을 읽으려고 하면 앞서 정의한 effect가 dep에 있음에도 불구하고 targetMap, depsMap, dep을 거쳐서 동일한 effect를 추가하려고 합니다.

```js
let product = reactive({ price: 5, quantity: 2 });
let total = 0;
let effect = () => {
  total = product.price * product.quantity;
};

effect(); // price의 dep과 quantity의 dep에 effect가 추가됩니다.
console.log(total); // 10

product.quantity = 3; // quantity dep에 trigger가 발생합니다.

// 이미 quantity dep에 effect가 있어도, track이 수행되면서 동일한 effect를 추가하려고 합니다.
console.log(product.quantity);
console.log(total); // 15
```

이와 같은 중복을 막기 위해서는 오직 effect가 발생할 경우에만 track이 수행되어야 하는데, 여기서 Vue3는 activeEffect라는 전역변수를 도입합니다.

activeEffect는 현재 진행되고 있는 effect를 가리키는 변수입니다.

```js
let activeEffect = null; // the active effect which is running
... // active, track, trigger definitions~

function effect(eff) {
	activeEffect = eff // Set this as the activeEffect
	activeEffect() // Run it --- (*)
	activeEffect = null // Unset it
}

// (*) activeEffect를 호출하면 콜백으로 전달된 eff가 실행되고 track이 수행된다.
```

여기서 activeEffect가 존재하는 경우에 track에 의해서 dep에 추가되도록 track의 정의도 수정할 필요가 있습니다.

```js
const targetMap = new WeakMap();

function track(target, key) {
  if (activeEffect) {
    let depsMap = targetMap.get(target);
    if (!depsMap) {
      targetMap.set(target, (depsMap = new Map()));
    }

    let dep = depsMap.get(key);
    if (!dep) {
      depsMap.set(key, (dep = new Set()));
    }

    dep.add(activeEffect); // effect에 의해서 activeEffect만 dep에 추가된다.
  }
}
```

activeEffect를 적용하여 수정하면 다음과 같습니다.

```js
let product = reactive({ price: 5, quantity: 2 }); // product는 Proxy임을 기억하자.
let salePrice = 0;
let total = 0;

effect(() => {
  total = product.price * product.quantity;
});
// effect의 실행으로 product.price와 product.quantity의 dep으로 effect가 추가되고
// total 값이 업데이트 된다.

effect(() => {
  salePrice = product.price * 0.9;
});
// effect의 실행으로 product.price의 dep으로만 effect가 추가되고
// salePrice 값이 업데이트 된다.
```

```js
console.log(`total: ${total}, salePrice: ${salePrice}`);
// total: 10, salePrice: 4.5

product.quantity = 3;
// 첫 번째 effect 실행

console.log(`total: ${total}, salePrice: ${salePrice}`);
// total: 15, salePrice: 4.5

product.price = 10;
// 첫 번째, 두 번째 effect 실행

console.log(`total: ${total}, salePrice: ${salePrice}`);
// total: 30, salePrice: 9

// property를 읽으려고 해도 activeEffect가 null이므로 불필요한 track이 발생하지 않습니다.
console.log(product.price);
console.log(product.quantity);
```
