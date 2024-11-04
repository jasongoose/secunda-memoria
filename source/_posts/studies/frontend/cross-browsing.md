---
title: Cross-Browsing
date: 2024-11-02 18:20:46
categories:
  - Studies
  - Frontend
#tags:
---
모든 브라우저에서 웹페이지가 의도한 대로 렌더링될 수 있도록 수행하는 작업을 의미합니다.

cross-browsing 체크가 필요한 이유는 다음과 같습니다.

- 브라우저별 default 스타일과 충돌할 수 있다.
- 브라우저별로 이해하지 못하는 HTML, CSS, JS 코드가 존재할 수 있다.
- 브라우저별로 렌더링 엔진이 다르다.

보통 관련 작업은 점유율이 높은 브라우저를 기준으로 개발을 진행하고 수정하면 됩니다.

- [caniuse](https://caniuse.com/) 사이트에서 브라우저별로 사용할 수 있는 코드인지 여부를 확인한다.
- [reset.css](https://meyerweb.com/eric/tools/css/reset/)로 브라우저별로 다른 default css를 모두 초기화 한다.
