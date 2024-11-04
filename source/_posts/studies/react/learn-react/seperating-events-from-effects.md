---
title: Seperating Events from Effects
date: 2024-11-04 08:18:32
categories:
  - Studies
  - React
  - Learn React
#tags:
---
Event handler는 사용자 이벤트마다 수행되고 props와 state의 변화여부가 반영될 필요없는 non-reactive 로직이 위치합니다.

Effect는 사용자 이벤트의 발생여부 상관없이 reactive value의 변화에 맞춰서 외부 시스템과의 동기화가 필요할 때마다 수행하는 reactive 로직이 위치합니다.
