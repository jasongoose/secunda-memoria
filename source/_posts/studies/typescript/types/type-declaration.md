---
title: Type Declaration
date: 2024-11-03 20:00:55
categories:
  - Studies
  - TypeScript
  - Types
#tags:
---
## .d.ts 파일

소스코드가 아닌 변수, 상수, class, function 등의 type declaration만 담고 있는 파일입니다.

`.js` 파일들로만 이루어진 패키지나 라이브러리들을 TS 프로젝트에서 사용할 때 필요한 type, interface, enum, function, class 등을 선언할 때 사용합니다.

보통 TS transpile 과정을 거친 뒤 결과물로 나오고 `.ts` 파일의 subset이기 때문에 TS 프로젝트 상에서는 일반 `.ts` 파일로 취급받습니다.

tsconfig.json에서 compiler-option인 `typeRoots` 또는 `types` 필드로 컴파일에 참여할 `.d.ts` 파일들의 경로를 설정할 수 있습니다.

## Local Type Declaration

module들 사이에서는 `import`, `export` 구문들을 사용하여 type을 서로 사용할 수 있습니다.

`import` 구문을 사용하면 tsc는 `.ts`, `.js`(`allowJs` 옵션이 `true`라면) 뿐만 아니라 `.d.ts` 확장자를 가진 파일도 검색합니다.

```ts
// local-declaration.d.ts
export interface Person {
  firstName: string;
  lastName: string;
  age: number;
}

// local-declaration-test.ts
import { Person } from "./local-declaration";

const ross: Person = {
  firstName: "Ross",
  lastName: "Geller",
  age: 29,
};
```

`.d.ts`에서 선언된 type은 `import` 구문 또는 triple slash directive를 사용하여 이용할 수 있습니다.

물론 tsc에 의해 컴파일된 `.js` 파일에는 type import 구문을 제거합니다.

```js
// local-declaration-test.js
"use strict";
exports.__esModule = true;
var ross = {
  firstName: "Ross",
  lastName: "Geller",
  age: 29,
};
```

## Global Type Declaration

### Project Example

global type declaration은 프로젝트 내의 컴파일 과정에 참여하는 모든 TS 파일들이 별도의 `import` 구문없이 사용할 수 있는 type들을 제공합니다.

기본적으로 compiler-option의 `lib` 필드를 설정하면(`lib`가 없다면 `target`으로) standard library의 implict type(global type)을 사용할 수 있습니다.

수동으로 global type declaration을 load하려면 `typeRoots`, `types` 필드를 설정하면 됩니다.

```text
/project/sample/
├── program.ts
├── tsconfig.json
└── types/
   └── common/
      ├── main.d.ts
      └── package.json
```

```json
// tsconfig.json
{
  "files": ["./program.ts"],
  "compilerOptions": {
    "typeRoots": ["./node_modules/@types", "./types"]
  }
}
```

```json
// types/common/package.json
{
  "name": "common",
  "version": "1.0.0",
  "typings": "main.d.ts"
}
```

```ts
// types/common/main.d.ts
interface Person {
  firstName: string;
  lastName: string;
  age: number;
}
```

위와 같은 프로젝트 구조에서 사용할 type-root는 `node_modules/@types`(defalut)와 `types/` 이고 `.d.ts` 파일들을 가지고 있을 declaration package(module)은 `types/common/` 이다.

보통 `types/` 폴더(type-root)에는 `index.d.ts` 파일(entry declaration file)이 위치하여 다른 모듈들로부터 type을 import하는 역할을 합니다.

만일 `index.d.ts` 파일이 없다면 declaration package 내의 package.json을 생성하여 `types` 또는 `typings` 필드를 모듈 내 entry 역할을 할 `.d.ts` 파일로 설정해야 합니다.

### Modularizing Declarations

규모가 커질수록 프로젝트에서 사용하는 type의 개수는 늘어나므로 type declaration 파일을 여러 개로 나눌 필요가 있습니다.

여러 개의 `.d.ts` 파일들을 만들고 entry 파일에서 import하는 방식으로 모듈화된 프로젝트 구조를 바꿔보자.

```json
/project/sample/
├── program.ts
├── tsconfig.json
└── types/
   └── common/
      ├── main.d.ts
      ├── interfaces.d.ts
      ├── functions.d.ts
      └── package.json
```

```ts
// types/common/interfaces.d.ts
interface Person {
  firstName: string;
  lastName: string;
  age: number;
}
```

```ts
// types/common/functions.d.ts
type GetFullName = (p: Person) => string;
```

그리고 `main.d.ts`(entry)에서 triple-slash directive를 사용하여 다른 declaration 파일들을 명시합니다.

```ts
// types/common/main.d.ts

/// <reference path="./interfaces.d.ts" />
/// <reference path="./functions.d.ts" />
```

### Third-party Declarations

node 환경에서 프로젝트를 만들면 node 패키지들을 많이 사용하게 되는데 패키지마다 필요한 type declaration 파일들은 [DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped)에서 제공하는 declaration package를 사용하면 됩니다.

설치 명령어는 다음과 같습니다.

```zsh
$ npm install -D @types/[package_name]
```

위 명령어를 사용하면 `node_modules/@types/[package_name]/` 아래로 패키지와 관련된 declaration file(`.d.ts`) 파일들이 위치하게 됩니다.

```text
/project/sample/
├── program.ts
├── package.json
├── tsconfig.json
├── node_modules
|  └── @types/
|     └── [package_name]
└── types
   └── common/
```

여기서 `node_modules/@types`도 tsconfig.json의 `typeRoots` 필드로 미리 지정해두어야 합니다.

```json
// tsconfig.json
{
  "files": ["./program.ts"],
  "compilerOptions": {
    "lib": ["ES2015"],
    "typeRoots": ["./node_modules/@types", "./types"]
  }
}
```

### Ambient Declaration

ambient value는 실행 중에만 존재하는 값으로, 대표적인 예로는 Node의 `global`, 브라우저의` window` 같은 전역객체가 있습니다.

TS에서 만일 ambient value를 바로 사용한다면 `undefined` 에러가 발생하는데, 이를 해결하기 위해서 `declare` 키워드를 사용하면 됩니다.

```ts
declare var window: any;
```

`declare` 키워드를 사용하여 ambient value를 선언하는 방식을 "ambient declaration"이라고 합니다.

`declare` 키워드는 tsc로 하여금 해당 변수는 언젠가 어디서든 정의되어 있으니 컴파일 에러를 발생시키지 말라는 의미를 가집니다.

ambient declaration은 전역적으로 사용되어야 하므로 global declaration entry에 작성되어야 합니다.

```ts
// types/common/main.d.ts

/// <reference path="./interfaces.d.ts" />
/// <reference path="./functions.d.ts" />
declare var version: string;
```

### Namespaced Declarations

declaration package에서 전역적으로 제공하는 type declaration들이 많아지면 패키지들 사이에서 이름이 중복될 수 있는데, 여기서 namespace를 사용하면 됩니다.

```ts
// types/common/main.d.ts

/// <reference path="./interfaces.d.ts" />
/// <reference path="./functions.d.ts" />
```

```ts
// types/common/interfaces.d.ts
declare namespace common {
  interface Person {
    firstName: string;
    lastName: string;
    age: number;
  }
}
```

```ts
// types/common/functions.d.ts
declare namespace common {
  type GetFullName = (p: Person) => string;
}
```

`.d.ts` 파일에 namespace를 `declare` 키워드와 함께 사용하는 이유는 type declaration 파일에는 value가 아닌 `type`, `interface`, `class`, `function`만 선언하기 때문입니다.

`namespace`는 value이므로 ambient declaration을 사용하여 전역적으로 사용할 수 있도록 만드는 겁니다.

namespace 내부의 `type`, `interface`, `class`, `function`은 자동으로 export됩니다.

naemespace는 type annotation에만 사용되기 때문에 컴파일 과정에는 포함되지 않습니다.

소스코드에서 namespace 내의 type을 사용할 때는 `<namespace_name>.<type_name>`으로 지정하면 됩니다.

### Extending Declarations

```ts
// types/common/main.d.ts

/// <reference path="./interfaces.d.ts" />
/// <reference path="./functions.d.ts" />

// types/common/interfaces.d.ts
interface Person {
  firstName: string;
  lastName: string;
  age: number;
}

// types/common/functions.d.ts
declare namespace common {
  type GetFullName = (p: Person) => string;
}
```

TS에서 동일한 이름을 가진 `interface`가 동일한 **module** 내에서 선언되면 `interface`의 속성들이 결합되면서 결과적으로 하나의 `interface`가 선언됩니다.

또한 이미 존재하고 있는 `namespace`를 **script**에서 다시 선언하면 기존의 `namespace`에 새로운 export member들이 추가되면서 확장됩니다.

```ts
// program.ts

// append global `Person` interface
interface Person {
  email: string;
}

// append global `Number` interface
interface Number {
  isEven(): boolean;
}

// create an object of type `Person`
const ross: Person = {
  firstName: "Ross",
  lastName: "Geller",
  age: 29,
  email: "ross@geller.com",
};

// (method) Number.isEven(): boolean
// won't work because isEven() is not implemented
const isAgeEven = ross.age.isEven();

// const isAgeEven: boolean
console.log("isAgeEven", isAgeEven);
```

그러나 위 `program.ts` 파일이 script가 아닌 아래와 같은 module이 되면 얘기가 달라집니다.

```ts
// program.ts

interface Person {
  email: string;
}

interface Number {
  isEven(): boolean;
}

// create an object of type `Person`
const ross: Person = {
  firstName: "Ross", // ERROR! : not assignable to  type {email: string;}
  lastName: "Geller",
  age: 29,
  email: "ross@geller.com",
};

// (method) Number.isEven(): boolean
const isAgeEven = ross.age.isEven(); // ERROR! : 'age' doesn't exit in type Person

// const isAgeEven: boolean
console.log("isAgeEven", isAgeEven);

// (make a module)
export {};
```

Person `interface`와 Number `interface`는 기존 global declaration을 확장하는 것이 아닌 module scope에서 새로 선언된 `interface`가 되므로 없는 속성을 사용하는 부분에서 에러가 발생합니다.

컴파일 과정에 포함되는 script 파일들만이 서로 value와 type들을 공유할 수 있지만 모듈은 import, export하지 않는 이상 서로 주고받을 방법이 없습니다.

모듈에서도 `interface`를 확장하려면 `declare global` 키워드를 사용하면 됩니다.

해당 키워드를 사용하면 내부에 포함된 type이나 variable이 global scope에 포함되기 때문에, 모듈 안에서도 script처럼 기존의 `interface`를 확장할 수 있습니다.

```ts
// program.ts

declare global {
  // append global `Person` interface
  interface Person {
    email: string;
  }
}

declare global {
  // append global `Number` interface
  interface Number {
    isEven(): boolean;
  }

  // declare global `version`
  var version: string;
}

// create an object of type `Person`
const ross: Person = {
  firstName: "Ross",
  lastName: "Geller",
  age: 29,
  email: "ross@geller.com",
};

// (method) Number.isEven(): boolean
const isAgeEven = ross.age.isEven();

// const isAgeEven: boolean
console.log("isAgeEven", isAgeEven);

// (make a module)
export {};
```

## 참고자료

[About "\*.d.ts" in TypeScript](https://stackoverflow.com/questions/21247278/about-d-ts-in-typescript)

[What is the difference between _.d.ts vs _.ts in typescript?](https://stackoverflow.com/questions/29196657/what-is-the-difference-between-d-ts-vs-ts-in-typescript)

[Modules .d.ts](https://www.typescriptlang.org/docs/handbook/declaration-files/templates/module-d-ts.html)
