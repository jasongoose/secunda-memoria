---
title: 3-3장. 모듈로 만들기
date: 2024-10-29 23:54:03
categories:
  - Books
  - Node.js 교과서(개정 3판)
  - Chapter 3
#tags:
---
Node.js는 코드를 모듈로 만들 수 있다는 점에서 브라우저의 자바스크립트와는 다릅니다.

모듈이란 특정한 기능을 하는 함수나 변수들의 집합을 의미하는데 모듈로 만들어두면 여러 프로그램에 해당 모듈을 재사용할 수 있습니다.

보통 파일 하나가 모듈 하나가 되며 Node.js에서 2가지 형식이 있습니다.

## CommonJS 모듈

Node.js 생태계에서 가장 널리 쓰이는 모듈로 표준이 나오기 이전부터 쓰였습니다.

### module.exports, exports

```js
// var.js
const odd = "odd";
const even = "even";

module.exports = {
  odd,
  even,
};

// 또는

exports.odd = odd;
exports.even = even;
```

`module.exports`와 `exports`는 동일한 객체를 참조합니다.

`exports`에는 반드시 객체처럼 속성명과 속성값을 대입해야 합니다. `exports`에 다른 값을 대입하면 객체의 참조 관계가 끊겨 더는 모듈로 기능하지 않습니다.

### require

다른 모듈을 불러올 때는 `require`문을 사용하면 됩니다.

```js
console.log("require가 가장 위에 오지 않아도 됩니다.");

module.exports = "저를 찾아보세요.";

require("./var");

console.log("require.cache:: ", require.cache);
console.log("require.main:: ", require.main === module);
console.log("require.main.filename:: ", require.main.filename);
```

`require`가 반드시 파일 최상단에 위치할 필요가 없고 `module.exports`도 최하단에 위치할 필요가 없습니다.

한번 require한 파일은 `require.cache`에 저장되므로 다음 번에 require할 때는 `require.cache`에 있는 것이 재사용됩니다.

`require.main`은 Node.js 실행 시 첫 모듈을 가리키는데 현재 파일이 첫 모듈인지 알아보려면 `require.main === module`을 해보면 됩니다.

### 순환참조

모듈 간의 순환참조가 있을 경우에는 순환참조되는 대상을 빈 객체(`{}`)로 조용히(?) 만듭니다.

```js
// dep1.js
const dep2 = require("./dep2");
```

```js
// dep2.js
const dep1 = require("./dep1");
```

## ECMAScript 모듈

공식 자바스크립트 모듈 형식으로, 브라우저와 Node.js 모두에 같은 모듈 형식을 사용할 수 있다는 것이 장점입니다.

ESM의 `import`, `export default`는 `require`나 `module`처럼 함수나 객체가 아닌 문법 그 자체입니다.

`.mjs` 확장자 대신 `.js` 확장자를 사용하면서 ES 모듈을 사용하려면 package.json에 `type: "module"` 속성을 넣으면 됩니다.

import 시 파일 경로에서 `.js`, `.mjs` 같은 확장자는 생략할 수 없습니다.

```js
// var.mjs
export const odd = "odd";
export const event = "even";
```

```js
// func.mjs
import { odd, even } from "./var.mjs";

function checkOddOrEven(num) {
  return num % 2 ? odd : even;
}

export default checkOddOrEven;
```

```js
// index.mjs
import { odd, even } from "./var.mjs";
import checkOddOrEven from "./func.mjs";

function checkStringOddOrEven(str) {
  return str.length % 2 ? odd : even;
}
```

## 다이내믹 임포트

조건부로 모듈을 불러오는 것을 "다이내믹 임포트"(Dynamic Import)라고 합니다.

CommonJS의 경우 기존에 있던 `require`문을 사용하면 되고 ESM의 경우 `import` 함수를 사용하면 됩니다.

`import` 함수는 Promise를 반환하기 때문에 `await`이나 `then`을 붙여야 합니다.

```js
const a = true;
if (a) {
  const m1 = await import("./func1.mjs");
  console.log(m1);

  import("./var.mjs").then((m2) => {
    console.log(m2);
  });
}
```

## \_\_filename, \_\_dirname

node.js는 `__filename`, `__dirname`이라는 키워드로 현재 스크립트가 실행되는 환경에 대한 정보를 제공하는데 각각 스크립트를 실행한 파일과 디렉토리의 절대경로를 반환합니다.

```js
console.log(__filename);
console.log(__dirname);
```

ESM에서는 사용할 수 없어서 내장된 [`path`와 `url` 모듈로 알아낼 수 있습니다.](https://jootc.com/p/202206123895)

```js
import { dirname } from "path";
import { fileURLToPath } from "url";

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);
```
