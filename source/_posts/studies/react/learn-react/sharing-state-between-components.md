---
title: Sharing State Between Components
date: 2024-11-04 08:16:13
categories:
  - Studies
  - React
  - Learn React
#tags:
---
sibling 컴포넌트들 사이에서 공유해야할 state가 있다면 (1) 공통 state를 가장 가까운 상위 컴포넌트로 옯기고 (2) 상태값과 해당 상태값을 업데이트하기 위한 setter를 props나 context로 전달하면 됩니다.

위와 같은 개발방식을 "lifting state up"이라고 합니다.
