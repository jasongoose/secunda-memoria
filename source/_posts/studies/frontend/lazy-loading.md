---
title: Lazy Loading
date: 2024-11-02 18:21:02
categories:
  - Studies
  - Frontend
#tags:
---
웹앱에서 사용하는 리소스들을 페이지의 첫 렌더링이 아닌 필요할 때마다 요청하는 기법으로, 브라우저의 [CRP](../../browser/populating-the-page/crp) 처리에 의한 페이지 로딩 지연을 줄이기 위한 목적으로 사용합니다.

## Code Splitting

어떤 코드를 여러 개의 bundle이나 컴포넌트들로 나누는 기법으로, 페이지 렌더링 초기 단계에서 다운받을 파일 또는 모듈들의 용량을 줄이기 위해서 활용됩니다.

### entry point splitting

웹앱에서 처음 실행할 코드를 여러 개로 나누는 방법

### dynamic splitting

[dynamic import](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/import) 구문으로 조건에 따라 리소스를 load하는 방법

## Inline script

`<script></script>`에 아래와 같은 attribute을 추가하면 다운받을 JS 코드를 모듈로 간주하는 동시에, DOM 파싱이 완료된 이후에 코드를 실행하여 중간 blocking을 막을 수 있습니다.

```html
<!DOCTYPE html>
<html>
  <head></head>
  <body>
    <script type="module" src="main.js"></script>
  </body>
</html>
```

## CSS

`.css` 경우, [render-blocking 리소스](../../browser/populating-the-page/crp#css-object-model)로 간주됩니다.

파싱 속도는 브라우저에 의존적이라 제어가 불가능하지만 아래와 같이 [media query](https://developer.mozilla.org/en-US/docs/Web/CSS/@media#media_types)를 지정하면 현재 device의 상태에 따라 필요한 스타일만 다운받으므로 blocking 시간을 줄일 수 있습니다.

```html
<link href="style.css" rel="stylesheet" media="all" />
<link href="portrait.css" rel="stylesheet" media="(orientation:portrait)" />
<link href="print.css" rel="stylesheet" media="print" />
```

## Fonts

font는 default로 [Render Tree](../../browser/populating_the_page/crp#render-tree)가 생성된 이후에 요청되기 때문에 text 렌더링이 지연되는 경우가 종종 발생합니다.

지연을 줄이는 방법으로 CSS `font-display` 속성을 지정하거나 [CSS Font Loading API](https://developer.mozilla.org/en-US/docs/Web/API/CSS_Font_Loading_API)를 사용할 수 있습니다.

아니면 `<link rel="preload" />`를 지정하여 렌더링 초반에 리소스를 요청할 수 있습니다.

## Images, iframes

필요한 이미지나 `<iframe></iframe>`들이 viewport에 노출될 타이밍에 리소스를 요청하는 방식입니다.

`loading=”lazy”` attribute을 추가하거나 [Intersection Observer API](https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserver)로 개발자 입장에서 더 유연하게 로직을 설계할 수 있습니다.

```html
<img src="image.jpg" alt="" loading="lazy" />
<iframe src="video-player.html" title="" loading="lazy"></iframe>
```

## 참고자료

[Lazy Loading](https://developer.mozilla.org/en-US/docs/Web/Performance/Lazy_loading)