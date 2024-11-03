---
title: Choosing the State Structure
date: 2024-11-04 07:58:43
categories:
  - Studies
  - React
  - Learn React
#tags:
---
state를 어떤 구조로 설계해야 쉽고 오류없이 업데이트 하는지에 대해서 언제나 고민해봐야 합니다.

1. 연관된 state들은 하나의 state 변수로 관리합니다.

    - 배열로 묶든, 객체로 묶든
    - 동일한 handler에 의해서 setter가 같이 실행되는지 여부를 연관성 기준으로 합니다.

2. 불가능한 상태값 조합이 나타나지 않도록 state간의 모순이 없어야 합니다.

3. 다른 컴포넌트의 state나 props에 의해서 연산될 수 있는 값들은 상태로 두지 않습니다.

4. 여러 state 또는 객체 내의 중첩 객체 내에서의 속성별 state는 동일한 데이터(특히 객체)를 참조하면 안됩니다.

5. 깊게 중첩된 객체 구조는 피합니다.
    - [Map](../../javascript/data_structures/map.md), [Set](../../javascript/data_structures/set.md), [WeakMap](../../javascript/data_structures/weakmap.md), [WeakSet](../../javascript/data_structures/weakset.md) 등 다양한 interface들도 활용합니다.
    - 깊이 위치한 속성값을 수정한 새로운 객체를 만들어내는 과정이 번거로우니 중첩된 객체를 flatten하여 관리합니다.

## 참고자료

[Learn React](https://react.dev/learn)