---
title: Binary Framing Layer
date: 2024-11-03 14:00:39
categories:
  - Studies
  - HTTP
  - HTTP2
#tags:
---
HTTP/2 는 socket API와 프로세스 사이에 Binary Framing Layer를 추가하여 HTTP/1.1 메시지의 header와 body를 각각 Frame 단위로 변환(인코딩)하고 결합합니다.

변환된 메시지는 Byte Stream 형식을 가집니다.

![Binary Framing Layer](/images/bfl.png)

BFL은 OSI 계층모델에서 [6계층](../../../../books/tcp-ip-guide/chapter-6-6)의 구현체로 볼 수 있습니다.

## HTTP/1.x의 한계

기존 HTTP/1.x 메시지는 실시간 반응이 필요한 현대 요구를 충족시키기에 다음과 같은 한계가 있습니다.

### body와 다르게 header는 압축되지 않는다.

header가 body에 비해 용량이 커서 불필요한 트래픽이 발생할 수 있고, 단일 connection 내의 메시지들 사이에서 헤더의 중복이 빈번히 발생합니다.

### 단일 TCP Connection에서 Multiplexing이 불가하다.

로딩 지연을 줄이는 동시성을 구현하려면 다수의 TCP Connection들이 필요합니다.

### 요청한 리소스들 사이의 우선순위를 부여할 수 없다.

TCP 세그먼트의 제어 비트값을 이용하여 특정 stream은 별다른 handling 없이 즉시 상위계층으로 전달하는 로직을 활용하지 않습니다.

## Frame이란?

HTTP/2는 HTTP/1.x 메시지의 header와 body를 각각 별도의 binary frame 단위로 변환한 메시지를 사용합니다.

Frame을 사용하면 아래와 같은 문제를 해결할 수 있습니다.

- stream들을 생성하면서 header와 body는 분리되기 때문에 개별 압축을 수행할 수 있다.
- 다수의 stream들을 결합할 수 있기 때문에 단일 TCP Connection에서 Multiplexing을 구현할 수 있다.

![HTTP2 Frame](/images/http2_frame.png)

## 참고자료

[Introduction to HTTP/2](https://web.dev/performance-http2/)