---
title: Type Assertion
date: 2024-11-03 19:31:43
categories:
  - Studies
  - TypeScript
  - Types
#tags:
---
## Non-null operator !

TS의 모든 변수 뒤에 `!`를 붙이면 해당 타입은 `null`이나 `undefined`가 아님을 tsc에 알려주는 구문입니다.

정말로 `null`, `undefined`가 아니라는 확신이 있을 경우에만 사용해야 합니다.

```ts
function liveDangerously(x?: number | null) {
  // No error
  console.log(x!.toFixed());
}
```

## as 키워드

특정 데이터에 대해서 TS가 추론한 타입과 내가 예상하는 타입이 다를 수 있는데, 이 경우 뒤에 `as` 키워드를 붙여서 해당 데이터의 더 구체적이거나 포괄적인 타입을 강제하면 됩니다.

`as` 키워드는 tsc에 의해서 컴파일 과정에 포함되지 않습니다.

```ts
const myCanvas = document.getElementById("main_canvas") as HTMLCanvasElement;
```

## as const

속성, 요소, 변수의 타입을 literal type으로 지정할 때 사용합니다.

```ts
// Type '"hello"'
let x = "hello" as const;

// Type 'readonly [10, 20]'
let y = [10, 20] as const;

// Type '{ readonly text: "hello" }'
let z = { text: "hello" } as const;
```
