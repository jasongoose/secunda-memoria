---
title: Namespace
date: 2024-11-03 19:31:14
categories:
  - Studies
  - TypeScript
  - Types
#tags
---
## Namespace란?

`.ts` 파일의 특정 영역을 전역공간(global scope)에 노출되지 않도록 생성한 공간입니다.

전역공간에 노출되면 안되는 이유는 (1) 제3의 라이브러리가 전역공간에 만든 함수, 변수들과의 이름 충돌을 피하고 (2) 외부에서 바로 접근하면 안되는 중요한 데이터를 숨기기 위해서입니다.

과거에는 이러한 성격을 반영하듯이 namespace를 “internal module”이라고 불렀습니니다.

namespace는 `namespace` 키워드로 선언할 수 있습니다.

```ts
// a.ts
namespace MyLibA {
  const _version = "1.0.0";
  function getVersion() {
    return _version;
  }
}

console.log(_version); // ❌ ERROR
console.log(getVersion()); // ❌ ERROR
```

namespace 안에 위치한 변수를 사용하려면 namespace 내부에서 `export` 키워드를 사용합니다.

```ts
// a.ts
namespace MyLibA {
  const _version = "1.0.0";
  export function getVersion() {
    return _version;
  }
}

console.log(MyLibA._version); // ❌ ERROR
console.log(MyLibA.getVersion()); // 1.0.0
```

namespace 내부 `export` 키워드는 선언, 할당과 동시에 부여할 수 있습니다.

```ts
namespace P {
  const private = 1;
  export const public = 2;
  export function publicFunc() { ... };
}
```

namespace는 tsc에 의해 컴파일된 `.js` 파일에서 IIFE(Immediate Invoked Function Expression)와 empty variable declaration으로 구현됩니다.

```js
// a.js
var MyLibA; // empty variable declaration

(function (MyLibA) {
  var _version = "1.0.0";
  function getVersion() {
    return _version;
  }
  MyLibA.getVersion = getVersion;
})(MyLibA || (MyLibA = {})); // IIFE

console.log(MyLibA.getVersion()); // 1.0.0
```

## Exporting Types & Namespaces

namepace에서 `type`, `interface`를 export할 수 있지만 타입 정보는 컴파일 뒤에 사라져서 `.js` 파일에서 확인할 수 없습니다.

```ts
// a.ts
namespace MyLibA {
  export interface Person {
    name: string;
    age: number;
  }
  export function getPerson(name: string, age: number): Person {
    return { name, age };
  }
}

const ross: MyLibA.Person = MyLibA.getPerson("Ross", 30);
s;
```

```js
// 컴파일 뒤, a.js
var MyLibA;

(function (MyLibA) {
  function getPerson(name, age) {
    return { name: name, age: age };
  }
  MyLibA.getPerson = getPerson;
})(MyLibA || (MyLibA = {}));

var ross = MyLibA.getPerson("Ross", 30);
```

namespace는 결과적으로 JS 객체이므로 `export`가 가능한데 아래와 같이 중첩된 namespace도 구현할 수도 있습니다.

```ts
// a.ts
namespace MyLibA {
  export namespace Types {
    export interface Person {
      name: string;
      age: number;
    }
  }
  export namespace Functions {
    export function getPerson(name: string, age: number): Types.Person {
      return { name, age };
    }
  }
}

const ross: MyLibA.Types.Person = MyLibA.Functions.getPerson("Ross Geller", 30);
```

## Aliasing

중첩된 namespace의 export memeber를 사용할 때는 chaining을 여러 번 해야하기 때문에 코드 가독성이 떨어질 수도 있습니다.

이를 해결하기 위해 `import` 구문을 사용하여 특정 namespace의 export member에 alias를 부여할 수 있습니다.

```ts
// a.ts
namespace MyLibA {
  export namespace Types {
    export interface Person {
      name: string;
      age: number;
    }
  }
  export namespace Functions {
    export function getPerson(name: string, age: number): Types.Person {
      return { name, age };
    }
  }
}

import Person = MyLibA.Types.Person;
import API = MyLibA.Functions;

const ross: Person = API.getPerson("Ross Geller", 30);
```

## Importing Namespaces

`import`, `export` 구문으로 다른 곳에 위치한 namespace를 가져와서 사용할 수 있습니다.

```ts
// b.ts
export namespace MyLibA {
  export interface Person {
    name: string;
    age: number;
  }
  export function getPerson(name: string, age: number): Person {
    return { name, age };
  }
}

// a.ts
import { MyLibA } from "./b";
const ross: MyLibA.Person = MyLibA.getPerson("Ross Geller", 30);
```

## Modularization

다른 모듈을 가져다 사용할 때, tsconfig.json의 `module` 필드에 명시된 module system에 따라서 다른 구문을 사용합니다.

TS에서는 triple-slash directive(reference directive)를 사용하여 컴파일 과정에 모듈을 자동으로 포함시킬 수 있습니다.

c/c++의 preprocessor directive(예: `#define "stdio.h"`)와 비슷합니다.

c/c++에서 컴파일하면 directive에 명시한 헤더파일의 내용을 모두 복사/붙여넣기하여 컴파일 과정에 포함시킵니다.

reference directive도 컴파일할 동안에만 존재하고 최종 `.js` 파일에는 없어집니다.

기존의 import 구문과 차이점은 (1) import할 member들을 일일이 명시하지 않아도 되고 (2) 컴파일 뒤 `.js` 파일에는 `require`/`exports`, `define`/`return` 등 모듈을 가져다 쓰는 부분이 없다는 겁니다.

```ts
// a.ts
/// <reference path="./b.ts"/>
const ross: MyLibA.Person = MyLibA.getPerson("Ross Geller", 30);

// b.ts
namespace MyLibA {
  export interface Person {
    name: string;
    age: number;
  }
  export function getPerson(name: string, age: number): Person {
    return { name, age };
  }
}
```

`tsc a.ts` 명령어를 실행하면 아래 코드로 컴파일되는데, 이 상태에서 `a.js`, `b.js`는 모듈이 아니기 때문에 node에서 실행할 수 없습니다.

```js
// a.js
var ross = MyLibA.getPerson("Ross Geller", 30);

// b.js
var MyLibA;

(function (MyLibA) {
  function getPerson(name, age) {
    return { name: name, age: age };
  }
  MyLibA.getPerson = getPerson;
})(MyLibA || (MyLibA = {}));
```

여기서 tsconfig.json의 `outFile` 필드를 `bundle.js`로 지정하여 컴파일된 `.js` 파일들을 하나로 합치는 bundling 과정을 거쳐야 합니다.

bundling을 거치면 tsc는 `.js` 코드 간의 우선순위(의존관계)를 분석하여 `bundle.js`에 순서대로 코드를 합쳐줍니다.

```js
// bundle.js

// (from b.ts)
var MyLibA;
(function (MyLibA) {
  function getPerson(name, age) {
    return { name: name, age: age };
  }
  MyLibA.getPerson = getPerson;
})(MyLibA || (MyLibA = {}));

// (from a.ts)
var ross = MyLibA.getPerson("Ross Geller", 30);
```

## Extending Namespaces

동일한 모듈에서 동일한 이름을 가진 namespace들은 하나로 합쳐집니다.

이를 활용하면 reference directive로 기존 namespace를 더 확장시킬 수 있습니다.

```ts
// a.ts
/// <reference path="./b.ts"/>
const john: MyLibA.Person = MyLibA.defaultPerson;
const ross: MyLibA.Person = MyLibA.getPerson("Ross Geller", 30);

console.log(john); // {name: 'John Doe', age: 21}
console.log(ross); // {name: 'Ross Geller', age: 30}

// b.ts
/// <reference path="./c.ts" />
namespace MyLibA {
  export const defaultPerson: Person = getPerson("John Doe", 21);
}

// c.ts
namespace MyLibA {
  export interface Person {
    name: string;
    age: number;
  }
  export function getPerson(name: string, age: number): Person {
    return { name, age };
  }
}
```

여기서 `tsc —-outFile bundle.js a.ts` 명령어로 번들파일(`bundle.js`)을 생성하면 다음과 같이 만들어집니다.

```js
// bundle.js

// (from c.ts)
var MyLibA;
(function (MyLibA) {
  function getPerson(name, age) {
    return { name: name, age: age };
  }
  MyLibA.getPerson = getPerson;
})(MyLibA || (MyLibA = {}));

// (from b.ts)
var MyLibA;
(function (MyLibA) {
  MyLibA.defaultPerson = MyLibA.getPerson("John Doe", 21);
})(MyLibA || (MyLibA = {}));

// (from a.ts)
var john = MyLibA.defaultPerson;
var ross = MyLibA.getPerson("Ross Geller", 30);
```

namespace + reference directive + bundling을 이용하면 ESM을 지원하지 않는 오래된 브라우저에서도 실행되는 앱을 만들 수 있습니다.

TS에서는 ESM, CommonJS module들의 bundling이 불가능하지만 browserify, webpack, vite와 같은 번들러 도구를 사용하면 bundling을 가능하게 하여 cross-platform + backward compatible한 앱을 충분히 만들 수 있습니다.
