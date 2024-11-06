---
title: Browsing Context
date: 2024-11-03 23:12:25
categories:
  - Studies
  - Web Terms
#tags:
---
브라우저가 HTML 문서를 표시할 환경을 가리키는 용어로 탭, 창, 페이지 내부 `<frame />`, `<iframe />` 등이 여기에 해당합니다.

browsing context는 현재 화면에 표시되는 문서의 origin과 과거에 표시되었던 문서들의 순서(history)를 가집니다.

browsing context 사이의 통신은 [BroadcastChannel API](../../browser/web-api/broadcast-channel), [Window.postMessage](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage) 등으로 구현할 수 있습니다.
