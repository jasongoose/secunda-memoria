---
title: Cache Storage
date: 2024-11-02 00:08:44
categories:
  - Studies
  - Browser
  - Web API
#tags:
---
브라우저 상에서 `Cache` 객체들을 저장하는 interface로 `Cache` 객체별로 고유한 id를 가집니다.

`ServiceWorker` 또는 `window` 전역객체를 통해서 접근이 가능하고 하나의 origin은 하나의 캐시 스토리지를 가질 수 있습니다.

![Cache Storage](/images/cache_storage.png)

## Cache

[Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/fetch) 의 `Request`, `Response` 객체 pair를 반영구적으로 브라우저에 저장하기 위한 interface이자 메커니즘입니다.

하나의 origin은 여러 개의 cache를 가질 수 있는데, 개별 cache의 flush 주기, 업데이트 주기 등의 cache 관리는 script를 통해서 가능합니다.

단일 cache의 최대용량은 정해져 있기 때문에 브라우저는 디스크 공간을 관리하기 위해 임의의 cache를 제거하거나 심지어 origin이 사용하는 캐시 스토리지 자체를 삭제할 수도 있습니다.

## 참고자료

[Cache Storage](https://developer.mozilla.org/en-US/docs/Web/API/CacheStorage)