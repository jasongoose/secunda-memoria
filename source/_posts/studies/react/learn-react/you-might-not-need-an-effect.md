---
title: You Might Not Need an Effect
date: 2024-11-04 08:21:02
categories:
  - Studies
  - React
  - Learn React
#tags:
---
1. 렌더링 중에 계산할 부분은 Effect에 넣지 않습니다.

2. 복잡한 연산의 결과값을 caching할 때는 `useMemo`를 사용합니다.

3. 전체 컴포넌트의 state를 초기화할 때는 `key` attribute을 사용합니다.

    - 동일한 컴포넌트라도 key가 다르면 리렌더링이 아닌 mount가 이루어집니다.
    - 하나의 컴포넌트가 props에 따라 다른 상태값을 가지게(렌더링하게) 하려면 반드시 필요합니다.

4. props의 변화에 맞춰서 특정 state를 업데이트하려면 Effect가 아닌 렌더링 중에 업데이트합니다.

5. 컴포넌트의 렌더링 이후에 수행할 로직들은 Effect에 구현하고 그 외의 로직들은 event handler에 구현합니다.

6. 다수 state들의 업데이트는 개별 state를 dependency로 갖는 Effect가 아닌 단일 event handler에서 일괄적으로 수행합니다.

7. 다른 컴포넌트들 간의 state 동기화가 필요하면 가장 가까운 상위 컴포넌트에서 관리하여 props나 context로 전달하는 state lifting을 적용합니다.

8. Effect에서 fetch를 진행하는 경우, race condition이 일어나지 않도록 re-mount를 고려하여 cleanup을 구현합니다.
