---
title: Stream Message Frame
date: 2024-11-03 14:01:10
categories:
  - Studies
  - HTTP
  - HTTP2
#tags:
---
![Streams Message Frames](/images/streams_messages_frames.png)

## 단위

### Stream

생성된 TCP connection 상에서 동일한 방향을 가진 1개 이상의 메시지들을 표현하는 byte flow입니다.

stream과 stream을 합치면 또 다른 stream이 되는데, stream id는 결합된 stream 내부에서 영역을 나누기 위한 수단으로 이해할 수 있습니다.

### Message

HTTP/1.1 요청 또는 응답 메시지를 표현하는 논리적인 단위으로 1개 이상의 Frame들로 구성됩니다.

### Frame

HTTP/2 통신의 최소 단위로, header(HTTP header ❌)에 자신이 속한 stream의 id를 가집니다.

## 단위간의 관계

- 단일 TCP conncection에서는 다수의 stream들을 주고받을 수 있습니다.
- 각 stream은 고유한 id와 내부 메시지들간의 우선순위 정보를 가집니다.
- 각 메시지는 논리적으로 HTTP 메시지를 담고 있고 다수의 Frame들로 구성되어 있습니다.
- 각 Frame은 HTTP header, body 등을 인코딩한 단위로, 서로 다른 stream끼리 결합할 때 Frame 단위로 이루어집니다.

## 참고자료

[Introduction to HTTP/2](https://web.dev/performance-http2/)