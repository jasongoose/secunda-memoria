---
title: 3-6장. 파일 시스템 접근하기
date: 2024-10-29 23:54:35
categories:
  - Books
  - Node.js 교과서(개정 3판)
#tags:
---
`fs` 모듈은 파일 시스템에 접근하는 모듈로 파일 또는 폴더의 생성, 삭제, 읽기, 쓰기 등을 지원합니다.

파일을 읽을 때는 `readFile` 메서드를 사용하면 됩니다.

첫번째 인자인 파일의 경로가 현재 파일 기준이 아니라 `node` 명령어를 실행하는 콘솔 기준이라는 점에 유의해야 합니다.

결과물인 `data`는 버퍼라는 형식으로 제공되는데 사람이 읽을 수 있는 형식이 아니므로 `toString` 메서드를 사용하여 문자열로 변환했습니다.

```js
const fs = require("fs");

fs.readFile("./readme.txt", (err, data) => {
  if (err) {
    throw err;
  }
  console.log(data.toString());
});
```

파일을 쓸 때는 `writeFile` 메서드를 사용하면 됩니다.

```js
const fs = require("fs");

fs.writeFile("./writeme.txt", "글이 입력됩니다", (err) => {
  if (err) {
    throw err;
  }
  fs.readFile("./writeme.txt", (err, data) => {
    if (err) {
      throw err;
    }
    console.log(data.toString());
  });
});
```

`fs` 모듈 자체는 콜백 형식의 모듈이므로 실무에서는 Promise 기반의 방법을 사용합니다.

```js
const fs = require("fs").promises;

fs.readFile("./readme.txt")
  .then((data) => {
    console.log(data.toString());
  })
  .catch((err) => {
    console.error(err);
  });
```

## 동기와 비동기 메서드

Node.js는 대부분의 메서드를 비동기 방식으로 처리하는데, 특히 `fs` 모듈이 그러한 메서드를 많이 갖고 있습니다.

`readFile`를 호출하면 백그라운드(?)에 해당 파일을 읽으라고 요청하고 다음 작업으로 넘어갑니다.

나중에 읽기가 완료되면 백그라운드가 다시 메인 스레드에 알리고 메인 스레드는 그제서야 등록된 콜백 함수를 실행합니다.

이 방식은 수백 개의 I/O 요청이 들어와도 메인 스레드는 백그라운드에 요청 처리를 위임합니다.

### 동기와 비동기, 블로킹과 논블로킹

- 동기 / 비동기: 백그라운드 작업 완료 확인 여부
- 블로킹 / 논블로킹: 함수가 바로 return되는지 여부

![동기, 비동기, 블로킹, 논블로킹의 관계](/images/sync_async_blocking_non-blocking.jpeg)

파일을 동기적으로 읽거나 쓰려면 "-Sync" postfix가 붙은 메서드를 사용하면 되는데, 이 경우 콜백 함수를 넣는 대신 직접 return 값을 받아옵니다.

```js
const fs = require("fs");

let data = fs.readFileSync("./readme.txt");

fs.writeFileSync("./writeme.txt", "내용");
```

위와 같이 IO 작업을 동기적으로 처리하면 백그라운드가 작업하는 동안 메인 스레드는 아무것도 못하고 대기하고 있어야 하니 비효율적입니다.

비동기 `fs` 메서드를 사용하면 백그라운드가 동시에 작업할 수도 있고 메인 스레드는 다음 작업을 처리할 수 있습니다.

따라서 동기 메서드는 프로그램을 처음 실행할 때 초기화 용도로만 사용하는 것이 권장되고 대부분의 경우에 비동기 메서드가 훨씬 더 중요합니다.

## 버퍼와 스트림 이해하기

파일을 읽거나 쓰는 방식에는 버퍼(buffer)를 이용하거나 스트림(stream)을 이용하는 방식 2가지가 있습니다.

비유하자면 버퍼링은 영상을 재생할 수 있을 때까지 데이터를 모으는 동작이고, 스트리밍은 방송인의 컴퓨터에서 시청자의 컴퓨터로 영상 데이터를 조금씩 전송하는 동작입니다.

### 버퍼

![Buffer](/images/buffer.png)

`readFile` 메서드를 사용하면 파일을 읽을 때 메모리에 파일 크기만큼 공간을 마련해두며 파일 데이터를 거기에 저장한 뒤 사용자가 조작할 수 있도록 합니다.

이 때 메모리에 저장된 파일 데이터가 "버퍼"이고, **Byte 단위의 이진 데이터 자체 또는 저장되는 메모리 공간**으로도 이해할 수 있습니다.

```js
const buffer = Buffer.from("내가 버퍼다.");

console.log(buffer.length);
console.log(buffer.toString());

const arr = [
  Buffer.from("ja"),
  Buffer.from("son"),
  Buffer.from("go"),
  Buffer.from("ose"),
];

const buffer2 = Buffer.concat(arr);
const buffer3 = Buffer.alloc(5);

/**
 * from(문자열)
 * 문자열을 버퍼로 바꿉니다.
 * length 속성으로 버퍼의 바이트 길이를 알 수 있습니다.
 *
 * toString(버퍼)
 * 버퍼를 다시 문자열로 바꿉니다.
 * "base64", "hex"를 인수로 넣으면 해당 인코딩으로 변환할 수 있습니다.
 *
 * concat(배열)
 * 배열 안에 든 버퍼들을 하나로 합칩니다.
 *
 * alloc(바이트)
 * 바이트 길이를 인수로 넣으면 해당 크기의 빈 버퍼를 생성합니다.
 */
```

### 스트림

![Stream](/images/stream.jpeg)

버퍼는 편리하지만 서버처럼 몇 명이 이용할지 모르는 환경에서 메모리 부족 문제가 생길 수 있습니다.

또한 모든 내용을 버퍼에 다 쓴 후에야 다음 동작으로 넘어가므로 파일 읽기, 쓰기, 압축 등의 조작을 연속으로 할 때는 매번 전체 용량을 버퍼로 처리해야 다음 단계로 넘어갈 수 있습니다.

이러한 단점을 보완하고자 버퍼의 크기를 작게 만들고 여러 번에 걸쳐서 데이터를 나눠 보내는 방식인 "스트림"이 등장했고, 스트림을 통해 전달되는 데이터는 "chunk"라고 합니다.

`createReadStream`으로 읽기 스트림을 생성하는데 첫 번째 인자로 읽을 파일 경로를, 두 번째 인자로 옵션 객체를 전달합니다.

아래 코드의 `readStream`은 이벤트 리스너를 붙여서 사용하는데 파일을 읽는 도중에 에러가 발생하면 `error`, 파일 읽기가 시작되면 `data`, 파일을 다 읽으면 `end` 이벤트가 발생합니다.

```js
const fs = require("fs");

const readStream = fs.createReadStream("./readme3.txt", {
  highWaterMark: 16, // 생성할 버퍼의 바이트 길이
});
const data = [];

readStream.on("data", (chunk) => {
  console.log(chunk.length);
  data.push(chunk);
});

readStream.on("end", () => {
  console.log(Buffer.concat(data).toString());
});

readStream.on("error", (err) => {
  // ...
});
```

`createWriteStream`으로 쓰기 스트림을 생생하는데 `write` 메서드로 쓸 데이터를 전달합니다.

데이터를 다 쓰고나서 `end` 메서드로 종료를 알리면 `finish` 이벤트가 발생합니다.

```js
const fs = require("fs");

const writeStream = fs.createWriteStream("./writeme2.txt");

writeStream.on("finish", () => {
  // ...
});

writeStream.write("이 글을 씁니다.");
writeStream.end();
```

파일을 읽는 스트림을 전달받아서 그대로 파일을 쓰는 스트림으로 연결할 수도 있는데 이러한 기법을 "파이핑"이라고 합니다.

```js
const fs = require("fs");
const zlib = require("zlib"); // 내장된 파일압축 모듈

const readStream = fs.createReadStream("readme4.txt");
const zlibStream = zlib.createGzip();
const writeStream = fs.createWriteStream("writeme3.txt");

readStream.pipe(zlibStream).pipe(writeStream);
```

stream 모듈의 `pipeline` 메서드로 여러 개의 파이프를 연결할 수 있고, 중간에 `AbortController`를 사용해서 원하는 시점에 파이핑을 중단할 수 있습니다.

```js
import { pipeline } from "stream/promises";
import zlib from "zlib";
import fs from "fs";

const ac = new AbortController();
const signal = ac.signal;

setTimeout(() => {
  ac.abort();
}, 1);

await pipeline(
  fs.createReadStream("readme4.txt"),
  zlib.createGzip(),
  fs.createWriteStream("writeme3.txt"),
  {
    signal,
  }
);
```

## 기타 fs 메서드 알아보기

`fs`는 파일 시스템을 조작하는 다양한 메서드를 제공하는 파일이나 폴더를 생성하고 삭제할 수 있습니다.

### fs.access

폴더나 파일에 접근할 수 있는지 여부를 확인합니다.

`F_OK`는 파일 존재 여부, `R_OK`는 읽기 권한 여부, `W_OK`는 쓰기 권한 여부를 체크합니다. 

파일/폴더 자체 또는 권한이 없다면 에러가 발생하는, 파일/폴더가 없을 때의 에러 코드는 `ENOENT`입니다.

### fs.mkdir

폴더를 만드는 메서드로 이미 폴더가 있다면 에러가 발생하므로 먼저 `access` 메서드를 호출해서 확인하는 것이 중요합니다.

### fs.open

파일의 아이디(fd 변수)를 가져오는 메서드입니다.

가져온 아이디를 사용해 `fs.read` 또는 `fs.write`로 읽거나 쓸 수 있습니다.

두번째 인수로 파일에 대해서 어떤 동작을 할지 설정할 수 있는데 쓰려면 "w", 읽으려면 "r", 기존 파일에 추가하려면 "a"입니다.

### fs.rename

파일의 이름을 바꾸는 메서드로 잘라내기 기능으로도 사용할 수 있습니다.

```js
const fs = require("fs").promise;
const constants = require("fs").constants;

fs.access("./folder", constants.F_OK | constants.W_OK | constants.R_OK)
  .then(() => {
    return Promise.resoleve("이미 폴더 있음");
  })
  .catch((err) => {
    if (err.code === "ENONENT") {
      console.log("폴더 없음");
      return fs.mkdir("./folder");
    }
    return Promise.reject(err);
  })
  .then(() => {
    console.log("폴더 만들기 성공");
    return fs.open("./folder/file.js", "w");
  })
  .then((fd) => {
    console.log("빈 파일 만들기 성공", fd);
    return fs.rename("./folder/file.js", "./folder/newfile.js");
  })
  .then(() => {
    console.log("이름 바꾸기 성공");
  })
  .catch((err) => {
    console.error(err);
  });
```

### fs.readdir

폴더 안의 내용물을 배열 형태로 확인합니다.

### fs.unlink

파일을 지우는 메서드로 파일이 없다면 에러가 발생하므로 먼저 파일이 있는지를 꼭 확인해야 합니다.

### fs.rmdir

폴더를 지우는 메서드로 폴더 안에 파일들이 있다면 에러가 발생하므로 먼저 내부 파일들을 모두 지우고 호출해야 합니다.

```js
const fs = require("fs").promises;

fs.readdir("./folder")
  .then((dir) => {
    console.log("폴더 내용 확인", dir);
    return fs.unlink("./folder/newfile.js");
  })
  .then(() => {
    console.log("파일 삭제 성공");
    return fs.rmdir("./folder");
  })
  .then(() => {
    console.log("폴더 삭제 성공");
  })
  .catch((err) => {
    console.error(err);
  });
```

### fs.copyFile

파일을 복사하는 메서드입니다.

첫 번째 인수로 복사할 파일의 경로, 두 번째 인수로 복사될 경로, 세 번째 인수로 복사 후 실행할 콜백함수를 전달합니다.

```js
const fs = require("fs").promises;

fs.copyFile("readme.txt", "writeme4.txt")
  .then(() => {
    console.log("복사완료");
  })
  .catch((err) => {
    console.error(err);
  });
```

Node.js v8.5 이전에는 `createReadStream`과 `createWriteStream`을 파이핑하여 파일을 복사했습니다.

### fs.watch

파일/폴더의 변경사항을 감지할 수 있는 메서드입니다.

파일의 내용을 수정할 때는 `change` 이벤트, 파일명을 변경하거나 파일을 삭제하면 `rename` 이벤트가 발생합니다.

`rename` 이벤트가 발생한 뒤에는 `watch` 메서드는 수행되지 않습니다.

`change` 이벤트가 중복으로 발생할 수도 있으니 유의해야 합니다.

```js
const fs = require("fs");

fs.watch("./target.txt", (eventType, filename) => {
  // ...
});
```

## 스레드 풀 알아보기

`fs` 메서드를 여러 번 실행해도 백그라운드에서 동시에 처리되는데, 바로 스레드풀이 있기 때문입니다.

`fs` 외에도 내부적으로 스레드 풀을 사용하는 모듈로는 crypto, zlib, dns.lookup 등이 있습니다.

스레드 풀이 작업을 동시에 처리하므로 처리순서는 요청순서와 다를 수도 있습니다.
