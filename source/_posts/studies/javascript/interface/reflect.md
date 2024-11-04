---
title: Reflect
date: 2024-11-03 16:50:23
categories:
  - Studies
  - JavaScript
  - Interface
#tags:
---
JS에서 intercept가 가능한 연산들을 static 메서드로 가지는 built-in 객체로, 함수가 아니므로 생성자 함수처럼 사용하면 안됩니다.

`Reflect` 메서드들은 proxy trap들과 일대일 대응되어서 주로 target 객체의 원본(proxy에 의해서 intercept되기 전) internal method를 수행할 때 사용됩니다.

```js
const target = {
  message1: "hello",
  message2: "everyone",
};

const handler3 = {
  get(target, prop, receiver) {
    if (prop === "message2") {
      return "world";
    }
    return Reflect.get(...arguments);
  },
};
```

```js
const proxy3 = new Proxy(target, handler3);

console.log(proxy3.message1); // hello
console.log(proxy3.message2); // world
```

## Better syntax

기존의 syntax보다 가독성을 높일 수 있습니다.

```js
Reflect.has(obj, name); // (name in object)
Reflect.deleteProperty(obj, name); // delete obj[name]
```

함수의 this-binding도 다음과 같은 syntax로 작성할 수 있습니다.

```js
Reflect.apply(f, obj, args); // Function.prototype.apply(f, obj, args);
```

생성자 함수를 호출하여 instance를 생성하는 부분도 아래와 같이 작성할 수 있습니다.

```js
const obj = Reflect.construct(F, args); // new F(...args);
```

## More useful return values

`ES5` 스펙 상에 정의된 static 메서드들 중 하나인, `Object.defineProperty` 를 예시로 들어봅시다.

보통 위 메서드의 예외처리는 `try/catch` 문으로 수행합니다.

```js
try {
  Object.defineProperty(obj, name, desc);
  // property defined successfully
} catch (e) {
  // possible failure (and might accidentally catch the wrong exception)
}
```

`Object.defineProperty` 메서드는 property를 정의하는 과정에서 잘못되면 `TypeError` 가 발생하는데, 위와 같은 예외처리를 하지 않으면 중간에 실행이 멈출 수도 있습니다.

하지만 `Reflect.defineProperty` 메서드를 사용하면 property를 정의하는 연산의 마무리 여부를 `boolean` 타입으로 반환해서 따로 예외처리 없이 유연한 코드를 작성할 수 있습니다.

```js
if (Reflect.defineProperty(obj, name, desc)) {
  // success
} else {
  // failure
}
```

위와 같은 `boolean` 타입을 반환하는 메서드들은 다음이 있다.

- `Reflect.set`
- `Reflect.deleteProperty`
- `Reflect.preventExtensions`
- `Reflect.setPrototypeOf`

## Control the this-binding of accessors

`Reflect.get` , `Reflect.set`의 마지막 인자인 `receiver` 을 사용하면 `Object`의 `getter`, `setter` 내부의 `this`를 다른 객체로 바인딩할 수 있습니다.

```js
const name = ... // get property name as a string
const obj = {
  get foo() { return this.bar(); },
  bar: function() { ... }
}

// if obj[name] is an accessor, it gets run with `this === wrapper`
Reflect.get(obj, name, wrapper)

Reflect.set(obj, name, value, wrapper)
```

## Prevent circular reference

아래와 같이 receiver가 proxy인 상태로 잘못 작성하면 순환참조가 일어날 수 있습니다.

```js
const obj = {
  record: [],
  get log() {
    return this.record;
  },
  set logger(newVal) {
    this.log.push(newVal);
  },
};

const proxy = new Proxy(obj, {
  get(target, key, receiver) {
    return receiver[key];
  },
  set(target, key, value, receiver) {
    receiver[key] = value;
  },
});

proxy.logger = 1; // RangeError: Maximum call stack size exceeded
proxy.log;
```

여기서 Reflect 메서드를 사용하면 순환참조를 막을 수 있습니다.

```js
// ...
const proxy = new Proxy(obj, {
  get(target, key, receiver) {
    return Reflect.get(...arguments);
  },
  set(target, key, value, receiver) {
    Reflect.set(...arguments);
  },
});

proxy.logger = 1;
proxy.log; // [1]
// ...
```
