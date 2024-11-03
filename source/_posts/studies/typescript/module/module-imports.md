---
title: Module Imports
date: 2024-11-03 19:29:17
categories:
  - Studies
  - TypeScript
  - Module
#tags:
---
## Module Caching

앱이 실행 중일 때 동일한 모듈이 다른 모듈들에 의해서 여러 번 import가 되는 경우, JS engine은 최초의 import 구문을 확인했을 때 import되는 모듈을 처음이자 마지막으로 초기화(메모리 한 공간에 모듈에 관한 정보를 저장)합니다.

그 뒤로 해당 모듈의 import가 발생할 때마다 초기화된 모듈을 참조합니다(caching).

이 방식은 모듈의 import로 일어나는 side-effect를 오직 한번만 만들 수 있다는 점에서 유용합니다.

## Import Position

`import` 구문은 모듈의 첫 번째 줄에 작성해야 합니다.

함수 블럭 내부나 조건문 블럭 내부에 작성하면 컴파일 에러가 발생합니다.

함수 안에 `import`를 사용하려면 import되는 모듈이 언제 초기화될지를 분석하는 control flow analysis가 tsc에 의해서 수행되어야 합니다.

## Module File Path

relative-import 방식에서 모듈의 path를 명시할 때는 compile-time constant string(tsc가 컴파일할 때 constant)여야 합니다.

즉, template literal, function call, string concatenation 방식으로 작성하면 안됩니다.

## Relative Import

프로젝트 root에 대한 상대경로를 사용하고 모듈 이름이 `/` , `./` , `../` 로 시작합니다.

해당 모듈의 [ambient module](../ambient-modules) 선언은 참조할 수 없습니다.

```ts
import Entry from "./components/Entry";
import { DefaultHeaders } from "../constants/http";
import "/mod";
```

## Non-relative Import

`tsconfig.json` 파일의 `baseUrl` 필드값 기준 상대경로에 위치한 모듈을 참조하고, 해당 모듈의 [ambient module](../ambient-modules) 선언을 참조할 수 있습니다.

```ts
import * as $ from "jquery";
import { Component } from "@angular/core";
```

## Dynamic Loading

TS 모듈의 제일 윗줄에서 `import` 구문을 사용하는 방식 말고도 원하는 때와 위치에서 import할 수 있는 mechanism이 있는데 이를 "dynamic loading"이라고 합니다.

dynamic loading은 `import()` expression을 사용합니다.

인자로 import할 모듈의 상대/절대경로를 전달하면 유효할 경우 해당 경로를 가진 모듈의 [named export member](../module-standard#named-exports), [default export member](../module_standard#default-export)를 resolve하는 `Promise` 객체를 반환합니다.

유효하지 않은 경로라면 rejected `Promise`를 반환합니다.

```ts
function run() {
  import("path/to/values")
    .then((mod) => {
      console.log(mod.default); // default export
      console.log(mod.A); // named export
      console.log(mod.B); // named export
    })
    .catch((error) => {});
}

// 또는

async function run() {
  const mod = await import("path/to/values");
  console.log(mod.default);
  console.log(mod.A);
  console.log(mod.B);
}
```

`import()` expression의 인자인 모듈의 경로는 실행 중에 구체화되기 때문에 (1)반환값의 타입이 `string`인 함수나 (2)JS expression을 인자로 지정해도 됩니다.

하지만 `import()` expression의 인자를 (1), (2) 방식처럼 동적으로 전달하면 실행 중에 어떤 모듈이 import될지 예상할 수 없기 때문에 `import()` expression의 반환값은 `Promise<any>` 타입을 가집니다.

(1), (2)와 같이 동적으로 전달된 모듈들은 컴파일 과정에서 제외되기 때문에 tsconfig.json 파일에서 `files` 또는 `include` 필드를 사용하여 수동으로 포함시켜야 합니다.

## Module Resolution Strategies

TS 모듈은 ESM syntax(`import`, `export`)를 사용합니다.

import할 파일은 `.ts`, `.d.ts`, `.js`가 가능하고 해당 파일은 (1)상대/절대경로 또는 (2)모듈이름으로 지정할 수 있습니다.

tsc가 모듈을 검색하는 과정을 module resolution strategy라고 부르고 크게 classic 방식과 node 방식으로 나뉩니다.

### Classic

backward compatability을 보장하고 `import` 구문에 주어진 이름과 `.ts`, `.d.ts` 확장자를 가진 파일을 검색합니다.

#### relative import

현재 폴더에 대한 상대경로로 해석하고 해당 위치에 존재하는 `.ts`, `.d.ts` 파일을 참조합니다.

```ts
// relative import - Classic strategy
Import statement: import "./moduleB";
Import location: /root/src/folder/A.ts

1. /root/src/folder/moduleB.ts
2. /root/src/folder/moduleB.d.ts
```

#### non-relative import

먼저 현재 폴더에서 찾아보고 없으면 상위 폴더를 차례대로 올라가면서 프로젝트 root까지 재귀적으로 검색하고, 없으면 에러를 던집니다.

```ts
// non-relative import - Classic strategy
Import statement: import 'package';
Import location: /root/dir/sub/example.ts

1. /root/src/folder/moduleB.ts
2. /root/src/folder/moduleB.d.ts
3. /root/src/moduleB.ts
4. /root/src/moduleB.d.ts
5. /root/moduleB.ts
6. /root/moduleB.d.ts
7. /moduleB.ts
8. /moduleB.d.ts

9. Error: Cannot find module
```

### Node

현재 대부분의 모듈들이 node package들이라서 대부분 이 방식을 선호합니다.

#### relative import

`import` 구문에 상대/절대경로가 명시되어 있으면 tsc는 해당 경로의 마지막 폴더로부터 주어진 이름과 `.ts`, `.d.ts` 확장자를 가진 파일을 검색합니다.

만일 일치하는 파일이 없고 주어진 이름이 폴더이름이라면 해당 폴더에 package.json 파일을 확인합니다.

있다면 package.json 파일의 `types` 필드에 적힌 경로를 가지는 파일들을 검색합니다.

package.json 파일과 `types` 필드가 없다면 해당 폴더로부터 `index.ts`, `index.d.ts` 파일을 검색합니다.

:::tip
`index.d.ts`, `index.ts` 파일이 default entry가 될 수 있는 이유가 여기에 있습니다.
:::

위의 경우에 모두 해당하지 않는다면 에러를 던집니다.

```ts
// relative import - Node strategy
Import statement: import '../program';
Import location: /root/dir/sub/example.ts

1. ../program.ts
2. ../program.d.ts
3. ../program/package.json -> [[types]] -> file path
4. ../program/index.ts
5. ../program/index.d.ts

6. Error: Cannot find module
```

#### non-relative import

`import` 구문에 이름이 명시되어 있다면 tsc는 현재 폴더에서 `node_modules` 폴더를 찾기 위해 위 과정을 그대로 수행합니다.

없다면 상위 폴더에서 `node_modules` 폴더를 찾아보는 행위를 재귀적으로 반복합니다.

```ts
// non-relative import - Node strategy
Import statement: import 'package';
Import location: /root/dir/sub/example.ts

1. /root/dir/sub/node_modules/package.ts
2. /root/dir/sub/node_modules/package.d.ts
3. /root/dir/sub/node_modules/package/package.json -> [[types]] ->
4. /root/dir/sub/node_modules/package/index.ts
5. /root/dir/sub/node_modules/package/index.d.ts

6. /root/dir/node_modules/package.ts
7. /root/dir/node_modules/package.d.ts
8. /root/dir/node_modules/package/package.json -> [[types]] ->
9. /root/dir/node_modules/package/index.ts
10. /root/dir/node_modules/package/index.d.ts

11. /root/node_modules/package.ts
12. /root/node_modules/package.d.ts
13. /root/node_modules/package/package.json -> [[types]] ->
14. /root/node_modules/package/index.ts
15. /root/node_modules/package/index.d.ts

16. Error: Cannot find module
```

### 그 외

위 두 방식은 tsconfig.json의 `moduleResolution` 필드로 설정할 수 있지만, 아래와 같은 필드로도 module resolution 방식을 지정할 수 있습니다.

#### baseUrl

[non-relative-import](../module-imports#non-relative-import)의 상대경로 기준이 되는 URL을 지정합니다.

이 때, tsconfig.json 파일에 대한 상대경로여야 합니다.

#### paths

`tsconfig.json` 의 `baseUrl` 필드값이 지정된 경우에만 사용할 수 있습니다.

특정 module을 import할 때 최종적으로 참조할 파일을 `baseUrl` 에 대한 상대경로로 지정합니다.

```json
{
  "compilerOptions": {
    "baseUrl": ".", // This must be specified if "paths" is.
    "paths": {
      "jquery": ["node_modules/jquery/dist/jquery"] // This mapping is relative to "baseUrl"
    }
  }
}
```

```ts
import A from "jquery";
// import A from "/node_modules/jquery/dist/jquery"
```

#### rootDirs

프로젝트 root를 여러 개로 지정할 때 사용합니다.

```json
{
  "compilerOptions": {
    "rootDirs": ["src/views", "generated/templates/views"]
  }
}
```

```text
src
 └── views
     └── view1.ts (can import "./template1", "./view2`)
     └── view2.ts (can import "./template1", "./view1`)

 generated
 └── templates
         └── views
             └── template1.ts (can import "./view1", "./view2")
```

## 참고자료

[Modules](https://www.typescriptlang.org/docs/handbook/modules.html)
