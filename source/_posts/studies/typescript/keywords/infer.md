---
title: infer
date: 2024-11-03 20:15:15
categories:
  - Studies
  - TypeScript
  - Keywords
#tags
---
특정 type을 사전에 정의하는 것이 아닌 참조하는 변수에 할당되었을 때 추론합니다.

단, 언제나 conditional type의 "extends clause" 내부에서만(`?` 왼쪽 조건영역) 사용할 수 있습니다.

extends clause 내부에서 `infer`는 types expression에서 type이 있는 곳에 대신 작성하면 됩니다.

```ts
type Unpacked<T> = T extends (infer U)[]
  ? U
  : T extends (...args: any[]) => infer U
  ? U
  : T extends Promise<infer U>
  ? U
  : T;

type T0 = Unpacked<string>; // string
type T1 = Unpacked<string[]>; // string
type T2 = Unpacked<() => string>; // string
type T3 = Unpacked<Promise<string>>; // string
type T4 = Unpacked<Promise<string>[]>; // Promise<string>
type T5 = Unpacked<Unpacked<Promise<string>[]>>; // string
```

```ts
type Parameters<F> = F extends (...args: infer X) => any ? X : never;
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;
```