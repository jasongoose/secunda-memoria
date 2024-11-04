---
title: Cache
date: 2024-11-03 13:56:01
categories:
  - Studies
  - HTTP
  - HTTP1
#tags:
---

HTTP를 따르는 요청과 그에 대응하는 응답 데이터를 같이 따로 저장하여, 추후 동일한 요청을 처리할 때 재사용하기 위한 메커니즘으로 서버의 트래픽 분산을 목적으로 많이 사용됩니다.

![HTTP1 Cache](/images/http1_cache.png)

HTTP cache는 여러 클라이언트들 사이에서 공유되는지 여부에 따라서 2가지 타입으로 나뉩니다.

## Private Cache

특정 클라이언트 안에서만 사용되어 다른 클라이언트와 리소스(응답 데이터)가 공유되지 않는 캐시로, 대표적으로 Browser Cache가 있습니다.

다른 클라이언트에 리소스가 공개되지 않는 성격에 기반하여 보통 사용자의 개인정보를 저장할 때 주로 사용합니다.

HTTP 응답 메시지의 `Cache-Control` 헤더 값이 `private`이면 body에 실려온 데이터는 오직 Private cache에만 저장되도록 만들 수 있습니다.

```bash
Cache-Control: private
```

하지만 `Authorizaton` 헤더가 있는 경우에는 private cache를 사용할 수 없습니다.

## Shared Cache

클라이언트와 서버 사이에 위치하여 다른 클라이언트들이 공유할 수 있는 캐시를 가리킵니다.

### Managed

서비스 개발자들에 의해서 관리되는 캐시로, 원본 서버로의 트래픽을 줄여서 사용자들에게 컨텐츠를 효율적으로 전달하기 위한 목적으로 활용됩니다.

주변에서 볼 수 있는 예시들을 나열하면 다음과 같습니다.

- Reverse Proxy
- Servic Worker w/ Cache API
- CDN

HTTP 응답의 `Cache-Control` 헤더뿐만 아니라 직접 구현한 로직으로도 충분히 제어(생성, 삭제)가 가능하다는 특징이 있습니다.

만일 캐시를 사용하고 싶지 않다면 HTTP 응답에 다음과 같은 헤더를 전달하면 됩니다.

```bash
Cache-Control: no-store
```