---
title: Module Standard
date: 2024-11-03 19:29:56
categories:
  - Studies
  - TypeScript
  - Module
#tags:
---
TS에서 사용하는 module system은 ESM 방식을 거의 따릅니다.

## Named Exports

다른 모듈을 import하는 경우 다음과 같이 사용할 변수들을 대괄호 안에 명시합니다.

```ts
// program.ts
import { A } from "path/to/values";
console.log(A); // "Apple"
```

여기서 export하는 모듈은 최하단 줄에서 한꺼번에 export 하거나,

```ts
// values.ts
var A = "Apple"; // can also use `let` or `const`
class B {}
function C() {}
enum D {}

export { A, B, C, D };
```

아래와 같이 변수의 선언, 할당과 동시에 바로 export할 수 있습니다.

```ts
// values.ts
export const A = { name: "Apple" };
export class B {}
export function C() {}
export enum D {}
```

## Default Export

`default` 키워드를 사용하여 이름을 명시하지 않고도 import할 수 있습니다.

```ts
// values.ts
var A = "Apple";
export default A;

// program.ts
import AA from "path/to/values";
console.log(AA); // "Apple"
```

오직 하나의 값만 default export할 수 있고 변수의 선언, 할당과 동시에 default export하는 경우는 `function`, `class`만 가능합니다.

```ts
// values.ts
export default var A = "Apple"; // ❌ invalid syntax
export default enum D{} // ❌ illegal: not a function or class
export default class B {} // ✅ legal
export default function C(){} // ✅ legal
```

하지만 변수를 사용하지 않고 값(expression)을 바로 export default하는 것은 어떤 타입의 데이터든 가능합니다.

```ts
// values.ts
export default "Hello"; // string
export default {}; // object
export default () => undefined; // function
export default function () {} // function
export default class {} // class
```

named export와 default export는 동시에 한 모듈에서 사용할 수 있습니다. 하지만 default export가 named export 앞에 위치해야 합니다.

```ts
// values.ts
const B = 1;
const C = {};

export {"Hello" as default, B, C};

// program.ts
import Apple, { B, C } from 'path/to/values'; // 이 순서로 import하면 됩니다.
```

## Import & Export Alias

`as` 키워드를 사용하여 [named export](#Named-Exports) member들을 다른 이름(alias)으로 import할 수 있습니다.

```ts
// program.ts
import A, { B as Ball } from "path/to/values";
console.log(B); // ❌ Error: Cannot find name 'B'.
console.log(Ball); // ✅ legal
```

또한 [defaut export](#Default-Export) member도 alias를 부여하여 import할 수 있습니다.

```ts
import { default as A, B as Ball } from "path/to/values";
```

`export` 모듈에서도 alias를 부여할 수 있습니다.

```ts
// values.ts
var A = "Apple"; // can also use `let` or `const`
class B {}
function C() {}
enum D {}

export { A as default, B, C as Cat, D };
```

## Import All Named Exports

모듈에서 [named export](#Named-Exports) member들 모두 하나의 객체로 import할 때, `* as` 키워드를 사용하면 됩니다.

```ts
// program.ts
import Apple, * as values from "path/to/values";

console.log(values.B);
console.log(values.C);
console.log(values.D);
```

## Re-exports

`export-from` 구문을 사용하여 모듈의 [named export](#Named-Exports) member들을 처음부터 import하지 않고 바로 export할 수 있습니다.

`as` 키워드로 alias도 사용할 수 있는데, 여기서 `export-from` 구문으로 re-export된 member들은 코드 내에서 사용할 수 없습니다.

```ts
// lib.ts

// re-exports
export { P as Paper, Q } from "path/to/values-1";
export { N, O as Orange } from "path/to/other/values-2";

console.log(P); // ❌ Error: Cannot find name 'P'.
console.log(Orange); // ❌ Error: Cannot find name 'Orange'.

// default export
export default "hello";
class B {}
function C() {}

// named exports
export { B, C as Cat };
```

`export * from` 구문으로 [named export](#Named-Exports) member들을 모두 re-export할 수 있는데, 여기서도 `as` 키워드로 alias를 사용할 수 있습니다.

```ts
// lib.ts
export * from "path/to/values-1";
export * as values2 from "path/to/other/values-2";

// program.ts
import { P, Q, values2 } from "path/to/lib";
```

`export-from` 구문에 `default` 키워드를 사용하면 모듈의 [default export](#Default-Export) member를 re-export할 수 있고, 물론 alias도 가능합니다.

```ts
// lib.ts
export { default } from "path/to/values-1";
export { default as Apple } from "path/to/values-2";
```
