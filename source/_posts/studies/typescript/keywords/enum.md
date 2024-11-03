---
title: Enum
date: 2024-11-03 20:15:03
categories:
  - Studies
  - TypeScript
  - Keywords
#tags
---
연관성이 있는 상수들을 모아서 하나의 객체처럼 접근할 수 있는 일종의 macro입니다.

enum을 지정하면 type과 value로 동시에 사용할 수 있고 상수에 이름을 붙일 수 있어서 코드의 가독성을 높일 수 있습니다.

### Numeric Enums

enum member들이 모두 `number` 타입을 갖습니다.

enum member의 초기화는 수동으로 지정하거나, `(직전 member의 값 + 1)`로 자동으로 지정할 수 있습니다.

```ts
enum Direction {
  Up = 1,
  Down, // 2
  Left, // 3
  Right, // 4
}
```

아래와 같이 enum의 이름을 type으로도 사용할 수 있습니다.

```ts
enum UserResponse {
  No = 0,
  Yes = 1,
}

function respond(recipient: string, message: UserResponse): void {
  // ...
}

respond("Princess Caroline", UserResponse.Yes);
```

### String Enums

enum member의 타입을 수동으로 `string`으로 지정합니다.

numeric enum에 비해서 enum member의 값에 의미를 부여할 수 있어서 런타임 중 값을 확인하거나 디버깅할 때 유용합니다.

```ts
enum Direction {
  Up = "UP",
  Down = "DOWN",
  Left = "LEFT",
  Right = "RIGHT",
}
```

### Enums at runtime

런타임 중 enum은 JS object로 존재하므로 아래와 같은 코드도 가능합니다.

```ts
enum E {
  X,
  Y,
  Z,
}

function f(obj: { X: number }) {
  return obj.X;
}

// Works, since 'E' has a property named 'X' which is a number.
f(E);
```

### Enums at compile(transpile) time

enum은 런타임 중에 JS object로 존재하지만 enum member의 key들을 조회하려면 `typeof`만이 아니라 아래와 같이 `keyof`와 같이 사용해야 합니다.

```ts
enum LogLevel {
  ERROR,
  WARN,
  INFO,
  DEBUG,
}

/**
 * This is equivalent to:
 * type LogLevelStrings = 'ERROR' | 'WARN' | 'INFO' | 'DEBUG';
 */
type LogLevelStrings = keyof typeof LogLevel;

function printImportant(key: LogLevelStrings, message: string) {
  const num = LogLevel[key];
  if (num <= LogLevel.WARN) {
    console.log("Log level key is:", key);
    console.log("Log level value is:", num);
    console.log("Log level message is:", message);
  }
}
printImportant("ERROR", "This is a message");
```

### Reverse Mapping

numeric enum의 경우, value -> key 방향으로 역매핑이 가능합니다.

```ts
enum Enum {
  A,
}

let a = Enum.A;
let nameOfA = Enum[a]; // "A"
```

이게 가능한 이유는 numeric enum은 아래와 같이 transpile되기 때문입니다.

```ts
"use strict";
var Enum;
(function (Enum) {
  Enum[(Enum["A"] = 0)] = "A";
})(Enum || (Enum = {}));
let a = Enum.A;
let nameOfA = Enum[a]; // "A"
```

하지만 string enum은 단일 매핑으로 transpile되기 때문에 역매핑이 안됩니다.

```ts
enum Direction {
  Up = "UP",
  Down = "DOWN",
  Left = "LEFT",
  Right = "RIGHT",
}
```

```js
"use strict";
var Direction;
(function (Direction) {
  Direction["Up"] = "UP";
  Direction["Down"] = "DOWN";
  Direction["Left"] = "LEFT";
  Direction["Right"] = "RIGHT";
})(Direction || (Direction = {}));
Try;
```

## 참고자료

[Enums](https://www.typescriptlang.org/docs/handbook/enums.html)

[How to use TypeScript Enums and why not to, maybe](https://www.youtube.com/watch?v=pWPClHdcvVg)
