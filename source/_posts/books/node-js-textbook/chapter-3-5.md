---
title: 3-5장. 노드 내장 모듈 사용하기
date: 2024-10-29 23:54:29
categories:
  - Books
  - Node.js 교과서(개정 3판)
  - Chapter 3
#tags:
---
## os

프로세스가 실행되고 있는 OS의 정보를 가져올 수 있는 모듈로, OS별 다른 서비스를 제공할 때 유용합니다.

```js
// const os = require("os");
const os = require("node:os");

os.arch(); // process.arch와 동일
os.platform(); // process.platform과 동일
os.type(); // OS의 종류
os.uptime(); // OS 부팅 이후 흐른 시간(초)
os.hostname(); // 컴퓨터의 이름
os.release(); // OS의 버전
os.homedir(); // 홈 디렉터리 경로
os.tmpdir(); // 임시 파일 저장 경로
os.cpus(); // 컴퓨터의 코어 정보
os.freemem(); // 사용 가능한 RAM의 용량
os.totalmem(); // 전체 RAM 용량
os.cpus().length; // CPU 코어의 개수
```

## path

폴더와 파일의 경로를 OS의 종류와 상관없이 쉽게 조작하도록 도와주는 모듈입니다.

```js
const path = require("path");

path.sep; // 경로 구분자(Window는 \, POSIX는 /)
path.delimeter; // 환경변수 구분자(Window는 ;, POSIX는 :)

path.dirname(경로); // 파일이 위치한 폴더의 경로
path.extname(경로); // 파일의 확장자
path.basename(경로, 확장자); // 확장자를 포함한 파일의 이름, 2번째 인수로 확장자를 넘기면 파일의 이름만 나옴
path.parse(경로); // 파일의 경로를 root, dir, base, ext, name으로 분리한 객체
path.format(객체); // path.parse()에서 반환된 객체를 파일 경로로 변환하여 반환
path.normalize(경로); // 경로 구분자를 여러 번 사용하거나 혼용한 부분을 수정하여 정상적인 경로로 변환
path.isAbsolute(경로); // 파일의 경로가 절대경로인지 여부(boolean형)
path.relative(기준경로, 비교경로); // 기준경로에 대한 비교경로의 상대경로
path.resolve(경로, ...); // 여러 인수를 넣으면 왼쪽 방향으로 하나의 절대경로로 합쳐서 반환
path.join(경로, ...); // 여러 인수를 넣으면 오른쪽 방향으로 하나의 경로로 합쳐지면서 부모 디렉터리(..)와 현재 위치(.)도 자동으로 처리

path.posix.sep; // Window에서 사용할 POSIX 스타일 구분자
path.posix.join(경로, ...); // Window에서 사용할 POSIX 스타일 join

path.win32.sep; // POSIX에서 사용할 Window 스타일 구분자
path.win32.join(경로, ...); // POSIX에서 사용할 Window 스타일 join
```

`path.resolve`의 경우 인수들을 합치는 과정에서 절대경로('/'로 시작하는) 인수를 만나서 합친 이후에는 더 이상 합치지 않지만 `path.join`의 경우 상대경로로 처리합니다.

```js
path.resolve("/a", "/b", "c"); /* /b/c */
path.join("/a", "/b", "c"); /* /a/b/c */
```

절대경로는 루트 폴더(Window의 "C:\", POSIX의 "/")나 Node.js 프로세스가 실행되는 위치가 기준이 되고 상대경로는 현재 파일이 기준이 됩니다.

## url

![WHATWG URL](/images/whatwg_url.png)

WHATWG(웹 표준을 정하는 단체의 이름) 방식의 URL을 쉽게 조작하도록 도와주는 모듈로, 참고로 이미 Node.js에 내장된 객체라서 require하지 않아도 됩니다.

```js
const url = require("url");
const { URL } = url;

const urlObj = new URL(
  "http://www.gilbut.co.kr/book/bookList.aspx?sercate1=001001000#anchor"
);
/*
{
    origin: 'http://www.gilbut.co.kr',
    protocol: 'http:',
    username: '',
    password: '',
    host: 'www.gilbut.co.kr',
    hostname: 'www.gilbut.co.kr',
    port: '',
    pathname: '/book/bookList.aspx',
    search: '?sercate1=001001000',
    searchParams: URLSearchParams {...}
    },
    hash: '#anchor',
    href: 'http://www.gilbut.co.kr/book/bookList.aspx?sercate1=001001000&category=nodejs#anchor',
    toJSON: ƒ toJSON(),
    toString: ƒ toString(),
    constructor: ƒ URL()
  }
*/

const formattedUrl = url.format(urlObj); // URL 생성자로 분해된 url 객체를 다시 조립합니다.
```


host 부분 없이 pathname 부분만 인자로 전달한 경우, WHATWG 방식은 이 주소를 처리할 수 없습니다.

search 영역은 보통 주소를 통해 데이터를 전달할 때 사용되고 search 부분을 다루기 위해 `searchParams`라는 특수한 객체가 생성됩니다.

```js
const { searchParams } = urlObj;

searchParams.getAll("category"); // 키에 해당하는 모든 값들을 배열로 반환
searchParams.get("sercate1"); // 키에 해당하는 첫 번째 값만 반환
searchParams.has("page"); // 해당 키가 존재하는지 여부를 반환

searchParams.keys(); // searchParams의 모든 키들을 반환하는 iterator를 반환
searchParams.values(); // searchParams의 모든 값들을 반환하는 iterator를 반환

searchParams.append("filter", "es3"); // 동일한 키의 존재여부와 상관없이 해당 키와 값을 그대로 추가
searchParams.set("filter", "es6"); // 동일한 키가 있는 경우 지정된 값으로 수정
searchParams.delete("filter"); // 해당 키를 제거
searchParams.toString(); // searchParams 객체를 다시 문자열로 반환
```

## dns

DNS를 다룰 때 사용하는 모듈로, 주로 도메인을 통해 IP나 기타 DNS 정보를 얻고자할 때 사용합니다.

```js
import dns from "dns/promises";

const ip = await dns.lookup("gilbut.co.kr"); // ip 주소 검색

const a = await dns.resolve("gilbut.co.kr", "A"); // ipv4 주소 검색

const aaaa = await dns.resolve("gilbut.co.kr", "AAAA"); // ipv6 주소 검색

const ns = await dns.resolve("gilbut.co.kr", "NS"); // 네임 서버 검색

const soa = await dns.resolve("gilbut.co.kr", "SOA"); // 도메인 정보

const cname = await dns.resolve("www.gilbut.co.kr", "CNAME"); // 별칭

const mx = await dns.resolve("gilbut.co.kr", "MX"); // 메일 서버 검색
```

## crypto

다양한 암호화 방식의 암호화를 도와주는 모듈입니다.

### 단방향 암호화

복호화할 수 없는 암호화 방식으로 보통 비밀번호 암호화에 활용되고 원본 데이터로 원복할 수 없으므로 해시 함수라고 부르기도 합니다.

로그인할 때마다 입력받은 비밀번호를 암호화한 후 DB의 비밀번호와 비교하면 됩니다.

단방향 암호화 알고리즘은 주로 해시 기법을 사용하는데, 해시 기법은 어떠한 문자열을 고정된 길이의 다른 문자열로 바꿔버리는 방식입니다.

```js
const crypto = require("crypto");

crypto.createHash("sha512").update("비밀번호").digest("base64");

/**
 * createHash(알고리즘): 사용할 해시 알고리즘을 지정하는 함수로 md5, sha1, sha256, sha512 등이 가능합니다.
 * update(문자열): 변환할 문자열을 입력합니다.
 * digest(인코딩): 인코딩할 알고리즘을 지정하고 변환된 문자열을 반환하는 함수로 base64, hex, latin1이 주로 사용됩니다.
 */
```

현재는 주로 pbkdf2, bcrypt, scrypt라는 알고리즘으로 비밀번호를 암호화하고 있습니다.

여기서 pbkdf2는 기존 문자열에 "salt"라고 불리는 문자열을 붙인 후 해시 알고리즘을 반복해서 적용하는 겁니다.

```js
const crypto = require("crypto");

crypto.randomBytes(64, (err, buff) => {
  const salt = buff.toString("base64");
  crypto.pbkdf2("비밀번호", salt, 100_000, 64, "sha512", (err, key) => {
    // 암호화 이후...
  });
});
// 위 예시에서는 sha512로 변환된 결과값을 10만번 반복합니다.
```

싱글 스레드 프로그래밍을 할 때 1초 동안 블로킹이 되는 것은 아닌지 걱정할 수도 있지만 다행히 `crypto.randomBytes`와 `crypto.pbkdf2` 메서드는 내부적으로 스레드 풀을 사용해 멀티 스레딩으로 동작합니다.

pbkdf2는 간단하지만 bcrypt나 scrypt보다 취약하므로 나중에 더 나은 보안이 필요하면 bcrypt, scrypt를 사용하면 됩니다.

### 양방향 대칭형 암호화

암호화된 문자열을 복호화할 수 있으며 키가 사용됩니다.

대칭형 암호화에서는 암호를 복호화하려면 암호화할 때 사용한 키와 같은 키를 사용해야 합니다.

```js
const crypto = require("crypto");

const algorithm = "aes-256-cbc";
const key = "key";
const iv = "iv";

const cipher = crypto.createCipheriv(algorithm, key, iv);

let result = cipher.update("암호화할 문장", "utf8", "base64");
result += cipher.final("base64");

console.log("암호화:: ", result);
```

더 같단하게 암호화하고 싶다면 [crypto-js](https://www.npmjs.com/package/crypto-js?activeTab=readme) 패키지를 사용할 수 있습니다.

## util

각종 편의 기능을 모아둔 모듈입니다.

```js
const util = require("util");
const crypto = require("crypto");

const dontUseMe = util.deprecate((x, y) => {
  // deprecate 처리될 함수
}, "deprecate 경고 메시지");

// deprecate
// 이전 사용자를 위해 기능을 제거하지는 않지만 곧 없앨 예정이므로 더 이상 사용하지 말라는 의미입니다.

dontUseMe(10, 30);

const randomBytesPromise = util.promisify(crypto.randomBytes);

randomBytesPromise(64)
  .then((buf) => {
    //
  })
  .catch((err) => {
    //
  });

/**
 * util.deprecate: deprecate 처리될 함수를 사용하는 경우 경고 메시지가 출력됩니다.
 * util.promisify: 콜백 패턴을 promise 패턴으로 바꿔줍니다.
 */
```

## worker_threads

![Worker Diagram](/images/worker_diagram.jpeg)

Node.js에서 멀티 스레드 방식으로 작업하기 위한 모듈입니다.

```js
const { Worker, isMainThread, parentPort } = require("worker_threads");

if (isMainThread) {
  // 메인 스레드가 실행하는 영역
  const worker = new Worker(__filename); // 현재 파일을 실행할 워커 스레드 생성
  worker.on("message", (message) => {
    // 워커로부터 메시지 수신 시 핸들링
  });
  worker.on("exit", () => {
    // 워커가 close 메서드로 종료할 시 핸들링
  });
  worker.postMessage("ping"); // 워커 스레드로 데이터 전송
} else {
  // 워커 스레드가 실행하는 영역
  parentPort.on("message", (value) => {
    // 메인 스레드로부터 메시지 수신 시 핸들링
    parentPort.postMessage("pong"); // 메인 스레드로 데이터 전송
    parentPort.close(); // 부모 스레드와의 연결 종료
  });
}
```

여러 개의 워크 스레드를 생성하는 경우 `Set`을 사용할 수 있습니다.

```js
const {
  Worker,
  isMainThread,
  parentPort,
  workerData,
} = require("worker_threads");

if (isMainThread) {
  const threads = new Set();
  // 워커 스레드를 생성할 때 workerData를 통해 워커 스레드로 데이터를 전송
  threads.add(
    new Worker(__filename, {
      workerData: {
        start: 1,
      },
    })
  );
  threads.add(
    new Worker(__filename, {
      workerData: {
        start: 2,
      },
    })
  );
  for (let worker of threads) {
    worker.on("message", (message) => {
      //
    });
    worker.on("exit", () => {
      threads.delete(worker);
    });
  }
} else {
  const data = workerData;
  parentPort.postMessage(data.start + 10);
}
```

스레드를 생성하고 스레드 사이에서 통신하는데 상당한 비용이 발생하므로 이 점을 고려해서 멀티 스레딩을 고려해야 합니다.

잘못하면 멀티 스레딩을 할 때 싱글 스레드보다 더 느려지는 현상도 발생할 수 있습니다.

## child_process

Node.js에서 다른 프로그램 또는 명령어를 수행하고 싶을 때 사용하는 모듈입니다.

이름이 "child_process"인 이유는 현재 프로세스 외에 새로운 프로세스를 띄워서 명령을 수행하고 다시 부모 프로세스로 결과를 알려주기 때문입니다.

```js
const { exec } = require("child_process");

// 인자로 전달된 명령어를 그대로 수행합니다.
const process = exec("dir");

process.stdout.on("data", function (data) {
  console.log(data.toString());
});

process.stderr.on("data", function (data) {
  console.log(data.toString());
});
```

```js
const { spawn } = require("child_process");

// 새로운 프로세스를 띄우면서 명령어를 수행합니다.
const process = spawn("python", ["test.py"]);

process.stdout.on("data", function (data) {
  console.log(data.toString());
});

process.stderr.on("data", function (data) {
  console.log(data.toString());
});
```

결과값은 `process.stdout`과 `process.stderr`에 붙여둔 "data" 이벤트 리스너에 Buffer 형태로 전달됩니다.
