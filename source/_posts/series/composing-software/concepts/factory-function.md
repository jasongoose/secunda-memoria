---
title: Factory Function
date: 2024-10-28 08:44:28
categories:
  - Series
  - Composing Software
  - Concepts
#tags:
---
`new` 키워드 없이 새로운 객체를 반환하는 함수로, 보통 OOP의 constructor의 대용으로 사용됩니다.

closure 현상을 이용하면 private 속성과 메서드를 생성하여 내부구현 로직이 사용자 영역에 노출되지 않도록 만들 수 있습니다.

새로운 객체 instance를 하나 만들어주는 것만으로 class나 prototype의 상속도 쉽게 구현할 수 있습니다.

```js
// Object Delegation
const withConstructor = (constructor) => (o) => ({
  // create the delegate [[Prototype]]
  __proto__: {
    // add the constructor prop to the new [[Prototype]]
    constructor,
  },
  // mix all o's props into the new object
  ...o,
});
```

```js
import withConstructor from "./with-constructor";

const pipe =
  (...fns) =>
  (x) =>
    fns.reduce((y, f) => f(y), x);
// or `import pipe from 'lodash/fp/flow';`

// Set up some functional mixins
const withFlying = (o) => {
  let isFlying = false;
  return {
    ...o,
    fly() {
      isFlying = true;
      return this;
    },
    land() {
      isFlying = false;
      return this;
    },
    isFlying: () => isFlying,
  };
};

const withBattery =
  ({ capacity }) =>
  (o) => {
    let percentCharged = 100;
    return {
      ...o,
      draw(percent) {
        const remaining = percentCharged - percent;
        percentCharged = remaining > 0 ? remaining : 0;
        return this;
      },
      getCharge: () => percentCharged,
      getCapacity: () => capacity,
    };
  };

// Object Concatenation
const createDrone = ({ capacity = "3000mAh" }) =>
  pipe(withFlying, withBattery({ capacity }), withConstructor(createDrone))({});
```

```js
const myDrone = createDrone({ capacity: "5500mAh" });

console.log(`
  can fly:  ${myDrone.fly().isFlying() === true}
  can land: ${myDrone.land().isFlying() === false}
  battery capacity: ${myDrone.getCapacity()}
  battery status: ${myDrone.draw(50).getCharge()}%
  battery drained: ${myDrone.draw(75).getCharge()}% remaining
`);

console.log(`
  constructor linked: ${myDrone.constructor === createDrone}
`);
```
