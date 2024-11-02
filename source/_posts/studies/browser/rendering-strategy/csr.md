---
title: CSR
date: 2024-11-02 00:07:10
categories:
  - Studies
  - Browser
  - Rendering Strategy
#tags:
---
웹 페이지 렌더링을 서버가 아닌 브라우저에서 수행합니다.

서버가 사용자로부터 페이지 요청을 받으면, 뼈대 페이지(`index.html`)에 필요한 static 리소스(`.css`, `.js`)들을 지정하여 응답으로 전달합니다.

```html
<!DOCTYPE html>
<html>
  <head>
    <meta />
    <link rel="stylesheet" href="/path/to/css" />
  </head>
  <body>
    <div id="root"></div>
  </body>
  <script src="path/to/js"></script>
</html>
```

이후에 필요한 데이터는 `.js` 파일에 명시된 API 호출함수들을 실행하여 fetch하고 페이지 렌더링이 완료되면 브라우저에 표시됩니다.

사용자가 다른 경로로 navigate할 때마다 서버로 요청하는 대신 경로에 맞게끔 브라우저에서 리렌더링을 진행합니다.

### 장점

- 사용자 동작에 따라 동적인 렌더링이 가능하여 UX를 향상시킬 수 있습니다.
- 요청 빈도 수가 훨씬 적어서 서버 부담이 감소합니다.

### 단점

- 첫 렌더링에 필요한 HTML과 static 리소스들을 받을 때까지 시간이 걸립니다.
- 처음부터 완성된 페이지가 렌더링되지 않으므로 SEO 최적화가 어렵습니다.
