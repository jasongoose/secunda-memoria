---
title: IndexedDB
date: 2024-11-02 00:09:03
categories:
  - Studies
  - Browser
  - Web API
#tags:
---
JSON 객체들로 구성된 noSQL(특히 객체지향(Object-Oriented)) DB를 관리하기 위해 브라우저에서 제공하는 API입니다.

indexedDB는 다음과 같은 기능을 제공합니다.

- 웹 스토리지에서 저장하기 어려운 대용량의 정형화된 데이터를 브라우저 내에 영구적으로 저장할 수 있다(로컬 디스크 사용).
- 네트워크 환경과는 상관없이 저장된 데이터에 접근하여 off-line 서비스를 구현할 수 있다.

저장된 객체는 index로 부여된 key로 검색할 수 있고 db에 대한 모든 연산은 transaction 내에서만 가능합니다.

하나의 origin에 대해서 단일 indexedDB를 가질 수 있고 [Same-origin Policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy)에 의해서 다른 도메인에서 사용하는 indexedDB로 접근이 불가능합니다.

indexedDB에서 transaction은 비동기적인 요청에 의해서 수행되고 요청했던 연산들의 성공여부는 완료되면서 발생한 DOM event로 알 수 있습니다.

indexedDB가 자동으로 제거되는 경우들을 나열하면 다음과 같습니다.

- 실제 브라우저 사용자가 indexedDB를 포함한 브라우저에 저장된 데이터(쿠키, cache 등등)를 삭제할 때
- 시크릿 모드로 생성된 창이나 탭을 닫을 때
- 디스크 용량에 거의 다다랐을 때

## DB

다수의 [Object Store](#Object-Store)들, [Index](#Index)들의 집합을 의미하고, 다음과 같은 속성을 가져야 합니다.

### 이름

- DB를 구분하기 위한 `string` 타입 id
- 단일 origin은 하나의 DB를 일정 기간동안 가질 수 있습니다.

### 현재 버전

- 버전을 나타내는 `number`(integer)
- 임의의 DB는 다수의 버전을 동시에 가동할 수 없습니다.
- 버전을 수정하려면 현재 버전보다 더 큰 버전의 DB를 open해야 합니다.

## Object Store

key-value pair 형태로 저장된 record들이 영구적으로 저장된 공간으로, DB 내에서 개별 id를 가집니다.

record들은 key 값을 기준으로 오름차순 정렬됩니다.

### Key

단일 Object Store 내에 저장된 record들을 구분, 정렬, 검색하기 위한 id로, Object Store 내에서 key값들 간에 중복이 없어야 합니다.

key로 사용할 수 있는 데이터 타입은 다음과 같습니다.

- `String`
- `Date`
- `Number`
- `Blob`
- `Array`

record를 저장할 때 사용할 key 값은 수동으로 지정하거나 아래와 같은 방법을 사용하여 자동으로 만들 수 있습니다.

#### key generator

단일 Object Store 내에서 순서에 맞게 key를 생성해주는 mechanism으로, 선택적으로 사용할 수 있습니다.

key genenerator는 Object Store들 사이에서 공유할 수 없고 생성된 key는 record의 value와는 별도로 저장되기 때문에 “out-of-line key”라고 합니다.

#### key path

브라우저에게 record의 value로부터 key를 extract하기 위한 경로를 알려주는 mechanism입니다.

record 내에서 value와 별도로 저장되지 않고 value의 일부로서 저장되는 key를 “in-line key”라고 합니다.

처음 record를 저장할 때 key generator에 의해서 key가 생성되면 해당 key는 Object Store에 등록한 key path에 맞춰서 record의 value의 일부로 저장됩니다.

key path는 다음과 같은 데이터 타입을 가질 수 있습니다.

- 빈 문자열
- 단일 [JS identifier](https://developer.mozilla.org/en-US/docs/Glossary/Identifier)
- period(.)으로 구분된 다수의 identifier들 또는 이 자체를 요소로 가지는 배열

### Value

record의 key에 대응되는 구체적인 값으로 JS에서 구현되는 모든 데이터 타입(primitive + reference + Blob, Files 등)을 가질 수 있습니다.

## Index

전혀 다른 Object Store에 저장된 record를 검색하기 위한 목적으로 사용되는 특수 Object Store로, 검색 대상이 되는 Object Store를 “Referenced Object Store” 라고 합니다.

임의의 Object Store `X`가 있다면 동일한 `X`를 검색하기 위한 index `X_i`가 있는 것으로 이해할 수 있습니다.

index에 저장된 임의의 record는 Referenced Object Store에서 검색할 특정 record의 key값을 value로 가집니다.

index의 record는 Referenced Object Store에 저장된 record 하나만 참조할 수 있고, 다수의 index들이 동일한 record를 참조할 수도 있습니다.

Referenced Object Store에 있는 record에 대해서 삽입, 수정, 삭제 transaction이 발생할 때마다 그에 맞춰서 대응되는 index도 자동으로 업데이트됩니다.

물론 특정 record를 검색할거면 그냥 저장된 Object Store를 open해서 key값으로 직접 검색할 수도 있습니다.

## DB Connection

DB를 처음 open할 때 수행하는 연산으로, 단일 DB는 다수의 connection들을 동시에 가질 수 있습니다.

## Transaction

DB Connection 이후에 특정 DB에 대해서 수행되는 atomic 연산들의 집합을 가리킵니다.

transaction 생성 초기에 고유의 작업범위인 scope가 정의되는데, scope를 통해 활성화된 transaction의 영향을 받는 Object Store들과 그렇지 않은 Object Store들로 나뉩니다.

단일 DB Connection 상에서는 다수의 transaction들이 발생할 수 있는데 서로 scope들이 겹치지 않는 경우에는 동시간대에 공존할 수 있습니다.

만일 scope가 겹친다면 queue에 쌓여서 순차적으로 수행됩니다.

transaction의 수명은 앱의 성능을 위해서 가급적이면 짧아야 하기 때문에 브라우저는 DB 리소스를 오랫동안 차지하는 transaction를 중지하여 roll-back을 수행합니다.

Object Store에 수행할 수 있는 transaction들은 다음과 같이 정리할 수 있습니다.

- `readwrite`
- `readonly`
- `versionchange`

## Durable

과거 Firefox 40 이전 버전에서 DB에 `readwrite` transaction이 일어난 뒤, DB에 있던 데이터가 디스크로 완전히 flush되었을 때 `complete` 이벤트가 발생했던 성질입니다.

이후 대부분의 브라우저에서는 OS가 flush notice를 받고 나서 디스크로 flush되기 직전에 `complete` 이벤트를 발생하도록 수정된 상태입니다.

전보다 `complete` 이벤트가 더 빠르게 발생하여 앱의 성능을 올릴 수는 있지만 아주 희박한 확률로 OS crash가 발생하거나 시스템 공급전력이 부족하여 중간에 디스크 flush가 중단될 수도 있습니다.
