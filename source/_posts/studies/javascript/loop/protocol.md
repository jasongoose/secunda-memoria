---
title: Protocols
date: 2024-11-03 17:13:28
categories:
  - Studies
  - JavaScript
  - Loop
#tags:
---
## Iterator

일련의 값들을 생성하기 위한 방법을 정의하는 객체로, `next` 메서드를 구현해야 합니다.

`next` 메서드는 호출될 때마다 `done`과 `value`라는 속성을 가지는 객체를 계속 반환하는데, 생성할 값들이 고갈되면 반환을 멈춥니다.

![Iterator](/images/iterator.png)

## Iterable

JS 객체의 속성들을 순회하는 방식을 커스터마이징하는데 사용됩니다.

반드시 객체 자체 또는 prototype chain 상에서 `@@iterator`라는 zero-argument 메서드를 가져야 합니다.

object에서 정의할 때는 `[Symbol.iterator]`라는 키를 사용합니다.

![Iterable](/images/iterable.png)

object가 순회된다면 `@@iterator` 메서드가 반환한 [Iterator](#Iterator) 객체로부터 iterated value를 얻을 수 있습니다.

반환되는 Iterator를 정의할 때 내부에서 `this` 키워드를 사용하면 object의 속성에 접근할 수 있습니다.

JS에 내장된 Iterable로는 다음이 있습니다.

- String
- Array
- TypedArray
- Map
- Set
- Array-like object
    - arguments
    - DOM Collection(NodeList)
    - FileList
    - …
- [Generator](#Generator)

Iterable를 순회하는 방법으로는 다음이 있습니다.

- `for-of` 문
- spread operator
- `yield*`
- destructuring assignment

## Generator

Generator Function에 의해서 반환되는 Iterable이자 Iterator인 객체입니다.

Iterable이므로 `for-of` 문 등으로 value들을 순회할 수 있고, iterator이므로 `next` 메서드로 생성된 value들을 하나씩 읽을 수 있습니다.

Iterable + Promise의 조합을 사용하면 Callback Hell, Inversion of Control 등의 문제를 해결하는 도구로 사용할 수 있습니다.

Generator의 `next` 메서드를 호출할 때마다 `yield` 문에서 실행을 멈추고 바로 뒤에 있는 값을 아래와 같은 객체로 반환합니다.

```js
yield x;

// 실제로 iterator에 의해서 반환되는 객체는...
{
	done: false,
	value: x,
}
```

`yield`가 아닌 `yield*`를 만나면 다른 Generator Function이 대신해서 yield합니다.

`next` 메서드에 인자를 전달하여 실행하면 Generator Function을 다시 실행하는데 중간에 멈춘 `yield` 키워드 대신 전달된 인자로 대체됩니다.

```js
function* gen() {
  while (true) {
    const value = yield;
    console.log(value);
  }
}

const g = gen();

// Note: The first call does not log anything,
// because the generator was not yielding anything initially.
g.next(1);
// No log at this step: the first value sent through `next` is lost
// "{ value: undefined, done: false }"

g.next(2);
// 2
// "{ value: undefined, done: false }"
```

```js
// 첫번째 next를 호출할 당시에는 generator 초기단계로, 아직 yield에 도착하지 않은 상태이다.
// 따라서 1을 출력하려면 g.next()를 최초로 호출하여 yield로 이동시켜야 한다.

g.next();
g.next(1);
g.next(2);
```

`next()` 메서드를 실행했을 때 Generator Function 내부에서 `yield` 대신 `return` statement를 만나면 순회를 종료합니다.

만일 특정 값을 return한다면 `done` property가 `true`인 객체를 반환하고 종료합니다.

Generator가 순회를 종료할 때는 `{done: true, value: undefined}`를 반환합니다.
