---
title: 1. Multiple Properties
date: 2024-11-03 20:54:14
categories:
  - Studies
  - Vue3
  - Reactivity API
#tags:
---

![Multiple Properties](/images/multiple_properties.jpg)

하나의 객체에는 여러 개의 속성들을 가지므로 속성별 effect들도 따로 관리할 필요가 있습니다.

여기서 각 속성에 대응되는 dep을 mapping해주는 dependency map(depsMap)을 이용하면 됩니다.

depsMap은 key-value pair를 가지는 Map으로, key는 reactive object(반응성이 부여된 객체)의 개별 속성의 이름이고 value는 해당 속성의 effect들을 저장한 dep입니다.

위 내용을 바탕으로 depsMap을 사용하여 track, trigger를 구현하면 다음과 같습니다.

```js
const depsMap = new Map();

function track(key) {
  // Make sure this effect is being tracked.
  // Get the current dep (effects) that need to be run when this key (property) is set
  let dep = depsMap.get(key);

  if (!dep) {
    // There is no dep (effects) on this key yet
    depsMap.set(key, (dep = new Set())); // Create a new effect Set
  }
  dep.add(effect); // Add effect to dep
}

function trigger(key) {
  // Get the dep (effects) associated with this key
  let dep = depsMap.get(key);

  if (dep) {
    // If they exist
    dep.forEach((effect) => {
      // run them all
      effect();
    });
  }
}
```

```js
let product = { price: 5, quantity: 2 };
let total = 0;

let effect = () => {
  total = product.price * product.quantity;
};

track("quantity");
effect();
console.log(total); // 10

product.quantity = 3;
trigger("quantity");
console.log(total); // 15
```
