---
title: Lens
date: 2024-10-29 22:12:03
categories:
  - Books
  - Composing Software
  - Data Structure
#tags:
---
## Lens란?

임의의 object(store)의 특정 필드(part)에 초점을 두면서 합성이 가능하고 순수한 getter(view)와 setter(set) pair를 가진 interface입니다.

### view

현재 store에서 Lens의 대상이 되는 part 값을 반환하는 함수입니다.

### set

현재 store의 part를 다른 값으로 업데이트한 새로운(= shallow copy된) store를 반환하는 함수입니다.

## 사용이점

Lens는 store의 shape(type, interface)와 독립적으로 적용할 수 있다는 특징이 있습니다.

그래서 코드에서 특정 state(store)의 type이 바뀌었다면 state에 의존하는 모든 부분을 바꿀 필요없이 state에서 사용하는 Lens의 정의만 수정하면 됩니다.

실무에서는 [Ramda](https://ramdajs.com/docs/)와 같이 검증된 라이브러리를 사용하는 것을 권장합니다.

## Lens Laws

Lens는 일련의 공리들을 만족해야 하는데 이를 "Lens Laws"라고 합니다.

### 첫번째 공리

store의 part를 value로 수정하고 바로 view로 해당 part를 읽으면 value가 반환한다.

```js
view(lens, set(lens, value, store)) === value;
```

### 두번째 공리

store의 part를 a 그 다음에 b를 수정한 store 상태는 처음 store의 상태와 동일하다.

```js
set(lens, b, set(lens, a, store)) === set(lens, b, store);
```

### 세번째 공리

store로부터 part 값을 읽어서 그대로 part를 수정한 뒤의 store의 상태는 처음 store의 상태와 동일하다.

```js
set(lens, view(lens, store), store) === store;
```

위 법칙에 맞춰서 Lens를 구현하면 다음과 같습니다.

```js
const view = (lens, store) => lens.view(store);
const set = (lens, value, store) => lens.set(value, store);

const lensProp = (prop) => ({
  view: (store) => store[prop],
  set: (value, store) => ({
    ...store,
    [prop]: value,
  }),
});
```

```js
const fooStore = {
  a: "foo",
  b: "bar",
};

const aLens = lensProp("a");
const bLens = lensProp("b");
// aLens, bLens는 각각 property a, b를 가지는 임의의 store에서 모두 사용할 수 있습니다!
// 🙌 type generic

const a = view(aLens, fooStore); // 'foo'
const b = view(bLens, fooStore); // 'bar'

const bazStore = set(aLens, "baz", fooStore);
console.log(view(aLens, bazStore)); // 'baz'
```

## Use Cases

[compose](../../concepts/curry-function#compose)나 [pipe](../../concepts/curry-function#pipe)로 합성하여 중첩된 객체 내부에 있는 속성을 읽는 Lens를 만들 수 있습니다.

Ramda로 구현하면 다음과 같습니다.

```jsx
import { compose, lensProp, view } from "ramda";

const lensProps = ["foo", "bar", 1];

const lenses = lensProps.map(lensProp);
const truth = compose(...lenses);

const obj = {
  foo: {
    bar: [false, true],
  },
};

console.log(view(truth, obj)); // true
```

Lens의 part를 현재 값의 mapping된 결과로 수정하여 새로운 store를 반환하는 연산을 "over"라고 부르는데, 아래와 같이 구현할 수 있습니다.

```js
const over = (lens, f) => store => set(lens, f(view(lens, store)), store);
const upperCase = x => x.toUpperCase();

conosle.log(
	over(aLens, upperCase)(fooStore) // { a: "FOO", b: "bar" }
);
```

Lens의 over 연산은 [Functor의 composition law](../../data-structure/functor-category#Composition)를 만족합니다.
