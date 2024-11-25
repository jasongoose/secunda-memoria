---
title: Multiple Reactive Objects
date: 2024-11-03 20:54:14
categories:
  - Studies
  - Vue3
  - Reactivity API
#tags:
---
![Multiple Reactive Objects](/images/multiple_reactive_objects.jpg)

reactive object의 개수가 늘어났을 때는 객체별 effect를 관리해야 하는데, 여기서 객체를 key로 가지는 WeakMap인 targetMap을 사용합니다.

targetMap의 key는 reactive object, value는 depsMap을 가지면 됩니다.

위 내용을 바탕으로 track, trigger를 구현하면 다음과 같습니다.

```js
// targetMap stores the effects that each object should re-run when it's updated
const targetMap = new WeakMap();

function track(target, key) {
  // We need to make sure this effect is being tracked.
  // Get the current depsMap for this target
  let depsMap = targetMap.get(target);

  if (!depsMap) {
    // There is no map.
    targetMap.set(target, (depsMap = new Map())); // Create one
  }

  // Get the current dependencies (effects) that need to be run when this is set
  let dep = depsMap.get(key);

  if (!dep) {
    // There is no dependencies (effects)
    depsMap.set(key, (dep = new Set())); // Create a new Set
  }

  dep.add(effect); // Add effect to dependency map
}

function trigger(target, key) {
  // Does this object have any properties that have dependencies (effects)
  const depsMap = targetMap.get(target);

  if (!depsMap) {
    return;
  }
  // If there are dependencies (effects) associated with this
  let dep = depsMap.get(key);

  if (dep) {
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

track(product, "quantity");
effect();
console.log(total); // 10

product.quantity = 3;
trigger(product, "quantity");
console.log(total); // 15
```
