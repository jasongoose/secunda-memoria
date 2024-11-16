---
title: Introduction to HTTP/2
date: 2024-11-01 22:15:38
categories:
  - Articles
#tags:
---
## Binary Framing Layer

![Binary Framing Layer](/images/binary_framing_layer.png)

HTTP/2는 socket API와 프로세스 사이에 Binary Framing Layer를 추가하여 HTTP/1.1 메시지의 header와 body를 각각 frame 단위로 변환(인코딩)하고 결합하는데 여기서 변환된 메시지는 byte stream 형식을 가집니다.

당연히 클라이언트와 서버 단에서 해당 layer를 가져야 통신이 가능합니다.

## streams, messages and frames

![Frame](/images/frame.png)

3가지 단위간의 관계를 정리하면 다음과 같습니다.

- 단일 TCP conncection에서는 다수의 stream들을 주고받을 수 있다.
- 각 stream은 고유한 id와 내부 메시지들간의 우선순위 정보를 가진다.
- 각 메시지는 논리적으로 HTTP 메시지를 담고 있고 다수의 frame들로 구성된다.
- 각 frame은 HTTP header, body 등을 인코딩한 단위로, 서로 다른 stream끼리 결합할 때 frame 단위로 이루어진다.

### stream

생성된 TCP Connection 사이에서 여러 메시지를 지닌 양방향성 byte flow입니다.

> 양방향성 - 임의의 시점에 가능한 방향이 2가지란 의미

### message

논리적인(겉은 이상하지만 멀쩡한 내용을 가진) HTTP/1.1 request 또는 response를 담고있는 일련의 frame들을 가리킵니다.

### frame

HTTP/2 통신에서 최소 단위로, header(HTTP header x)에 자신이 속한 stream의 id를 가집니다.

기존 HTTP/1.x 메시지는 실시간 반응을 충족시키기에 다음과 같은 한계가 있었습니다.

#### 1. body와 다르게 header는 압축되지 않습니다.

header가 body에 비해 용량이 큰 overhead로 불필요한 트래픽이 발생할 수 있고, 단일 connection 내의 타입별 메시지들 사이에서 헤더의 중복이 빈번히 발생합니다.

#### 2. 단일 TCP Connection에서 Multiplexing이 불가합니다.

로딩 지연을 줄이는 동시성을 구현하려면 *다수의 TCP Connection들*이 필요합니다.

#### 3. 요청한 리소스들 사이의 우선순위를 부여할 수 없습니다.

TCP 세그먼트의 제어 비트값을 이용하여 특정 stream은 별다른 handling 없이 즉시 상위계층으로 전달하는 로직이 있는데 이걸 활용하지 않습니다.

그래서 HTTP/2 에서 위와 같은 문제를 해결하기 위해 HTTP/1.x 메시지의 header와 body를 각각 별도의 binary frame 단위로 변환한 메시지를 사용합니다.

![Message](/images/message.png)

그래서 위 문제들을 아래와 같이 해결할 수 있습니다.

1. stream들을 생성하면서 header와 body는 분리되기 때문에 별도로 압축이 가능합니다.
2. 다수의 stream들을 결합할 수 있기 때문에 단일 TCP Connection에서 Multiplexing을 구현할 수 있습니다.

## Request and response multiplexing

![Multiplexing](/images/multiplexing.png)

HTTP/2에서는 stream간의 frame 단위 결합과 재조립으로 Multiplexing을 구현합니다.

전송할 HTTP 메시지들을 _single Persistent TCP Connection_ 내에서 결합된 stream으로 주고받는 로직은 다음과 같은 장점을 가집니다.

- 여러 개의 request/response를 병렬적으로 전송해도 어느 하나도 block되지 않아서 HOL blocking 해결한다.
- 불필요한 네트워크 지연이 없고 가용 대역폭를 최대한 활용할 수 있다.

또한 Persistent TCP Connection을 전제로 하기 때문에 여러 개의 HTTP request, response stream들을 주고받을 수 있음을 알 수 있습니다.

## Stream Prioritization

![Stream Prioritization](/images/stream_prioritization.png)

HTTP/2에서는 리소스의 우선순위를 반영하여 stream들 사이의 우선순위를 결정하는데 결정요인은 2가지가 있습니다.

- dependency(1~256): request stream 처리 순서 또는 대응된 응답 stream의 전송순서
- weight: stream을 처리하기 위해 투입할 리소스 양의 비율

클라이언트는 stream별 dependency와 weight를 조합하여 만든 우선순위 트리를 서버와 공유하고 서버는 "최대한" 리소스 우선순위에 맞춰서 처리하고 응답을 전송합니다.

우선순위가 높은 stream을 처리하느라 낮은 우선순위 stream을 처리하는데 지연이 발생할 수 있기 때문에 “최대한”을 붙였습니다.

또한 우선순위 트리는 어느 때에서나 클라이언트 상에서 사용자 상호작용이나 다른 signal에 의한 업데이트가 가능합니다.

## One connection per origin

HTTP/2 stream은 frame 단위로 결합 및 재조립이 가능하기 때문에 단일 Persistent TCP Connection 상에서 Multiplexing을 구현할 수 있습니다.

이는 여러 개의 TCP Connection 상에서 요청들을 처리할 필요가 없어지므로 client, server, 중간 proxy들의 연산량이 감소하고 기존 HTTP 메시지를 압축하여 네트워크 대역폭을 절약할 수 있는 효과가 있습니다.

여기서 TCP Connection은 origin 당 하나씩 생성됩니다.

## 출처

[Introduction to HTTP/2](https://web.dev/articles/performance-http2?hl=ko)
