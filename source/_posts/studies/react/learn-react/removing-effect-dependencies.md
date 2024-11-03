---
title: Removing Effect Dependencies
date: 2024-11-04 08:15:52
categories:
  - Studies
  - React
  - Learn React
#tags:
---
## 불필요한 dependency

Effect를 구현할 때 사용하는 reactive value들은 모두 linter에 의해서 dependency로 지정됩니다.

하지만 막상 코드를 돌려보면 특정 dependency에 의해서 Effect가 불필요하게 재실행되는 이슈가 발생할 수 있습니다.

이 부분을 해결하려면 해당 dependency를 제거하여 일부 로직을 event handler나 새로운 Effect로 옮기면 됩니다.

## object as dependency

Effect의 dependency에 함수를 포함한 객체를 전달하면 리렌더링될 때마다 새로운 객체가 생성되므로 Effect의 빈도수가 예상보다 많아지는 현상이 발생합니다.

그래서 객체는 dependency로 지정하지 않고 Effect 내부 또는 컴포넌트 함수 외부(상수인 경우)에 두어서 non-reactive value로 만들거나, 속성 단위로 dependency로 전달하면 됩니다.

`useCallback` hook을 사용하면 특정 dependency들 기반으로 함수 객체를 caching할 수 있습니다.

## 참고자료

[Learn React](https://react.dev/learn)