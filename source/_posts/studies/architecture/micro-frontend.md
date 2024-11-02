---
title: Micro Frontend
date: 2024-11-02 00:02:56
categories:
  - Studies
  - Architecture
#tags:
---
전체 앱에 대해서 적용했던 [MSA](./micro-service)를 FE 영역에 적용한 케이스로 이해할 수 있습니다.

기존 monolith 기반의 FE 프로젝트를 더 작은 서비스 단위(이하 svc)로 분리하여 개발 + 테스트 + 배포를 독립적으로 수행하면서 사용자 경험에는 거의 변화가 없도록 구성합니다.

![Micro Frontend](/images/mfe.jpeg)

## Benefits

### Incremental Upgrades

이슈에 대응하기 위한 결정을 svc 단위로 내리면서 구조, dependency들, 사용자 경험 등을 점진적으로 개선시키거나 새로운 기술스택을 시도할 수 있습니다.

### Simple, decoupled codebases

svc를 구현하는 코드는 간결하고 다른 svc와 context를 공유하는 coupling이 없기 때문에 개발 편의성을 보장합니다.

이러한 특성은 개발자로 하여금 svc들 사이에서 어떤 데이터와 이벤트를 어떻게 주고받아야 하는지에 대해서 고민하도록 만들어줍니다.

### Independent deployment

개별 svc는 테스트 + 빌드 + 배포작업을 수행하는 독립적인 CI/CD 파이프라인을 가집니다.

### Autonomous teams

사용자가 바라보는 **서비스의 파트 즉, 페이지 컨텐츠 단위**로 목적조직을 형성하여 개발부터 배포까지 책임을 부여합니다.

필요하다면 페이지 컨텐츠별로 서버를 운영하여 별도 도메인을 가지도록 만들 수도 있습니다.

## Integration approaches

일반적으로 Micro Frontend(이하 mfe)를 구성할 때 아래와 같은 template 역할을 가진 container application(이하 container)이 있습니다.

- 페이지별 header와 footer 등 공통 레이아웃 영역을 렌더링 한다.
- 인증이나 페이지 이동(navigation) 등 svc들간의 횡단을 요구하는(cross-cutting) 이슈들을 처리한다.
- svc별 페이지 컨텐츠의 렌더링 시점과 위치를 제어한다.

![Micro UI](/images/micro_ui.jpeg)

### Build-time integration

svc 컨텐츠들을 npm 패키지로 만들어서 container의 dependency로 지정하는 방식으로 통합할 수 있습니다.

```json
{
  "name": "@feed-me/container",
  "version": "1.0.0",
  "description": "A food delivery web app",
  "dependencies": {
    "@feed-me/browse-restaurants": "^1.2.3",
    "@feed-me/order-food": "^4.5.6",
    "@feed-me/user-profile": "^7.8.9"
  }
}
```

하지만 이 방식은 기존 monolith 방식처럼 일부 svc의 사소한 변화를 반영하기 위해서 해당 svc의 빌드와 배포뿐만 아니라 container의 빌드와 배포까지 필요하다는 번거로움이 있습니다.

### Run-time integration via iframes

`<iframe></iframe>`을 사용하면 페이지 컨텐츠별 스타일이나 전역변수의 오버라이딩을 방지할 뿐만 아니라 svc별 배포만으로 컨텐츠 업데이트가 가능하다는 이점을 가집니다.

```html
<html>
  <head>
    <title>Feed me!</title>
  </head>
  <body>
    <h1>Welcome to Feed me!</h1>

    <iframe id="micro-frontend-container"></iframe>

    <script type="text/javascript">
      const microFrontendsByRoute = {
        "/": "https://browse.example.com/index.html",
        "/order-food": "https://order.example.com/index.html",
        "/user-profile": "https://profile.example.com/index.html",
      };

      const iframe = document.getElementById("micro-frontend-container");
      iframe.src = microFrontendsByRoute[window.location.pathname];
    </script>
  </body>
</html>
```

다만 페이지 navigation이나 deep-linking 등 페이지 내 component 단위 통합이 필요한 작업을 복잡하게 만들 수 있다는 단점이 있고, 이는 페이지에 반응성을 부여하는 부분에서도 동일하게 나타납니다.

### Run-time integration via Javascript

svc별 컨텐츠를 `<script></script>`로 load하고 컨텐츠를 렌더링하는 전역함수(entry point)를 container에 제공하는 방식입니다.

container에서는 적절한 시점과 위치에 특정 렌더링 함수를 호출하여 컨텐츠를 그려냅니다.

```html
<html>
  <head>
    <title>Feed me!</title>
  </head>
  <body>
    <h1>Welcome to Feed me!</h1>

    <!-- These scripts don't render anything immediately -->
    <!-- Instead they attach entry-point functions to `window` -->
    <script src="https://browse.example.com/bundle.js"></script>
    <script src="https://order.example.com/bundle.js"></script>
    <script src="https://profile.example.com/bundle.js"></script>

    <div id="micro-frontend-root"></div>

    <script type="text/javascript">
      // These global functions are attached to window by the above scripts
      const microFrontendsByRoute = {
        "/": window.renderBrowseRestaurants,
        "/order-food": window.renderOrderFood,
        "/user-profile": window.renderUserProfile,
      };
      const renderFunction = microFrontendsByRoute[window.location.pathname];

      // Having determined the entry-point function, we now call it,
      // giving it the ID of the element where it should render itself
      renderFunction("micro-frontend-root");
    </script>
  </body>
</html>
```

위 방식은 build-time integration과는 다르게 개별 svc에서 bundle 파일을 독립적으로 배포할 수 있고 iframe 방식과는 다르게 컨텐츠 렌더링 방식을 JS로 더 유연하게 제어할 수 있다는 이점이 있습니다.

## Styling

svc별 컨텐츠에 적용하는 stylesheet들이 모두 동일한 페이지에 load되면 CSS의 cascading 특성 때문에 스타일이 망가지고 분산된 팀들 사이에서 해당 원인을 파악하는데 시간이 걸릴 수도 있습니다.

그래서 스타일 같은 경우 CSS 전처리기 중 하나인 [SASS](https://sass-lang.com/)나 동적으로 스타일을 제어할 수 있는 [CSS Module](https://github.com/css-modules/css-modules)이나 [CSS-in-JS](https://cssinjs.org/?v=v10.10.0) 라이브러를 사용하면 관리가 용이해집니다.

다만 bundle의 용량이 커진다는 trade-off를 고려해야 합니다.

## Shared component libraries

svc별 컨텐츠 내 스타일은 시각적으로 일관성을 가져야 하는데 이를 구현하기 가장 쉬운 방법은 공유와 재사용이 가능한 UI 컴포넌트 라이브러리를 도입하는겁니다.

제대로 만들어진 라이브러리는 UI 영역을 개발하는데 비용을 줄이면서 디자이너와 개발자들 사이에서 협력을 용이하게 만듭니다.

공통 컴포넌트는 개발하면서 중복되는 소스를 공통 API 단위로 확실히 만든 뒤에 외부 라이브러리로 옮기는 방식으로 추가하면 됩니다.

**여기서 컴포넌트 내부에는 비즈니스나 도메인 로직이 아닌 오직 UI가 동작하기 위한 최소한 로직만이 있어야 합니다!**

라이브러리의 품질과 일관성 관리는 팀 단위 또는 개인이 담당하되 모든 개발자들이 해당 프로젝트에 기여할 수 있어야 합니다.

## Cross-application communication

container 내부에 서로 다른 컨텐츠들 간의 통신은 Custom event, React의 props, URL query 등을 통해서 가능합니다.

페이지 내부 통신은 메시지 또는 event 기반으로 하고 의도치않은 coupling이 일어날 수 있는 상태공유 방식은 최대한 지양해야 합니다.

따라서 컨텐츠들을 페이지 내에서 결합하는 중에 어떤 종류의 coupling이 발생하는지와 해당 coupling을 어떻게 해결할 것인지에 대해서 항상 생각해야 합니다.

## Backend communication

mfe가 프론트 영역을 다룬다면 API 서버와의 통신을 구현하는 부분은 주로 [BFF(Backend For Frontend)](./bff.md) 패턴을 사용합니다.

![MSA BFF](./images/msa_bff.png)

BFF는 svc별로 필요한 데이터만을 내려주는 서버와 담당 DB가 존재하고 각 서버 내에는 독립적으로 구동되는 비즈니스 로직과 DB 접근로직이 있고 다수의 downstream 서비스들을 결합(하여 사용)합니다.

특정 svc를 관리하는 팀에서는 downstream 서비스들의 빌드와 배포를 기다릴 필요가 없습니다.

## Testing

코드의 품질을 확인하기 위한 자동화된 테스팅 프로세스는 svc별로 반드시 필요합니다.

비즈니스 로직과 렌더링 로직을 확인할 때는 Unit Testing으로, 페이지 내에서 특정 컨텐츠가 정해진 위치에 동적으로 노출되는지 여부 등 페이지 내에서 svc들 간의 통합 테스트는 [Cypress](https://www.cypress.io/)나 [Selenium](https://www.selenium.dev/)을 사용한 Functional/E2E Testing으로 구현합니다.

## Downsides

### Payload size

독립적인 빌드를 거쳐서 배포된 JS bundle들이 공통으로 사용한 dependency들로 인해서 클라이언트가 다운받을 데이터의 용량이 증가합니다.

공통으로 사용하는 dependency들을 외부 영역에 저장하여 해결할 수 있지만 모든 svc들이 동일한 버전의 패키지만 사용해야 하는 번거로움이 있습니다.

아니면 컨텐츠 단위가 아닌 단일 페이지 단위로 svc를 구성하여 한 페이지를 렌더링하는데 필요한 bundle만 다운받도록 하여 해결할 수도 있지만 페이지 navigation이 일어날 때마다 동일한 dependency들을 다운받는 것은 마찬가집니다.

### Environment differences

특정 svc 영역을 개발하는데 다른 svc들의 영향을 받아서는 안됩니다.

또한 개발환경과 production 환경에서 container가 모두 동기화된 상태를 유지할 수 있도록 개발할 때마다 환경별로 지속적인 통합과 배포(CI/CD)가 이루어져야 합니다.

### Operational and governance complexity

아무래도 단일 앱이 여러 개의 svc들로 구성되므로 관리할 repo들, tool들, CI/CD 파이프라인들, 도메인들이 많아질 수 밖에 없습니다.

mfe를 채택하기 전에 현재 조직이 기술적으로 숙달된 상태인지 확인하여 혼란을 방지할 필요가 있습니다.

## 참고자료

[Micro Frontends](https://martinfowler.com/articles/micro-frontends.html)