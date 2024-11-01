---
title: The Ultimate Guide to Optimizing JavaScript for Quick Page Loads
date: 2024-11-01 22:21:42
categories:
  - Articles
#tags:
---
프론트엔드 개발의 목적은 사용자에게 필요한 정보를 최대한 빠르고 정확하게 전달하여 상호작용성(interactivity)을 부여하는 겁니다.

이를 위한 웹앱의 최적화 및 성능개선은 필수이고 여기에 작용하는 변수들로는 사용자의 리소스와 관련이 있습니다.

- CPU 사양
- GPU 사양
- 메모리 사양
- 네트워크 품질
- ...

초기 페이지 로딩 속도를 높이고 다양한 시각적 효과를 부여하려면 DOM 조작이 반드시 필요하므로 브라우저가 실행할 JS 코드의 용량을 줄이고 연산 복잡도를 낮추면 됩니다.

## Reducing the JavaScript in your code

### code splitting

전체 JS 코드들을 분해하여 한번에 다운로드 받을 chunk의 용량을 줄입니다.

### lazy loading

viewport에 노출될 차례가 될 때에만 관련 chunk나 이미지, 동영상 같은 asset들을 다운받습니다.

### caching

HTTP caching, Service Worker caching, long-term caching 등을 사용하여 동일한 요청에 대한 응답 리소스를 따로 cache에 저장하여 network congestion을 방지합니다.

## Decreasing the costs of parsing/compiling and execution of JavaScript

### Web worker

브라우저에서 컴파일된 JS 코드를 실행하는 thread는 오직 1개이기 때문에 압축된 코드라도 중간에 block될 수 있습니다.

브라우저의 Web worker를 사용하여 background thread도 JS 코드 실행에 동참하여 blocking을 방지할 수 있습니다.

## 참고

[The Ultimate Guide to Optimizing JavaScript for Quick Page Loads](https://www.builder.io/blog/the-ultimate-guide-to-optimizing-javascript-for-quick-page-loads)
