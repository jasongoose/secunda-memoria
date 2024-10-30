---
title: Object Composition
date: 2024-10-28 08:44:28
categories:
  - Books
  - Composing Software
  - 2. Concepts
#tags:
---
SW에서 추상화는 공통 interface에 기능을 추가/오버라이드/확장하여 구체화할 수 있도록 일반화하는 작업을 의미합니다.

추상화 구현은 보통 OOP의 상속이 흔하게 사용되지만, 필자는 독립적이고 작은 단위의 객체들을 합성하여 복잡하고 확장된 객체를 만드는 Object Composition 방법을 권장합니다.

객체를 구성하는 방법들은 3가지가 있는데 아래 예시 코드를 공통으로 사용하겠습니다.

```js
const objs = [{ a: "a", b: "ab" }, { b: "b" }, { c: "c", b: "cb" }];
```

## Aggregation

다수의 하위 객체들을 모은 collection으로 새로운 객체를 생성하는 방법입니다.

aggreation으로 생성되는 객체들을 나열하면 다음과 같습니다.

- Array
- Map
- Set
- DOM Nodes
- UI Components

### Array Aggreation

```js
const collection = (a, e) => a.concat([e]);
const a = objs.reduce(collection, []);

console.log(
  "collection aggregation",
  a,
  a[1].b,
  a[2].c,
  `enumerable keys: ${Object.keys(a)}`
);
/*
collection aggregation
[{"a":"a","b":"ab"},{"b":"b"},{"c":"c","b":"cb"}]
b c
enumerable keys: 0,1,2
*/
```

### Linked List Aggregation

```js
const pair = (a, b) => [b, a];
const l = objs.reduceRight(pair, []);

console.log("linked list aggregation", l, `enumerable keys: ${Object.keys(l)}`);
/*
linked list aggregation
[
  {"a":"a","b":"ab"}, [
    {"b":"b"}, [
      {"c":"c","b":"cb"},
      []
    ]
  ]
]
enumerable keys: 0,1
*/
```

## Concatenation

기존의 객체에 새로운 속성, 메서드들을 점진적으로 추가하여 객체를 생성하는 방법입니다.

단, 아래 사항들을 유의해야 합니다.

- side-effect가 일어나지 않도록 원본 객체를 mutate하면 안됩니다.
- 클래스 상속이나 객체 간의 is-a 관계를 구현하지 않습니다.
- 속성 key값들 사이에서 충돌이 발생하지 않도록 합니다.

```js
const concatenate = (a, o) => ({ ...a, ...o });
const c = objs.reduce(concatenate, {});

console.log("concatenation", c, `enumerable keys: ${Object.keys(c)}`);
// concatenation { a: 'a', b: 'cb', c: 'c' } enumerable keys: a,b,c
```

## Delegation

객체의 속성과 메서드 검색을 다른 객체로 넘겨주는 기법으로, JS에서는 prototype chain으로 구현됩니다.

Delegation은 다음과 같은 이점이 있습니다.

- 메모리 절약
    - 여러 객체 instance간의 공통 속성이나 메서드를 한곳에서만 저장하여 중복을 방지합니다.
- State 공유
    - 런타임 중 동적으로 다수 intance들의 state 동기화가 필요하다면 delegate된 객체만 mutate하면 됩니다.
    - 하지만 흔한 버그의 원인들 중 하나입니다😅

```js
const delegate = (a, b) => Object.assign(Object.create(a), b);
const d = objs.reduceRight(delegate, {});

console.log("delegation", d, `enumerable keys: ${Object.keys(d)}`);
// delegation { a: 'a', b: 'ab' } enumerable keys: a,b

console.log(d.b, d.c);
// ab c
```