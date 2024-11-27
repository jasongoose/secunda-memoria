---
title: 3-7장. 이벤트 이해하기
date: 2024-10-29 23:54:39
categories:
  - Books
  - Node.js 교과서(개정 3판)
#tags:
---
이벤트를 만들고 호출하고 삭제할 때는 `events` 모듈을 사용할 수 있습니다.

## events 메서드 알아보기

### events.on

이벤트 이름과 이벤트 발생 시의 콜백을 연결합니다.

이렇게 연결하는 동작을 "event listening"이라고 하고 이벤트 하나에 이벤트 여러 개를 달아줄 수 있습니다.

### events.addListener

`on`과 동일합니다.

### events.emit

이벤트를 호출합니다.

이벤트 이름을 인수로 넣으면 미리 등록한 이벤트 콜백이 실행됩니다.

### events.once

한번만 실행되는 이벤트와 이벤트 발생 시의 콜백을 연결합니다.

### events.removeAllListeners

이벤트에 연결된 모든 event listener를 제거합니다.

### events.removeListener

이벤트에 연결된 리스너를 하나씩 제거합니다.

### events.off

Node.js v10부터 추가된 메서드로, `removeListener`와 기능이 같습니다.

### events.listenerCount

특정 이벤트에 연결된 listener들을 확인합니다.

```js
const EventEmitter = require("events");

const myEvent = new EventEmitter();

myEvent.addListener("event1", () => {
  console.log("event1");
});

myEvent.on("event2", () => {
  console.log("event2");
});

myEvent.on("event2", () => {
  console.log("event2 추가");
});

myEvent.once("event3", () => {
  console.log("event3");
});

myEvent.emit("event1"); // 이벤트 호출
myEvent.emit("event2"); // 이벤트 호출

myEvent.emit("event3");
myEvent.emit("event3"); // 실행 x

myEvent.on("event4", () => {
  console.log("event4");
});
myEvent.removeAllListeners("event4");
myEvent.emit("event4"); // 실행 x

const listener = () => {
  console.log("이벤트 5");
};

myEvent.on("event5", listener);
myEvent.removeListener("event5", listener);
myEvent.emit("event5"); // 실행 x
```
