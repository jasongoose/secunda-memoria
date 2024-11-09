---
title: Multithreading in JavaScript with Web Workers
date: 2024-11-01 22:18:07
categories:
  - Articles
#tags:
---
JS는 single-thread 기반의 언어라서 한번에 하나의 task(context 연산)를 처리할 수 있습니다.

그래서 call stack에서 pop하여 받은 CPU-intensive 연산을 처리하다보면 뒤에 오는 task들이 대기해야 하는 blocking 현상이 발생합니다.

특히 브라우저에서 실행되는 웹앱에 blocking이 생기면 앱의 처리속도가 느려지면서 UX가 떨어지므로 이와 관련된 최적화에 신경써야 합니다.

브라우저 런타임 자체는 single-thread 기반이지만 Web Worker를 사용하면 mulitithread 연산을 구현하여 main thread blocking을 방지할 수 있습니다.

브라우저와 Node.js 같은 JS 런타임은 event loop에 의한 context switch를 지원하여 비동기 작업을 가능하게 합니다.

따라서 JS는 병렬성이 아닌 동시성 언어이자 비동기(non-blocking) 언어입니다!

Web Worker를 마치 **브라우저 런타임 인스턴스**라고도 이해할 수 있습니다.

보통 Web Woker가 필요한 CPU-intensive 연산들은 다음과 같습니다.

- 매우 긴 string의 encoding/decoding
- 복잡한 mathematical computation
- 외부 디바이스와 같은 background IO

## Parallelism

![Parallelism](/images/parallelism.jpeg)

2개 이상의 CPU core를 사용하여 다수의 task를 동시간 대에 처리하는데 여러 비트들을 동시에 조합하는 드럼 연주가들을 생각하면 됩니다.

## Concurrency

![Concurrency](/images/concurrency.gif)

1개의 CPU core로 다수의 task들을 돌아가면서 조금씩 빠르게 처리하는데 내부적으로 context switch가 일어납니다.

한 명의 드러머가 빠른 손놀림으로 연주하는 드러머를 생각하면 됩니다.

## 참고자료

[Multithreading in JavaScript with Web Workers](https://www.honeybadger.io/blog/javascript-web-workers-multithreading/)

[Using Web Workers](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers)
