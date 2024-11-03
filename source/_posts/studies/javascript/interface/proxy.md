---
title: Proxy
date: 2024-11-03 16:50:15
categories:
  - Studies
  - JavaScript
  - Interface
#tags:
---
원본 객체의 object internal method의 동작방식을 재정의(intercept, redefine)하기 위한 interface를 제공합니다.

보통 객체의 속성에 접근하면서 로그를 남기거나, 값을 검증하거나, 값을 sanitize하거나(trim 같은), 출력 포맷을 지정하는 등의 작업에 사용됩니다.

Proxy에 의한 interception은 2가지 경우에 발생합니다.

- Proxy에 대해서 trap을 수행하는 경우
- trap에 대응되는 `Reflect` 메서드를 Proxy에 대해 실행하는 경우

## Object Internal Method

JS는 기본적으로(언어 자체에서) 컴퓨터가 객체의 속성을 직접 read/write하는 것을 방지하기 위해서 객체 속성과의 상호작용을 수행하는 메서드들이 내장되어 있는데 이들을 [object internal method](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy#object_internal_methods)라고 합니다.

지금까지 당연하게 사용해온 객체에 대한 연산들은 일련의 internal method들을 호출하여 구현됩니다.

## Trap

Proxy를 걸쳐서 object internal method의 동작방식이 원본과는 다른 객체를 Exotic Object라고 부르고 재정의 대상이 되는 internal method를 "Trap"이라고 부릅니다.

### handler.get

Proxy를 통해 target 객체의 특정 속성을 read할 때, 수행되는 trap으로 아래와 같은 매개변수를 가집니다.

```js
const proxy = new Proxy(obj, {
  get(target, property, receiver) {},
});

// target: obj와 동일한 객체(===)
// property : lookup 대상 key
// receiver : lookup을 시작한 객체
```

```js
const obj = {
  log: [1, 2, 3, 4],
  get latest() {
    // getter
    return this.log[this.log.length - 1];
  },
};

const proxy = new Proxy(obj, {
  get(target, key, receiver) {
    return receiver;
  },
});

console.log(proxy.latest); // proxy
```

```js
const inherited = Object.create(proxy);

console.log(inherited.latest); // inherited
```

여기서 receiver는 target의 `getter`, `setter` 내부 `this`에 binding되는 객체를 가르키는데, 가능한 값으로 2가지가 됩니다.

- Proxy 자체
- Proxy를 prototype으로 가지는 객체

### handler.set

Proxy를 통해 target 객체의 특정 속성을 write할 때, 수행되는 trap으로 아래와 같은 매개변수를 가집니다.

```js
const proxy = new Proxy(obj, {
  set(target, property, value, receiver) {},
});

// target: obj와 동일한 객체(===)
// property : write 대상 key
// value: write되는 value
// receiver : write를 시작한 객체
```

```js
const obj = {
  set current(name) {
    this.log.push(name);
  },
  log: [],
};

const proxy = new Proxy(obj, {
  set(target, key, value, receiver) {
    console.log(receiver);
  },
});

proxy.current = "EN"; // proxy
```

```js
const inherited = Object.create(proxy);
inherited.current = "hello"; // inherited
```

## 참고자료

[Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy)