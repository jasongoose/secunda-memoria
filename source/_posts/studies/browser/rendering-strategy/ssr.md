---
title: SSR
date: 2024-11-02 00:07:13
categories:
  - Studies
  - Browser
  - Rendering Strategy
#tags:
---
브라우저에서 요청하면 서버에서 모든 컨텐츠가 표시된 페이지를 응답으로 전달합니다.

필요한 리소스 fetch도 서버 단에서 수행하고 사용자가 다른 경로로 navigate를 요청할 때마다 새로운 페이지를 생성하여 응답합니다.

### 장점

- 초기 loading 속도가 빨라서 사용자가 컨텐츠를 신속히 확인할 수 있습니다.
- 모든 검색엔진에 대한 SEO가 가능합니다.

### 단점

- 매번 페이지를 요청할 때마다 refresh가 필요하므로 UX가 떨어집니다.
- 요청 빈도 수가 높아서 트래픽과 서버 부하가 증가합니다.

## 참고자료

[📜 CSR-SSR](https://velog.io/@namezin/CSR-SSR)

[📜 CSR vs. SSR](https://dev.to/jeremypanjaitan/ssr-vs-csr-2617)