---
title: 3. Proxy, Reflect
date: 2024-11-03 20:54:54
categories:
  - Studies
  - Vue3
  - Reactivity API
#tags:
---
![Proxy, Reflect](/images/proxy_reflect.png)

`Proxy`와 `Reflect`를 이용하면 객체의 속성을 읽을 때는 track, 쓸 때는 trigger 함수를 자동으로 실행하도록 구현할 수 있습니다.

`Proxy`의 get/set trap에 각각 `Reflect`의 정적 메소드와 track, trigger를 함께 작성하여 아래와 같이 구현할 수 있습니다.

```js
function reactive(obj) {
  const trap = {
    get(target, key, receiver) {
      track(target, key);
      return Reflect.get(...arguments);
    },
    set(target, key, value, receiver) {
      let oldValue = target[key];
      let result = Reflect.set(...arguments);
      if (result && oldValue != value) {
        trigger(target, key);
      }
    },
  };

  return new Proxy(obj, trap);
}
```

```js
let product = reactive({ price: 5, quantity: 2 });
let total = 0;

let effect = () => {
  total = product.price * product.quantity;
};

effect();
console.log("before updated quantity total = " + total); // 10

product.quantity = 3;
console.log("after updated quantity total = " + total); // 15
```

중첩된 객체(nested object)의 Proxy로 접근하면 내부 중첩된 객체들도 개별 proxy가 생성됩니다.

reactive object의 중첩된 객체도 reactive 함수에 wrapping되므로 targetMap에 추가됩니다.

```js
const handler = {
  get(target, property, receiver) {
    track(target, property);
    const value = Reflect.get(...arguments);
    if (isObject(value)) {
      // Wrap the nested object in its own reactive proxy
      return reactive(value);
    } else {
      return value;
    }
  },
  // ...
};
```

## 참고자료

[Gregg Pollack free Vue3 Reactivity Workshop](https://www.youtube.com/watch?v=BfDQD4Y6W8c)