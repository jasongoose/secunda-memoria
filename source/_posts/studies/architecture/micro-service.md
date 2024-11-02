---
title: Micro Service
date: 2024-11-02 00:03:04
categories:
  - Studies
  - Architecture
#tags:
---
단일 앱을 작은 서비스들의 집합으로 구성하는 기법으로, 개별 서비스는 독립적인 배포 파이프라인을 가지고 서비스들 간의 통신은 주로 HTTP API로 이루어집니다.

![MSA](/images/msa.png)

## Monolith

Microservice(이하 msa)가 보편화되기 전에는 일반적으로 앱을 구성하는 UI 영역, 비지니스 로직 영역, DB 접근 영역들이 하나의 프로젝트로서 존재했는데, 이를 msa와 구분하기 위한 용어로 Monolith(이하 monolith) 기법이라고 합니다.

전체 앱을 하나의 장소에서만 관리하기에 배포 프로세스나 프로젝트 구조가 간단하다는 장점이 있지만 아래와 같은 단점이 있습니다.

- 특정 모듈의 사소한 변화로 인해서 전체 프로젝트를 재빌드해야 하는 번거로움이 있다.
- 전체 앱 단위의 확장만 가능하여 리소스 낭비가 심하다.

## Componentization via Services

Component(이라 component)란 독립적으로 대체되면서 업데이트할 수 있는 SW 구성단위를 의미하는데, 크게 Library와 Service로 나뉩니다.

Library(이하 lib)는 임의의 프로그램에 연결되어 in-memory 함수 내에서 호출하는 방식으로 사용되고, Service(이하 svc)는 프로세스 외부에 위치하여 웹 API나 원격 호출로 사용할 수 있습니다.

monolith 환경에서 component는 lib이고 msa 환경에서 component는 svc입니다.

svc를 component로 가지면 앱 전체 재빌드 및 재배포가 불필요해지고 component API를 원격 호출 메커니즘에 기반하여 더 구체적으로 구성할 수 있는 장점이 있습니다.

다만, 네트워크 통신과 인프라 유지를 위한 추가비용이 발생하고 통합적인 관리가 용이하도록 개별 svc에게 작업을 어떻게 할당할지에 대한 고민이 필요합니다.

## Organized around Business Capabilites

조직의 구성이 곧 시스템의 구성으로 이어진다는 법칙을 Conway's Law라고 합니다.

> Any organization that designs a system (defined broadly) will produce a design whose structure is a copy of the organization's communication structure.
>
> _Melvin Conway, 1968_

msa 환경에서는 비지니스 역량을 기반으로 목적조직을 구성할 수 있어서 단일 팀에서 단일 svc에 대한 기획, 개발 등의 다양한 역할들을 수행할 수 있습니다.

## Products not Projects

개발팀은 특정 svc의 개발부터 배포까지 모두 책임지기에 사용자와의 피드백을 효율적으로 받고 반영할 수 있습니다.

## Smart endpoints and dumb pipes

msa 기반 앱은 RESTful API를 사용하여 component들을 분리하거나 결합하기가 최대한 용이하도록 구성하는 것을 목표로 합니다.

## Decentralized Governance & Data Management

앱을 구성하는 svc는 독자적인 기술 스택과 DB를 가지기 때문에 svc별로 발생하는 다양한 이슈들을 해결하기 위한 적절한 도구들을 자율적으로 선택할 수 있습니다.

## Design for failure

하나가 아닌 다수 svc들로 구성된 msa는 임의의 svc가 다운되거나 연결이 끊기는 등의 이슈들을 빠르게 대처할 수 있도록 실시간 모니터링이 필요합니다.

## Evolutionary Design

다른 svc에 영향을 주지 않으면서 개발할 수 있으므로 장기적으로 특정 svc나 전체 앱의 디자인을 개선할 수 있습니다.

## 참고자료

[Micro Services](https://martinfowler.com/articles/microservices.html)