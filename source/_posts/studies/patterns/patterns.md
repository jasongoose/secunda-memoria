---
title: Patterns
date: 2024-11-03 18:34:59
categories:
  - Studies
  - Patterns
#tags:
---
코드를 작성하면서 쌓인 노하우들을 기반으로 일반화된 SW 개발패턴을 가리킵니다.

패턴을 사용하면 다음과 이점이 있습니다.

- 프로젝트의 구조와 동작원리를 팀원과 공유하기 쉬워진다.
- 상황에 따라 다른 디자인을 도입하기 쉬워진다.
- 효율적인(경제적 관점에서, 비용 대비 이익이 더 큰) 기능 고도화가 가능해지면서 장기적으로 SW 내부 품질을 높일 수 있다.

## MVC

가장 초창기에 나온 모델로, 사용자들에게 노출되는 페이지 영역과 화면에 표시할 데이터를 관리하는 영역을 분리하고자 했습니다.

- **Model** : 화면에 표시할 데이터
- **View** : HTML, CSS, JS로 만들어진 페이지
- **Controller** : 사용자 상호작용에 대응되는 데이터 업데이트 로직

![MVC](/images/mvc.png)

서비스 초창기 시절(PHP, JSP, Ruby On Rails 등)에는 MVC 패턴을 다음과 같이 구현했습니다.

| Model |     View      | Controller |
| :---: | :-----------: | :--------: |
|  DB   | HTML, CSS, JS |  SSR 영역  |

시간이 지나 [AJAX](https://developer.mozilla.org/en-US/docs/Web/Guide/AJAX)가 등장하면서 페이지 렌더링 이후에도 브라우저에서 서버로 필요할 때마다 데이터를 요청할 수 있게되자 구현방식에 변화가 생겼습니다.

|      Model       |   View    | Controller |
| :--------------: | :-------: | :--------: |
| REST API w/ AJAX | HTML, CSS |     JS     |

## MVVM

MVC 패턴의 Controller는 다음과 같은 반복적인 개발패턴을 가집니다.

1. 이벤트 핸들러 연결
2. API 요청
3. 데이터 수정
4. DOM 수정/삭제/추가

이러한 로직상의 중복을 극복하고자 서버에서 `.html` 파일을 생성할 때 활용하는 “템플릿 바인딩” 개념을 프론트 영역에 도입하게 됩니다.

템플릿 바인딩은 Model과 View의 기능을 통합하고 Controller의 기능들을(위 4가지) 프레임워크에 맡겨서 선언적으로 페이지를 생성할 수 있도록 만듭니다.

Controller는 여전히 JS지만 이제 템플릿(View)에 바인딩할 데이터(Model)만 다루게 되어 ViewModel이라는 새로운 이름을 가졌고 여기서 MVVM 패턴이 탄생했습니다.

MVVM의 구현체로는 React, Vue, Angular, Sevelte 등이 있습니다.

![MVVM](/images/mvvm.jpeg)

## Container-Presenter

MVVM 패턴의 등장 이후로, 하나의 페이지는 단일 문서가 아닌 다수의 재사용이 가능한 단위 즉, 컴포넌트들의 조합이라는 새로운 관점이 생겨났습니다.

컴포넌트는 내부적으로 MVVM 패턴을 따르고 문맥과 상관없이 재사용되려면 VM에 비즈니스 로직을 가지면 안됩니다.

이를 계기로 비즈니스 로직을 가지지 않고 특정 영역의 렌더링만 담당하는 Presenter와 비지니스 로직에 맞춰 Presenter들을 조합하여 최종 화면을 만드는 Container로 컴포넌트의 역할을 분리하는 Container-Presenter 패턴이 탄생합니다.

![Container-Presenter](/images/container_presenter.png)

### Props Drilling

Container-Presenter 패턴을 따르는 페이지에서 컴포넌트들은 계층구조를 가지고 Container는 props를 통해 Presenter로 데이터를 전달합니다.

Container에서 깊게 중첩된(deeply nested) Presenter로 데이터를 전달하려면 중간에 거치는 컴포넌트들도 동일한 props를 가져야 하는데, 이러한 현상을 Props Drilling이라고 합니다.


![Props Drilling](/images/props_drilling.png)

Props Drilling은 다음과 같은 문제점들을 가집니다.

- 중간에 위치한 컴포넌트가 내부 상태와 관련없는 props를 강제로 가지게 되어 재사용성이 떨어질 수 있다.
- props의 이름이나 타입이 변할 때마다 일일이 찾아서 수정해야하는 번거로움이 발생한다.

## Flux

Container-Presenter 패턴의 props drilling 문제를 해결하려는 목적으로 facebook(현 Meta)에서 제안했던 패턴입니다.

기존에는 중재자(Controller, ViewModel)를 중심으로 데이터가 양방향으로 흘렀지만(M <=> VM <=> V) Flux에서는 중재자 없이 user interaction → dispatch → store → view 라는 일관된 방향으로 앱의 동작을 제어할 수 있는 단방향성을 가집니다.

![Flux](/images/flux.png)

Flux의 특징들을 정리하면 다음과 같습니다.

- 컴포넌트 내부 Model과 Controller 영역(비즈니스 로직)을 컴포넌트 외부 store에서 관리한다.
- View는 다수 컴포넌트들의 조합이 아닌 단일 template을 가지는 넓은 범위로 이해한다.

Flux의 구현체로는 Redux, Vuex 등이 있습니다.

## Observer-Observable

Flux는 상태관리를 이용하여 일관된 수단으로 앱을 제어할 수 있다는 장점이 있지만 action, dispatch 등의 생소한 개념과 장황한 문법으로 인한 높은 진입장벽이 문제였습니다.

이에 몇몇 개발자들은 상태관리는 유지하되 View(Observer)에서 store(Observable)를 subscribe하여 store에서 일어나는 변화를 비동기적으로 감지할 수 있는 Observer-Observable 패턴을 생각하게 됩니다.

해당 패턴은 Flux의 기능을 가지면서 복잡한 개념없이 store에서 변화가 일어날 때마다 전달되는 notification을 기반으로 렌더링을 수행할 수 있다는 특징이 있습니다.

![Observer-Observable](/images/oo.png)

## MVI

[cycle.js](https://cycle.js.org/model-view-intent.html)라는 라이브러리에서 처음 제시한 패턴으로 View와 Model를 분리하지만 사용자의 이벤트별로 수행되는 action들을 정의한 Intent 영역을 Controller로 두는 특징이 있습니다.

![MVI](/images/mvi.png)

해당 패턴은 다음과 특징이 있습니다.

- Controller를 2개의 layer로 나눈다.
    - View의 UI를 통해서 전달된 event별 사용자의 의도를 파악하는 로직들
    - 의도별로 model을 조작하는 함수들
- Intent와 View 사이의 의존성이 없다.
    - 하나의 Intent 로직이 여러 개의 View에서도 사용할 수 있다.
    - 개별 Intent 로직 단위의 테스트들로 전체 View 테스트를 대체할 수 있다.
    - 컴포넌트 간의 통신(props)에 의존하지 않아서 일관된 상태를 유지할 수 있다.

## 참고자료

[소프트웨어 아키텍쳐를 얘기할 때는...](https://www.youtube.com/watch?v=4E1BHTvhB7Y)

[프론트엔드에서 MV\* 아키텍쳐란 무엇인가요?](https://velog.io/@teo/%ED%94%84%EB%A1%A0%ED%8A%B8%EC%97%94%EB%93%9C%EC%97%90%EC%84%9C-MV-%EC%95%84%ED%82%A4%ED%85%8D%EC%B3%90%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80%EC%9A%94)
