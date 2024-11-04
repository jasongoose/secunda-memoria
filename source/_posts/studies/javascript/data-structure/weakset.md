---
title: WeakSet
date: 2024-11-03 16:21:14
categories:
  - Studies
  - JavaScript
  - Data Structure
#tags:
---
기존의 `Set` 과 같이 고유한 데이터를 저장하는 객체인데, 아래와 같은 두드러진 차이점이 있습니다.

- 요소가 무조건 객체여야 한다.
- 객체 요소가 외부에서 참조되지 않으면 garbage collect 대상이 된다.

`WeakMap`과 비슷한 이유로 요소가 언제, 어떻게 수거될지 모르기 때문에 반복문을 통한 enumeration이 불가합니다.

`WeakSet`은 중첩된 객체를 재귀적으로 접근하면서 순환참조를 방지할 때 유용합니다.

```js
// Execute a callback on everything stored inside an object
function execRecursively(fn, subject, _refs = new WeakSet()) {
  // Avoid infinite recursion
  if (_refs.has(subject)) {
    return;
  }

  fn(subject);

  if (typeof subject === "object") {
    _refs.add(subject);
    for (const key in subject) {
      execRecursively(fn, subject[key], _refs);
    }
  }
}

const foo = {
  foo: "Foo",
  bar: {
    bar: "Bar",
  },
};

foo.bar.baz = foo; // Circular reference!
execRecursively((obj) => console.log(obj), foo);
```
