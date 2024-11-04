---
title: Package
date: 2024-11-03 18:12:44
categories:
  - Studies
  - Node
#tags:
---
소프트웨어가 배포되는 단위로, 보통 라이브러리, 개발툴(컴파일러, test ruuner, documentation generator) 등이 있습니다.

보통 패키지 안에는 JS 라이브러리, 라이브러리 테스트를 위한 테스트 코드들이 포함됩니다.

패키지는 파일 또는 `package.json`을 포함한 디렉토리이자 NPM Registry에서 지정된 범위에 공개되는 프로젝트를 가리킵니다.

모듈은 Node 환경에서 `require` 함수에 의해서 load될 수 있는 파일이나 디렉토리로, node_modules 디렉토리에 위치합니다.

`require` 함수에 의해서 load되려면 `.js` 파일이거나 `package.json`(main 필드를 가진)을 포함한 디렉토리여야 합니다.

`package.json`을 요구하지 않는다는 점에서 패키지와의 차이점이 있습니다.

## Registry

패키지들은 보통 [NPM Registry](https://www.npmjs.com/about)에 업로드되면 원격으로 프로젝트에 다운로드하여 사용할 수 있습니다.

registry에 등록된 패키지들을 구분할 때는 다음과 같은 name을 부여합니다.

- Global names : 전체 registry에서 유일한 이름
- Scoped names : 전체 registry에서 유일한 scope 내에서 이름

## 구조

보통 패키지는 다음과 같은 file system(폴더구조)를 가집니다.

![패키지 구조](/images/package_structure.png)

### package.json

패키지의 meta 데이터(name, version, author 등)와 dependency(패키지를 동작할 때 필요한 라이브러나 개발툴)들의 정보를 저장합니다.

### package-lock.json

`node_modules` dependency tree, `package.json`에 변화가 생길 때마다 npm에 의해서 생성 또는 업데이트되는 `.json` 파일입니다.

`package-lock.json`의 용도는 다음과 같습니다.

- `node_modules`의 과거 state로 time-travel이 가능합니다.
- 이미 설치된 패키지는 다시 중복되어 설치되지 않도록 metadata를 저장합니다.

### node_modules

dependency들의 소스가 위치한 폴더로, `npm install` 명령어를 실행하면 프로젝트 root에 node_modules 폴더를 생성하여(원래 없다면) `package.json`의 `dependencies` 필드에 명시된 패키지들을 버전에 맞게 설치합니다.

## Module Specifiers

`node_modules`에 설치된 패키지 내의 모듈들을 사용하려면, 해당 리소스를 구분하기 위한 **module specifier**와 모듈을 접근하기 위한 **import statement**을 사용해야 합니다.

`import` statement로는 static import과 dynamic import가 있습니다.

```js
// Static import
import { namedExport } from "https://example.com/some-module.js"; // (A)
console.log(namedExport);
```

```js
// Dynamic import
import("https://example.com/some-module.js") // (B)
  .then((moduleNamespace) => {
    console.log(moduleNamespace.namedExport);
  });
```

### Absolute Specifier

웹 상에서 호스팅되는 라이브러리에 접근할 때 사용하는 full URL로, 마지막에 파일 확장자가 붙습니다.

```js
"https://www.unpkg.com/browse/yargs@17.3.1/browser.mjs";
"file:///opt/nodejs/config.mjs";
```

### Relative Specifier

`/`, `./`, `../`로 시작하는 relative URL로, JS 엔진은 현재 모듈 URL을 기준으로 하는 absolute specifier로 변환하고 마지막에 파일 확장자가 붙입니다.

```js
"./sibling-module.js";
"../module-in-parent-dir.mjs";
"../../dir/other-module.js";
```

### Bare Specifier

protocol(scheme)과 domain이 없고 보통 `node_modules` 폴더에 설치된 npm 패키지를 참조할 때 사용합니다.

```js
"some-package";
"some-package/sync";
"some-package/util/files/path-tools.js";

"@some-scope/scoped-name";
"@some-scope/scoped-name/async";
"@some-scope/scoped-name/dir/some-module.mjs";
```

JS 엔진은 현재 모듈 URL을 기준으로 하는 absolute specifier로 변환합니다.
