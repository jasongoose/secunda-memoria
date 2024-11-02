---
title: Backend For Frontend
date: 2024-11-02 00:02:35
categories:
  - Studies
  - Architecture
#tags:
---
## Introduction

페이지 UI를 구현하는 영역이 클라이언트에서 서버 단으로 이동하면서 우리는 기존 데스크톱(이하 pc) 웹앱이 사용한 백엔드 서비스(이하 api)를 모바일(이하 mo) 환경에 어떻게 적용할지에 대한 고민을 하게 됩니다.

## The General-Purpose API Backend

첫번째로 생각할 수 있는 것은 기존 pc에 최적화된 단일 api가 mo도 지원할 수 있도록 기능을 점진적으로 추가하는 방식이지만 문제점이 있습니다.

![General Purpose API Teams](/images/general_purpose_api_teams.jpg)

우선 mo는 pc에 비해서 화면이 작아서 한번에 보여줄 수 있는 정보의 양이 적고 여러 서버와의 연결을 생성하면 베터리가 금방 소진될 수 있다는 등의 물리적인 차이점을 고려하지 않습니다.

mo 기기와 pc 기기를 사용하는 방식 자체에도 차이점이 있어서 mo 앱을 만들면서 pc와는 별개의 로직과 기능을 추가적으로 구현해야 하는데, 이는 새로운 feature를 반영하는 과정 중 bottleneck이 될 수도 있습니다.

그리고 특정 비지니스 도메인을 가지지 않는 단일 api를 운영하다 보면 mo, pc를 포함한 다양한 클라이언트 담당팀들과의 조율과 다수 downstream 서비스들 간의 동기화 등 많은 책임들이 api 담당팀에게 부여되어 작업량이 편중될 수도 있습니다.

임의의 서비스가 Upstream, Downstream인지 여부의 기준은 의존성 또는 요청 메시지의 방향에 따라 달라집니다.

- [의존성의 방향](https://reflectoring.io/upstream-downstream/) : 클라이언트는 서버로부터 내려온 응답에 의존하므로 클라이언트 쪽이 Downstream, 서버 쪽이 Upstream

- [요청 메시지의 방향](https://www.rfc-editor.org/rfc/rfc7230#section-2.3) : 요청 메시지는 클라이언트 서버 방향으로 전달되므로 클라이언트 쪽이 Upstream, 서버 쪽이 Downstream

## Introducing The Backend For Frontend

BFF는 위 문제를 해결하기 위해 기존의 단일 api를 클라이언트별로 나눕니다.

그래서 UI에서 필요로 하는 api만을 제공하는 edge 서버로서 클라이언트 담당팀이 자체적으로 관리하도록 합니다.

![BFF Overview](/images/bff_overview.jpg)

## How many BFFs?

BFF를 운영하는 방식은 조직마다 다르겠지만 동일한 UI를 서로 다른 플랫폼에서 제공하는 경우 2가지로 나눌 수 있습니다.

하지만 일반적으로 BFF 운영방식은 [Conway's Law](./microservice.md#organized-around-business-capabilites)에 의해 조직구조의 영향을 받을 수 밖에 없습니다.

### client 타입 기준

![One BFF Per Mobile](/images/one_bff_per_mobile.jpg)

### UI 기준(기기 기준)

![Generic Mobile BFF](/images/generic_mobile_bff.jpg)

## And Multiple Downstream Services(Microservices!)

BFF는 많은 수의 downstream 서비스들을 통합하여 사용하기에 클라이언트에서 BFF로 전송한 단일 요청이 다수 downstream 서비스들로의 요청으로 이어집니다.

![Sequence](/images/sequence.jpg)

효율성을 위해서라면 위와 같이 다수의 요청들을 가능한 병렬적으로 처리하는 것이 맞겠지만 복잡한 시나리오에서 요청의 개수가 많아지면 제어가 힘들어질 수 있으니 [RxJava](https://github.com/ReactiveX/RxJava) 등을 사용한 [반응형 프로그래밍](https://en.wikipedia.org/wiki/Reactive_programming)의 도입을 고려해야 합니다.

downstream 서비스들 일부가 연결이 끊기거나 다운된 상황이라면 BFF 단에서도 예외처리가 필요하지만 클라이언트에서도 부분적으로 완성된 응답에 대해서도 제대로 렌더링할 수 있어야 합니다.

## Reuse and BFFs

BFF들 사이에서 downstream 서비스들을 통합하는 로직이 중복될 수도 있습니다.

![BFF Duplication](/images/bff_duplication.jpg)

일부 BFF 서버들을 하나로 합친다면 [단일 api의 문제점](#The-General-Purpose-API-Backend)을 고스란히 가져오게 됩니다.

공통 로직을 라이브러리로 추출할 수도 있지만 이마저도 다수 클라이언트들 간의 coupling 문제를 가집니다.

여기서 일부 downstream 서비스에서 중복되는 통합로직을 구현하도록 재설계하여 해결할 수 있습니다.

![Removing Duplication](/images/removing_duplication.jpg)

## BFFs for Desktop Web and Beyond

보통 pc 웹앱을 운영하는 물리적인 환경은 mo에 비해서 더 많은 downstream 서비스들을 관리할 수 있고 더 나은 연결성을 보장하기 때문에 BFF 자체가 불필요할 수도 있습니다.

페이지 컨텐츠 생성을 대부분 서버에 맡기는 환경이라면 BFF 앞에 reverse proxy를 두어 caching을 적용할 수 있습니다.

## And Autonomy

BFF를 도입하면 UI 영역과 downstream 서비스 영역을 분리하여 독립적으로 개발과 배포가 가능해지고 클라이언트에서 재사용할 기능을 서버 단에서 api 단위로 관리하여 개발 편의성을 높일 수 있습니다.

## General Perimeter Concerns

인증/인가 또는 요청 로깅과 같이 어느 클라이언트 단에서나 필요로 하는 공통 로직(Perimeter Service)은 BFF가 아닌 BFF 앞단의 layer 형식으로 위치시키거나 라이브러리로 따로 분리하는 것이 권장됩니다.

![Perimeter Layers](/images/perimeter_layer.jpg)

## When To Use

BFF는 모바일이나 제3의 환경에 특화된 기능 또는 UI를 추가하거나 UI 개발과 downstream 서비스 개발 영역을 분리할 때 도입할 수 있습니다.

## 참고자료

[Pattern: Backends For Frontends](https://samnewman.io/patterns/architectural/bff/)