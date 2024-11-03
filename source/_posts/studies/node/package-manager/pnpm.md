---
title: Pnpm
date: 2024-11-03 18:14:15
categories:
  - Studies
  - JavaScript
  - Node
#tags:
---
node 패키지들을 관리하기 위한 툴로, 로컬 파일 시스템에 버전별로 패키지를 하나만 저장하고 node 프로젝트들이 필요한 버전을 symlink를 통해 참조하는 메커니즘을 가집니다.

여기서 원본 패키지들이 저장된 단일 폴더를 content-addressable store(cas)라고 부르고 default path는 `~/Library/pnpm/store/v3`입니다.

기존 npm이나 yarn에서 발생했던 패키지 중복문제를 해결해서 디스크 공간을 절약할 수 있는 효과가 있습니다.

## node_modules의 구조

`pnpm i` 로 dependency들을 설치하면 프로젝트 `node_modules`는 아래와 같은 구조를 가집니다.

```text
node_modules
├- .pnpm
├- .modules.yaml
├- pkg1 -> .pnpm/pkg1@1.2.3/node_modules/pkg1
|  ├- package.json
|     ├- "dependencies"
|        ├- "dep1": "1.1.1"
|
├- pkg2 -> .pnpm/pkg2@3.2.1/node_modules/pkg2
|  ├- package.json
|     ├- "version": "3.2.1"
|     ├- "dependencies"
|        ├- "dep2": "1.1.2"
├- ...
```

`node_module/.pnpm`에 위치한 pkg1와 pkg2와 같은 개별 dependency들은 아래와 같이 flat 구조로 저장됩니다.

```text
node_modules
├- .pnpm
   ├- pkg1@1.2.3
   |  ├- node_modules
   |.    ├- pkg1 -> cas/pkg1
   |     ├- dep1 -> ../../dep1@1.1.1/node_modules/dep1
   |
   ├- pkg2@3.2.1
   |  ├- node_modules
   |.    ├- pkg2 -> cas/pkg2
   |     ├- dep2 -> ../../dep2@1.1.2/node_modules/dep2
   |
   ├- dep1@1.1.1
   |  ├- node_modules
   |     ├- dep1 -> cas/dep1
   |
   ├- dep2@1.1.2
      ├- node_modules
         ├- dep2 -> cas/dep2
```

위에서 볼 수 있듯이 실제로 `node_modules` 또는 `.pnpm`에 있는 pkg1와 pkg2, 그리고 관련 dep들은 모두 cas에 저장된 원본 패키지들을 참조하는 symlink입니다.


이러한 성격을 반영한듯 `.pnpm`을 "virtual store directory"라고 칭합니다.

## peerDependencies Resolution

패키지가 `peerDependencies`를 가지는 경우, `.pnpm`의 구성은 어떻게 달라질까?

`peerDependencies`는 해당 패키지를 사용하는 부모 패키지에 의해서 구체화되는 성질을 가지는데, 동일한 패키지를 사용하는 2개 이상의 부모 패키지에서 실제로 사용하는 peerDependency 패키지의 버전이 다른 경우가 생길 수 있습니다.

예를 들어, 패키지 foo의 `peerDependencies`가 아래와 같다면…

```text
peerDependencies
├- bar@^1
├- baz@^1
```

foo를 사용하는 두 부모 패키지들(foo-parent-1, foo-parent-2)에서 foo가 사용할 baz의 버전이 다를 수 있다는 겁니다.

```text
node_modules
├- foo-parent-1
|  ├- bar@1.0.0
|  ├- baz@1.0.0
|  ├- foo@1.0.0
|
├- foo-parent-2
   ├- bar@1.0.0
   ├- baz@1.1.0
   ├- foo@1.0.0
```

위와 같은 경우, baz 버전별로 foo의 virtual store를 분리하면 됩니다.

```text
node_modules
└── .pnpm
    ├── foo@1.0.0_bar@1.0.0+baz@1.0.0
    │   └── node_modules
    │       ├── foo
    │       ├── bar   -> ../../bar@1.0.0/node_modules/bar
    │       ├── baz   -> ../../baz@1.0.0/node_modules/baz
    │
    ├── foo@1.0.0_bar@1.0.0+baz@1.1.0
    │   └── node_modules
    │       ├── foo
    │       ├── bar   -> ../../bar@1.0.0/node_modules/bar
    │       ├── baz   -> ../../baz@1.1.0/node_modules/baz
    │
    ├── bar@1.0.0
    ├── baz@1.0.0
    ├── baz@1.1.0
    ├── qux@1.0.0
    ├── plugh@1.0.0
```

현재 패키지는 `peerDependencies`는 없지만 특정 dependency에는 `peerDependencies`를 가지는 경우도 비슷하게(?) 처리하면 됩니다.

예를 들어, 패키지 a는 `peerDependencies`는 없는데 dependency인 패키지 b에 `peerDependencies`로 `c@^1(c@1.0.0, c@1.1.0)`이 있는 상태라면, `.pnpm`은 다음과 같이 구성됩니다.

```text
node_modules
└── .pnpm
    ├── a@1.0.0_c@1.0.0
    │   └── node_modules
    │       ├── a
    │       └── b -> ../../b@1.0.0_c@1.0.0/node_modules/b
    ├── a@1.0.0_c@1.1.0
    │   └── node_modules
    │       ├── a
    │       └── b -> ../../b@1.0.0_c@1.1.0/node_modules/b
    ├── b@1.0.0_c@1.0.0
    │   └── node_modules
    │       ├── b
    │       └── c -> ../../c@1.0.0/node_modules/c
    ├── b@1.0.0_c@1.1.0
    │   └── node_modules
    │       ├── b
    │       └── c -> ../../c@1.1.0/node_modules/c
    ├── c@1.0.0
    ├── c@1.1.0
```


## 참고자료

[Motivation](https://pnpm.io/motivation)

[Flat node_modules is not the only way](https://pnpm.io/blog/2020/05/27/flat-node-modules-is-not-the-only-way)

[Symlinked node_modules structure](https://pnpm.io/symlinked-node-modules-structure)