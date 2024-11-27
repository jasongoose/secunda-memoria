---
title: 3-8장. 예외 처리하기
date: 2024-10-29 23:54:44
categories:
  - Books
  - Node.js 교과서(개정 3판)
#tags:
---
예외란 보통 처리하지 못한 에러를 말하는데, Node.js의 메인 스레드는 하나뿐이므로 에러로 인해 멈춘다는 것은 스레드를 갖고 있는 프로세스가 멈춘다는 뜻이고 전체 서버도 멈춘다는 것과 같습니다.

에러 로그가 기록되더라도 작업은 계속 진행할 수 있도록 해야하고 실제 배포용 코드에 문법 에러가 있어서는 안됩니다.

에러가 발생할 것 같다면 미리 try/catch 문으로 감싸면 됩니다.

```js
setInterval(() => {
  try {
    throw new Error("서버를 고장낸다...");
  } catch (err) {
    console.error(err);
  }
}, 1000);
```

Node.js 내장 모듈의 에러는 실행 중인 프로세스를 멈추지 않으므로 에러로그를 기록해두고 나중에 원인을 찾아서 수정하면 됩니다.

```js
const fs = require("fs");

setInterval(() => {
  fs.unlink("./index.js", (err) => {
    if (err) {
      console.error(err);
    }
  });
}, 1000);
```

에러를 throw하는 경우에는 반드시 try/catch 문으로 에러를 잡아야 합니다.

Node.js v16부터 Promise의 에러는 반드시 catch해야 하는데 catch하지 않으면 에러와 함께 프로세스가 종료됩니다.

```js
const fs = require("fs").promises;

setInterval(() => {
  fs.unlink("./index.js").catch(console.error);
});
```

정말 예측이 불가능한 에러를 처리하려면 `process`의 `uncaughtException` 이벤트를 사용하면 됩니다.

```js
process.on("uncaughtException", (err) => {
  // ...
});

setInterval(() => {
  throw new Error("서버를 고장낸다...");
}, 1000);
```

Node.js 공식 문서에서는 `uncaughtException` 이벤트를 최후의 수단으로 사용할 것을 명시하고 있는데, 해당 이벤트 발생 후 다음 동작이 제대로 동작하는지를 보장하지 않습니다.

`uncaughtException`은 단순히 에러 내용을 기록하는 정도로 사용하고, 에러를 기록한 후 `process.exit`으로 프로세스를 종료하는 것이 좋습니다.

따라서 에러가 발생했을 때 철저히 로깅하는 습관을 들이고 주기적으로 로그를 확인하면서 보완해야 합니다.
