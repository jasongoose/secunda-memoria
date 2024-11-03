---
title: package.json
date: 2024-11-03 18:20:01
categories:
  - Studies
  - JavaScript
  - Node
#tags:
---
현재 package에 관한 정보를 기술한 `.json` 파일로 패키지를 개발하거나 다운받아 사용할 때 필요한 필드들을 정의합니다.

## files

본 패키지가 다른 패키지의 `dependencies` 중 하나로 설치되었을 때, 포함시킬 파일이나 폴더들의 name pattern을 JSON array로 나열한 필드입니다.

default value는 `*`입니다.

### 필드값과 상관없이 언제나 패키지에 포함되는 파일, 폴더들

- `package.json`
- `README.md`
- `LICENSE`, `LICENCE`
- `main` 필드에 명시된 파일

### 필드값과 상관없이 언제나 패키지에서 제외되는 파일, 폴더들

- `.git`
- `CVS`
- `.svn`
- `.hg`
- `.lock-wscript`
- `.wafpickle-N`
- `.*.swp`
- `.DS_Store`
- `._*`
- `npm-debug.log`
- `.npmrc`
- `node_modules`
- `config.gypi`
- `.orig`
- `package-lock.json`

## main

패키지를 설치하여 module로서 import했을 때 가장 먼저 실행되는 파일의 경로를 지정하는 필드입니다.

패키지 root에 대한 상대경로로 명시되고 해당 파일을 실행하여 export 되는 객체로 여러 기능들을 구현할 수 있습니다.

default value는 `index.js` 입니다.

## scripts

프로젝트 내에서 실행할 수 있는 npm script 명령어들을 지정하는 필드입니다.

npm built-in script나 패키지 life cycle event hook으로서 실행하는 script가 위치하고 보통 `npm run <script 이름>` 형식으로 실행합니다.

### Pre/Post Scripts

특정 script 명령어들을 실행하기 전과 후에 자동으로 실행하는 script로, `pre*`, `post*` prefix를 가집니다.

```json
{
  "scripts": {
    "precompress": "{{ executes BEFORE the `compress` script }}",
    "compress": "{{ run command to compress files }}",
    "postcompress": "{{ executes AFTER `compress` script }}"
  }
}
// npm run compress를 실행하면 precompress, postcompress도 자동으로 실행합니다.
```

### Life Cycle Scripts

패키지 life cycle event가 발생할 때마다 실행하는 script로, 예시는 다음과 같습니다.

- prepare(≥npm@4.0.0)
- prepublishOnly
- prepack
- postpack

npm CLI command 하나를 실행할 때마다 life cycle script들이 순서대로 실행됩니다.

## exports

[exports](../package/package_exports.md)

## imports

[imports](../package/package_imports.md)

## dependencies

패키지의 구성모듈들이 사용하는 패키지들의 name과 버전을 명시한 필드로, `npm install` 명령어로 패키지들을 설치할 수 있습니다.

보통 번들링, 빌드과정에 포함되는 패키지들이 위치합니다.

`dependencies`의 가능한 값들을 나열하면 다음과 같습니다.

- semantic versioning
- URL
- Git URL
- Github URL
- local path w/ `file:` scheme

## devDependencies

transpiler, compiler 등 개발 중에 사용하는 패키지들의 name과 버전을 명시한 필드로 `npm install -D` 또는 `npm link` 명령어로 패키지들을 설치할 수 있습니다.

```json
{
  "name": "ethopia-waza",
  "description": "a delightfully fruity coffee varietal",
  "version": "1.2.3",
  "devDependencies": {
    "coffee-script": "~1.6.3"
  },
  "scripts": {
    "prepare": "coffee -o lib/ -c src/waza.coffee"
  },
  "main": "lib/waza.js"
}
```

## peerDependencies

현재 패키지를 다운로드한 프로젝트(부모 패키지)에서 제대로 동작하기 위해 필요한 dependency들을 지정하는 필드입니다.

예를 들어, `react@17.0.0`을 기반으로 만든 패키지에 다음과 같은 `peerDependencies`를 지정한다고 해봅시다.

```json
// my-ui-library package.json
{
  "peerDependencies": {
    "react": "^17.0.0"
  }
}
```

`my-ui-library` 패키지를 `react@16.0.0`를 사용하는 프로젝트에 설치한다면 아래와 같은 node_modules tree가 생성됩니다.

```text
node_modules
ㄴ react ^16.0.0 (dependancy)
ㄴ my-ui-library ^0.0.1 (dependancy)
  ㄴ node_modules
    ㄴ react ^17.0.0 (peer dependancy)
```

여기서 npm, yarn 등의 package manager는 `react@17.0.0`가 프로젝트에 설치되어야 한다는 메시지를 남거나 버전이 맞을 때까지 `my-ui-library`의 설치를 막습니다.

## 참고자료

[About packages and modules](https://docs.npmjs.com/about-packages-and-modules)

[package-lock.json](https://docs.npmjs.com/cli/v8/configuring-npm/package-lock-json)

