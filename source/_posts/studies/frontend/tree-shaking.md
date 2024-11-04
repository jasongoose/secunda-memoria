---
title: Tree Shaking
date: 2024-11-02 18:21:34
categories:
  - Studies
  - Frontend
#tags:
---
JS 파일 내에서 사용하지 않는 dead code를 제거하는 작업을 가리키는 용어입니다.

파일 간의 주고받는 module 내에서 사용되는지 여부를 확인하기 위해서 ES2015 `import`, `export` 문에 의존적입니다.

`.js` 번들파일 크기를 줄이고 간결한 코드 구조를 구현하기 위해서 필요한 작업으로 보통 [Rollup](https://rollupjs.org/), [Webpack](https://webpack.kr/) 같은 모듈 번들러는 번들링 과정에서 Tree Shaking을 수행합니다.
