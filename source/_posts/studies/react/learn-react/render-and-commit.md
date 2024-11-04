---
title: Render and Commit
date: 2024-11-04 08:16:13
categories:
  - Studies
  - React
  - Learn React
#tags:
---
React 컴포넌트가 화면에 표시되기까지 과정은 크게 4단계로 구분할 수 있습니다.

## Trigger

`createRoot(rootNode).render(<App />)`의 호출이나 state의 변화 또는 부모 컴포넌트의 리렌더링으로 렌더링이 시작됩니다.

여기서 여러 state들의 변화를 한번에 반영하기 위해서 리렌더링 전에 업데이트 항목들은 queueing되어 비동기적으로 처리됩니다.

## VDOM Rendering

root로부터 컴포넌트를 정의한 함수를 재귀적으로 호출하여 새로운 vdom을 만드는 과정으로, 자식 컴포넌트가 없는 leaf들의 렌더링을 마쳐야 완전히 종료됩니다.

## Committing

최초 렌더링시, DOM API의 `appendChild` 메서드로 컴포넌트가 Document에 mount됩니다.

리렌더링시, 직전 vdom과 비교하여 최소한의 연산으로 새로운 vdom에 맞춰서 실제 DOM의 일부만 업데이트합니다.

vdom간의 diff를 계산하는 부분은 VDOM Rendering 단계에서 수행됩니다.

## Browser Rendering

DOM을 업데이트했으니 reflow와 repaint을 거쳐서 화면에 표시합니다.
