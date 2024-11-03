---
title: 6. computed
date: 2024-11-03 20:53:57
categories:
  - Studies
  - Vue3
  - Reactivity API
#tags:
---
Vue3에서는 컴포넌트를 떠나 "다른 state에 의존하는 getter"를 단독적으로 구현할 수 있는 computed 함수를 제공합니다.

computed 함수의 정의는 다음과 같습니다.

```js
function computed(getter) {
  let result = ref();
  effect(() => (result.value = getter())); // --- (*)

  return result;
}
```

```js
let product = reactive({ price: 5, quantity: 2 })

let salePrice = computed(() => product.price * 0.9)
let total = computed(() => salePrice.value * product.quality))

console.log(`total: ${total}, salePrice: ${salePrice}`);
// total: 10, salePrice: 4.5

product.quantity = 3;
// 첫 번째 effect만 실행

console.log(`total: ${total}, salePrice: ${salePrice}`);
// total: 13.5, salePrice: 4.5

product.price = 10;
// 첫 번째, 두 번째 effect 실행

console.log(`total: ${total}, salePrice: ${salePrice}`);
// total: 27, salePrice: 9
```

## 참고자료

[Gregg Pollack free Vue3 Reactivity Workshop](https://www.youtube.com/watch?v=BfDQD4Y6W8c)