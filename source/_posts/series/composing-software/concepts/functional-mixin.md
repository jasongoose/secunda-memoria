---
title: Functional Mixin
date: 2024-10-28 08:44:55
categories:
  - Series
  - Composing Software
  - Concepts
#tags:
---
## Mixin

[object concatenation](../object-composition#concatenation)의 단위 객체로, 이들을 결합하여 개별 속성과 메서드들을 하나의 합성객체에서 사용할 수 있습니다.

```js
const chocolate = {
  hasChocolate: () => true,
};

const caramelSwirl = {
  hasCaramelSwirl: () => true,
};

const pecans = {
  hasPecans: () => true,
};
```

```js
const iceCream = Object.assign({}, chocolate, caramelSwirl, pecans);
/*
// or, if your environment supports object spread...
const iceCream = {...chocolate, ...caramelSwirl, ...pecans};
*/
console.log(`
  hasChocolate: ${iceCream.hasChocolate()}
  hasCaramelSwirl: ${iceCream.hasCaramelSwirl()}
  hasPecans: ${iceCream.hasPecans()}
`);
```

## Functional Inheritance

함수를 사용하여 특정 객체의 속성과 메서드를 상속시키는 기법입니다.

```js
// Base object factory
function base(spec) {
  var that = {}; // Create an empty object
  that.name = spec.name; // Add it a "name" property
  return that; // Return the object
}

// Construct a child object, inheriting from "base"
function child(spec) {
  // Create the object through the "base" constructor
  var that = base(spec);
  that.sayHello = function () {
    // Augment that object
    return "Hello, I'm " + that.name;
  };
  return that; // Return it
}
```

```js
// Usage
var result = child({ name: "a functional object" });
console.log(result.sayHello()); // "Hello, I'm a functional object"
```

## Functional Mixin

기존의 객체를 인자로 받아서 특정 속성이나 메서드를 append하여 확장(= concatenate)된 shallow-copy를 반환하는 factory function입니다.

```js
const flying = (o) => {
  let isFlying = false;
  return Object.assign({}, o, {
    fly() {
      isFlying = true;
      return this;
    },
    isFlying: () => isFlying,
    land() {
      isFlying = false;
      return this;
    },
  }); // 세번째 인자가 바로 mixin 입니다!
};
```

```js
const bird = flying({});
console.log(bird.isFlying()); // false
console.log(bird.fly().isFlying()); // true
```

Functional Mixin의 사용이점들은 다음과 같습니다.

- closure 현상으로 private 속성을 추가할 수 있다.
- [compose](../curry-function#compose)나 [pipe](../curry-function#pipe)를 사용하여 합성할 수 있다.
- class 기반 상속과는 달리 root(base) class 없이 다수의 객체로부터 여러 속성과 기능들로 확장할 수 있다.

단, 유의할 점들이 있습니다.

- 인자로 받은 객체에 mutation이 발생하여 side-effect가 발생할 수 있다.
- Mixin으로 Functional Inheritance를 구현한 사례이므로 기존 class inheritance의 단점을 가질 수 있다.
