---
title: API
date: 2024-11-03 23:12:42
categories:
  - Studies
  - Web Terms
#tags:
---
Application Programming Interface의 약자로 2가지 정의가 있습니다.

- 두 프로그램 사이에서 리소스를 주고받거나 특정 기능을 수행하기 위한 수단
- 앱 구성요소들을 만들거나 결합하기 위한 관련 정의들

시스템의 구현방식은 몰라도 interface에 맞춰서 요청을 구성하면 필요한 리소스나 기능을 사용할 수 있습니다.

![What is API](/images/what_is_api.png)

## 공개범위

### Private

오직 조직 내부에서만 사용할 수 있는 API

### Partner

특정 partner사들만 사용할 수 있는 API

### Public

제3자 조직이나 개인이 사용할 수 있도록 외부로 명세를 공개한 API

## Remote API

네트워크를 통해서 리소스를 주고 받을 수 있도록 설계된 API로, Web Standard들을 따릅니다.

Web Standard는 인터넷을 통해서 다양한 리소스들을 접근할 수 있는 정보 시스템인 WWW(World Wide Web)의 동작을 정의하기 위한 다양한 표준을 가리키는데 HTTP, HTML/XML, CSS, ECMAScript, DOM, MIME 등이 있습니다.

## Specification

Remote API가 퍼지면서 HTTP 기반으로 두 주체간의 리소스 전달방식을 구체화 및 표준화하기 위한 명세서들이 대거 등장했는데 대표적으로 SOAP와 REST가 있습니다.

### SOAP

Simple Object Access Protocol의 약자로 메시지 포맷은 XML을 가지면서 HTTP, SMTP 프로토콜을 따릅니다.

어느 프로그래밍 언어에서나 XML 형태로 인코딩/디코딩할 수 있기 때문에 다양한 환경에서 정보를 주고받기에 최적화된 특징이 있습니다.

하지만 내장된 요구사항(XML messaging, built-in security, transaction compliance)에 의해서 전송하는데 시간이 걸리고 packet의 크기가 크다는 단점이 있습니다.

### RESTful

REST 구조를 따르는 API로, SOAP처럼 구체적인 리소스 전달방식을 정의하는 것과 달리 6가지의 RESTful System Guideline을 제시한다는 특징이 있습니다.

다르게 말하자면 실제 API의 구조는 창작자 마음이지만 제약조건을 만족해야 RESTful API라고 부를 수 있습니다.

#### Client-Server architecture

API의 두 주체는 요청하는 클라이언트와 응답하는 서버로 구성되고 HTTP 프로토콜을 따른다.

#### Stateless

클라이언트 관련 정보(metadata, URI, caching, cookie, authorization 등)는 서버가 아닌 클라이언트 단에만 저장한다(→ server는 client를 기억하지 않는다).

#### Cacheability

특정 요청에 대한 응답은 클라이언트 cache에 저장할 수 있다.

#### Layered System

클라이언트와 서버간의 통신은 다수의 layer들(Load Balancer, API Gateway, Shared Caches 등)을 거쳐서 조정될 수 있다.

#### Code on demand(optional)

필요하다면 서버에서 클라이언트로 실행가능한 코드를 전송하여 클라이언트의 기능확장을 지원할 수 있다.

#### Uniform Interface(4 facets)

- 요청하는 리소스의 ID(URI)는 요청 메시지에 포함되고 리소스 자체는 형태와 무관하다(→ 리소스는 JSON, XML, Text 등 다양한 포맷으로 표현할 수 있다).
- 클라이언트는 리소스의 형태 정보(metadata)를 이용하여 리소스의 state를 수정하거나 일부를 삭제할 수 있다.
- 응답 메시지는 클라이언트 단에서 리소스를 어떻게 처리해야 하는지에 대한 정보를 포함한다(→ format별로 다른 paser가 필요하다).
- 최초 리소스를 접근한 이후, 클라이언트는 서버에서 제공한 hyperlink를 통해서 필요한 리소스들을 점진적으로 + 동적으로 얻을 수 있다.

6번째 제약조건으로부터 클라이언트와 서버 사이에서 주고받는 리소스란 **데이터로 표현되는 정보**임을 유추할 수 있습니다.
