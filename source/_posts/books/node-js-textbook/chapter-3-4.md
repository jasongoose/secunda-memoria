---
title: 3-4장. 노드 내장 객체 알아보기
date: 2024-10-29 23:54:24
categories:
  - Books
  - Node.js 교과서(개정 3판)
  - Chapter 3
#tags:
---

## global

전역객체이므로 모든 파일에서 접근할 수 있습니다.

`require` 함수도 `global.require`에서 `global`이 생략된 것이고 `console` 객체도 원래는 `global.console`입니다.

Node.js에서 `window` 객체는 전역객체가 아니라서 [DOM(Document Object Model)이나 BOM(Browser Object Model)을 사용할 수 없습니다.](https://geniee.tistory.com/33)

런타임 환경에 따라 다른 전역객체를 참조하려면 `globalThis` 키워드를 적용하면 됩니다.

## console

`console`도 Node.js에서는 `global` 객체 안에 들어 있고 관련 메서드들을 일부 살펴보면 다음과 같습니다.

### console.time

같은 레이블을 가진 time과 timeEnd 사이의 시간을 측정합니다.

```js
console.time("시간측정");
// ...
console.timeEnd("시간측정"); // 여기서 시간이 출력됩니다.
```

### console.log

평범한 로그를 콘솔에 출력합니다.

### console.error

에러를 콘솔에 표시합니다.

### console.table

배열의 요소로 객체 리터럴을 넣으면, 객체의 속성들이 테이블 형식으로 표시됩니다.

```js
console.table([
  {
    name: "jason",
    birth: 1994,
  },
  {
    name: "goose",
    birth: 1988,
  },
]);
```

### console.dir

객체를 콘솔에 표시하기 위한 메서드입니다.

```js
const obj = {
  outside: {
    inside: {
      key: "value",
    },
  },
};

console.dir(obj, {
  colors: true,
  depth: 4,
});
```

### console.trace(레이블)

에러가 어디서 발생했는지 추적할 때 사용할 수 있습니다.

```js
console.trace("error label");
```

## 타이머

타이머의 기능을 제공하는 함수인 `setTimeout`, `setInterval`, `setImmediate`는 전역객체 안에 정의된 메서드입니다.

위 타이머 함수들은 고유한 아이디를 반환하고 개별 타이머를 취소하는데 이 아이디를 사용합니다.

### setTimeout

주어진 밀리초 이후에 콜백함수를 실행하고 타이머를 취소할 때는 `clearTimeout` 메서드를 사용합니다.

### setInterval

주어진 밀리초마다 콜백함수를 반복해서 실행하고 타이머를 취소할 때는 `clearInterval` 메서드를 사용합니다.

### setImmediate

콜백함수를 즉시 실행하고 타이머를 취소할 때는 `clearImmediate` 메서드를 사용합니다.

타이머는 콜백 기반 API이지만 Promise 방식으로도 사용할 수 있습니다.

```js
import { setTimeout, setInterval } from "timers/promises";

await setTimeout(3_000);

for await (const startTime of setInterval(1_000, Date.now())) {
  console.log("1초마다 실행", new Date(startTime));
}
```

## process

현재 실행되고 있는 Node.js 프로세스에 대한 정보를 담고 있는 객체입니다.

```js
process.version; // 설치된 Node.js 버전
process.arch; // 프로세서 아키텍처 정보
process.platform; // OS 플랫폼 정보
process.pid; // 현재 프로세스의 아이디
process.uptime(); // 프로세스가 시작된 후 흐른 시간
process.execPath; // Node.js의 설치경로
process.cwd(); // 현재 프로세스가 실행되는 위치
process.cpuUsage(); // 현재 cpu 사용량
```

### process.env

시스템의 환경변수들을 담고 있는 객체로, 서비스의 중요한 키를 저장하는 공간으로도 사용됩니다.

한번에 모든 운영체제에 동일하게 넣을 수 있는 방법으로 [dotenv](https://www.npmjs.com/package/dotenv)가 있습니다.

### process.nextTick

이벤트 루프는 다른 콜백함수보다 `process.nextTick`의 콜백함수를 우선으로 처리합니다.

`setImmediate`나 `setTimeout`보다 먼저 실행되는데, 이는 settled promise와 동일한 우선순위로 인식되어 마이크로태스크 큐에서 처리되기 때문입니다.

### process.exit

실행 중인 Node.js 프로세스를 종료합니다.

`process.exit` 메서드는 인수로 코드 번호를 줄 수 있는데 인수를 주지 않거나 0을 주면 정상종료, 1을 주면 비정상 종료를 의미합니다.

```js
let i = 1;

setInterval(() => {
  if (i === 5) {
    console.log("종료");
    process.exit();
  }
  console.log(i++);
}, 1_000);
```
