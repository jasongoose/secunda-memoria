---
title: 5-2장. package.json으로 패키지 관리하기
date: 2024-10-31 22:31:02
categories:
  - Books
  - Node.js 교과서(개정 3판)
#tags:
---
`npm install`로 패키지를 설치하는 중에 WARN 메시지가 나올 수도 있는데 "ERROR"만이 진짜 에러이고 "WARN"은 단순한 경고일 뿐입니다.

npm은 패키지를 설치할 때 패키지에 있을 수 있는 취약점 즉, vulnerability을 자동으로 검사합니다.

```text
[숫자] [심각도] serverity vulnerability

To address all issues, run:
  npm audit fix --force

Run `npm autdit` for details.
```

`npm audit`은 패키지의 알려진 취약점을 검사할 수 있는 명령어로, `npm audit fix`를 입력하면 npm이 스스로 취약점을 알아서 수정하는데 주기적으로 수행해야 합니다.

`package-lock.json` 파일은 패키지를 설치, 수정, 삭제할 때마다 패키지들 간의 정확한 내부 의존 관계를 저장합니다.

`require.resolve`로 특정 모듈을 어디에서 불러오는지 확인할 수 있는데, 내장 모듈의 경우 이름만 표시되고 npm에서 설치된 경우 전체 경로가 표시됩니다.

```js
require.resolve("fs");
require.resolve("express");
```

Node.js 내장 모듈과 npm 패키지의 이름이 같은 경우, 명시적으로 내장 모듈을 사용함을 밝히고 싶다면 노드 프로토콜(`node:`)을 사용하면 됩니다.

```js
const fs = require("node:fs");
```

### npm install -D

`npm install -D` 명령어로 개발용 패키지를 설치할 수도 있는데, 실제 배포 시에는 사용되지 않고 개발 중에만 사용되는 패키지들입니다.

`package.json`의 `devDependencies` 속성에서 개발용 패키지들만 따로 관리합니다.

`peerDependencies`가 `package.json`에 있는 경우도 있는데, 직접적으로 import(또는 require)하지는 않지만 설치되어 있다는 가정 하에 코드를 작성했다는 의미를 가집니다.

`peerDependencies`와 다른 버전의 패키지가 설치되어 있다면 `ERESOLVE unable to resolve dependency tree` 에러 메시지가 표시되므로 애초에 `peerDependencies`가 서로 충돌하는 패키지를 같이 설치하지 않습니다.

### npm install

`npm install` 명령어는 `--global` 옵션도 있는데, npm이 설치되어 있는 폴더의 경로가 보통 시스템 환경변수에 등록되어 있으므로 전역으로 설치한 패키지는 콘솔의 명령어로 사용할 수 있습니다.

대부분 명령어로 사용하기 위해 전역으로 패키지를 설치합니다.

`npm i` 명령어를 실행할 때마다 `package.json`과 `package-lock.json`이 변하므로 실제 서비스를 배포할 때는 `npm i` 대신 `npm ci` 명령어를 사용해서 `package-lock.json`에 적힌대로 설치합니다.

전역으로 패키지를 설치하면 `package.json`에 기록되지 않는 성질을 기피한다면 `npx` 명령어를 앞에 붙여서 실행하면 됩니다.

```sh
npm install -D rimraf
npx rimraf node_modules
```