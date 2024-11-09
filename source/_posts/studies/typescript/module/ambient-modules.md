---
title: Ambient Modules
date: 2024-11-03 19:28:56
categories:
  - Studies
  - TypeScript
  - Module
#tags:
---
TS 프로젝트에서 JS 라이브러리를 사용하려면 각 모듈에서 export하는 객체, 상수, 변수, 함수, 클래스 등에 대응되는 `type alias`와 `interface`들의 정보가 tsc로 전달되어야 합니다.

이를 위해서는 정의가 없는 선언들 즉, “ambient declaration”들을 담은 top-level `.d.ts` 파일과 `declare` 키워드가 필요합니다.

## declare module

라이브러리에 존재하는 모듈에서 제공하는 객체, 상수, 변수, 함수, 클래스 등에 대응되는 type alias들, interface들의 정보는 `declare module` 블럭 안에 다음과 같이 작성하면 됩니다.

```ts
// node.d.ts

declare module "url" {
  export interface Url {
    protocol?: string;
    hostname?: string;
    pathname?: string;
  }
  export function parse(
    urlStr: string,
    parseQueryString?,
    slashesDenoteHost?
  ): Url;
}

declare module "path" {
  export function normalize(p: string): string;
  export function join(...paths: any[]): string;
  export var sep: string;
}
```

그럼 이제 `.ts` 파일에서 아래와 같이 사용하면 됩니다.

```ts
/// <reference path="node.d.ts"/> ← 컴파일에 포함시킬 경우에만 작성, 그 외는 생략
import * as URL from "url";

let myUrl = URL.parse("https://www.typescriptlang.org");
```

`declare module` syntax로 자세한 타입 정보없이 빠르게 `.js` 모듈을 사용하고 싶다면 `src/`에 아래와 같이 쓰면 됩니다.

```ts
// sth.d.ts
declare module "hot-new-module";
```

이와 같이 선언된 모듈로부터 import한 것들은 모두 `any` 타입을 가지지만, 실제 구현된 기능들은 `.js` 모듈에 정의되어 있으므로 코드는 문제없이 실행됩니다.

```ts
import x, { y } from "hot-new-module";

x(y);
```

## Wildcard Module Declaration

뭔가 `.js` 모듈 이름들 간의 패턴이 있다면 wildcard(\*) 문자로 표시할 수 있습니다.

```js
declare module "*!text" {
  const content: string;
  export default content;
}

// Some do it the other way around.
declare module "json!*" {
  const value: any;
  export default value;
}
```

아래와 같이 import 구문에 모듈의 이름뿐만 아니라 모듈에 접근하는 경로를 포함해서도 사용할 수 있습니다.

```ts
import fileContent from "./xyz.txt!text";
import data from "json!http://example.com/data.json";

console.log(data, fileContent);
```
