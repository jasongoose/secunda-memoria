---
title: Browser Runtime
date: 2024-11-02 00:04:26
categories:
  - Studies
  - Browser
#tags:
---
![브라우저 런타임 구조](/images/js_runtime_environment.jpg)

## 구조

### JS 엔진

JS 코드를 기계어로 변환하는 모듈로 힙 메모리(memory heap)와 콜 스택(call stack)으로 구성됩니다.

콜 스택은 실행 중 생성된 실행 컨택스트(execution context)들이 LIFO(Last In First Out) 방식으로 대기하는 스택 공간이고, 힙 메모리는 동적할당을 위한 큰 메모리 공간으로 다양한 객체들이 저장됩니다.

힙 메모리에 위치한 각종 객체나 함수들을 참조하여 콜 스택의 top에 위치한 context를 동기적으로 처리하고, 반환된 context는 콜 스택으로부터 pop합니다.

**여기서 콜 스택이 하나만 존재한다는 점은 JS의 메인 스레드가 하나만 존재한다는 점과 일맥상통합니다.**

### Web API

JS 엔진이 이용할 수 있는 브라우저 API로, mulit-thread 환경을 모방하기 위한 [Web Workers API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API), 웹 페이지를 객체 단위로 접근하기 위한 [DOM API](../web_api/dom), 힙 메모리를 제외하고 브라우저에 데이터를 저장하기 위한 [Web Storage API](storage/web_storage) 등이 있습니다.

context 내의 AJAX 요청, timebased 이벤트, DOM 이벤트 등의 비동기 작업들은 Web Workers API에 의해서 처리됩니다.

### MicroTasks Queue

Web Workers API로 처리된 settled promise들이 대기하는 queue 공간입니다.

### Callback Queue

Web Workers API에 의해서 전달된 콜백함수 context들(settled promise를 제외한)이 대기하는 queue 공간입니다.

### Event Loop

콜 스택에서 context 실행간에 break가 있거나 스택이 빌 때마다 MicroTasks -> Callback 순으로 콜백함수 context를 dequeue하여 콜 스택에 push합니다.

MicroTasks Queue의 우선순위가 Callback Queue보다 높기 때문에 임의의 microtask에서 loop가 발생하면 blocking 현상이 일어날 수도 있습니다.

**메인 스레드가 담당하는 동기적 작업과 비동기적 작업의 처리시점을 정하기 때문에 JS의 동시성 모델의 핵심 요소라고 볼 수 있습니다.**

## 비동기 작업 예시

### setTimeout

setTimeout 메서드가 호출되어 콜 스택에서 pop되면 Web API에 의해서 timer가 생성되고 지정한 시간이 지났을 때 callback queue에 enqueue합니다.

그럼 여기서 두 가지 변수로 인해서 시간 지연이 발생합니다.

- callback queue에 enqueue하려는데 queue가 full한 경우
- callback queue로부터 dequeue하려는데 콜 스택이 full한 경우

따라서 setTimeout 메서드의 콜백함수를 실행하기까지 걸리는 시간은 인자로 지정한 시간 이상입니다.

### 렌더링

브라우저는 짧은 주기에 맞춰서 repaint를 수행하는데 해당 작업은 일반 비동기 콜백처럼 따로 render queue에 enqueue되고 콜 스택이 빌 때마다 event loop에 의해서 dequeue되어 처리됩니다.

즉, 콜 스택에 다른 동기작업들이나 비동기 콜백들로 어느 정도 채워져 있는 상태라면 blocking 현상에 의해 적절한 타이밍에 repaint가 수행되지 않을 수도 있습니다.

repaint가 block된다면 사용자 event에 의한 UI 업데이트가 제때 수행되지 않아서 버튼 클릭과 같은 단순한 motion이나 animation이 보여지는 중 끊김현상이 일어날 수도 있습니다.

scrolling과 같이 짧은 시간 안에 빈도수가 높은 이벤트를 비동기적으로 처리한다면 수많은 handler 콜백들이 callback queue에 쌓이고 콜 스택에 차례대로 처리되는 동안에 repaint는 계속 blocking될 수 있습니다.

이러한 문제를 해결하기 위해서 throttling, debounce과 같은 최적화 기법이 자주 활용됩니다.

## 참고자료

[Javascript 동작원리 (Single thread, Event loop, Asynchronous)](https://medium.com/@vdongbin/javascript-%EC%9E%91%EB%8F%99%EC%9B%90%EB%A6%AC-single-thread-event-loop-asynchronous-e47e07b24d1c)

[The event loop](https://developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop)

[어쨌든 이벤트 루프는 무엇입니까? | Philip Roberts | JSConf EU](https://www.youtube.com/watch?v=8aGhZQkoFbQ)

[Understanding the JavaScript Runtime Environment and DOM Nodes](https://vahid.blog/post/2021-03-21-understanding-the-javascript-runtime-environment-and-dom-nodes/#1-the-js-runtime-environment)
