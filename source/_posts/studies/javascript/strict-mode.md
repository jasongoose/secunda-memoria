---
title: Strict Mode
date: 2024-11-03 17:46:02
categories:
  - Studies
  - JavaScript
  - Strict Mode
#tags:
---
ES5에서 등장한 개념으로, 코드를 작성하면서 보이지 않게 범할 수 있는 에러들을 잡아주는 JS 코드의 형태입니다.

## Invoking Strice Mode

Strict Mode는 전체 스크립트 코드나 개별 함수 범위에 적용할 수 있습니다.

### scripts

전체 코드에 적용하려면 상단에 다음과 같은 구문을 작성합니다.

```js
// Whole-script strict mode syntax
"use strict";
var v = "Hi! I'm a strict mode script!";
```

## functions

함수 내부에 적용하려면 다음과 같이 작성합니다.

```js
function strict() {
  // Function-level strict mode syntax
  "use strict";
  function nested() {
    return "And so am I!";
  }
  return "Hi!  I'm a strict mode function!  " + nested();
}

function notStrict() {
  return "I'm not strict.";
}
```

### modules

ES2015에 소개된 module의 경우, 전체 코드가 자동적으로 Strict Mode가 적용됩니다.

### classes

ES2015에 소개된 Class defintion, Class expression을 포함한 모든 코드에 자동적으로 Strict Mode가 적용됩니다.

## Changes in Strict Mode

Strict Mode를 적용하면 코드에서는 다음과 같은 변화가 생깁니다.

### Converting mistakes into Errors

1. 선언하지 않은 변수에 값을 할당할 수 없습니다.

```js
"use strict";

let mistypeVariable;

// Assuming no global variable mistypeVarible exists
// this line throws a ReferenceError due to the
// misspelling of "mistypeVariable" (lack of an "a")
mistypeVarible = 17;
```

2. non-writable 전역변수나 속성(property), getter-only property, non-extensible object에 속성을 추가하면 에러가 발생합니다.

```js
"use strict";

// Assignment to a non-writable global
var undefined = 5; // throws a TypeError
var Infinity = 5; // throws a TypeError
```

```js
"use strict";

// Assignment to a non-writable property
var obj1 = {};
Object.defineProperty(obj1, "x", { value: 42, writable: false });
obj1.x = 9; // throws a TypeError
```

```js
"use strict";

// Assignment to a getter-only property
var obj2 = {
  get x() {
    return 17;
  },
};
obj2.x = 5; // throws a TypeError
```

```js
"use strict";

// Assignment to a new property on a non-extensible object
var fixed = {};
Object.preventExtensions(fixed);
fixed.newProp = "ohai"; // throws a TypeError
```

3. 삭제할 수 없는 속성을 삭제하면 에러가 발생합니다.

```js
"use strict";

delete Object.prototype; // throws a TypeError
```

4. 함수인자의 이름이 겹치면 에러가 발생합니다.

```js
function sum(a, a, c) {
  // !!! syntax error
  "use strict";
  return a + a + c; // wrong if this code ran
}
```

5. 팔진수(octal number)를 표시할 때 0-prefix를 붙이면 에러가 발생합니다.

```js
"use strict";

var sum =
  015 + // !!! syntax error
  197 +
  142;

var sumWithOctal = 0o10 + 8; // 대신 ES2015의 0o-prefix를 사용하면 됩니다
console.log(sumWithOctal); // 16
```

6. primitive value에 속성을 추가하면 에러가 발생합니다.

```js
(function () {
  "use strict";

  false.true = ""; // TypeError
  (14).sailing = "home"; // TypeError
  "with".you = "far away"; // TypeError
})();
```

7. 객체에 중복된 속성이 있는 경우 에러가 발생합니다(ES5 기준)

```js
"use strict";

var o = { p: 1, p: 2 }; // syntax error prior to ECMAScript 2015
```

ES2015에서는 computed property names로 뒤에 있는 중복된 속성이 앞의 속성을 덮어쓰기 때문에 문제가 되지 않습니다.

### Simplifying variable uses

1. with 구문을 사용하면 에러가 발생합니다.

```js
"use strict";
var x = 17;
with (obj) {
  // !!! syntax error
  // If this weren't strict mode, would this be var x, or
  // would it instead be obj.x?  It's impossible in general
  // to say without running the code, so the name can't be
  // optimized.
  x;
}
```

2. `eval("")` syntax의 내부 expression에서 발생하는 일은 `eval`를 포함하는 주변 scope에 영향을 주지 않습니다.

```js
"use strict";
var x = 2,
  y = 3;
// NB: Strictness propagates into eval code evaluated by a
//     direct call to eval — a call occurring through an
//     expression of the form eval(...).
print(eval("var x = 5; x")); // prints 5
print(eval("var x = 7; x")); // prints 7
print(eval("y")); // prints 3
print(x); // prints 2
```

3. 객체의 속성을 제외한 일반변수에 `delete` 연산을 적용하면 에러가 발생합니다.

```js
"use strict";

var x;
delete x; // !!! syntax error

eval("var y; delete y;"); // !!! syntax error
```

## Making eval and arguments simpler

1. `eval`, `arguments`의 바인딩(binding)이나 할당(assigning)을 차단합니다.

```js
"use strict";

// assigning disabled!
eval = 17;
arguments++;
++eval;

// binding disabled!
var obj = { set p(arguments) {} };
var eval;
try {
} catch (arguments) {}
function x(eval) {}
function arguments() {}
var y = function eval() {};
var f = new Function("arguments", "'use strict'; return 17;");
```

2. `arguments`는 함수가 실행될 때 전달된 인자들의 정보만 가집니다.

```js
function f(a) {
  "use strict";
  a = 42;
  return [a, arguments[0]]; // strict mode가 아니었다면 arguments[0]에 a값이 저장됩니다.
}

var pair = f(17);
console.assert(pair[0] === 42);
console.assert(pair[1] === 17);
```

### Securing Javascript

1. 함수 내부에서 `this`를 지정하지 않으면 [globalThis](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/globalThis) 대신 `undefined`를 가집니다.

```js
"use strict";

function fun() {
  return this;
}

console.assert(fun() === undefined);
console.assert(fun.call(2) === 2);
console.assert(fun.apply(null) === null);
console.assert(fun.call(undefined) === undefined);
console.assert(fun.bind(true)() === true);
```

2. 함수 내부에서 함수의 `caller`, `arguments`를 사용할 수 없습니다.

```js
function fun(a, b) {
  "use strict";
  var v = 12;
  return arguments.caller; // throws a TypeError
}

fun(1, 2); // doesn't expose v (or a or b)
```

### Paving the way for future ECMAScript versions

미래에 새로운 ECMAScript의 새로운 syntax나 기능들을 공개될 수 있도록 스포일러를 방지합니다.

1. 아래 리스트에 포함된 이름들은 reserved keyword로 변수명, 함수명, 인자명으로 사용할 수 없습니다.

- implements
- interface
- let
- package
- private
- protected
- public
- static
- yield

2. script나 function의 상단에 함수를 정의하지 않으면 에러가 발생합니다(근데 ES2015에서는 허용된다).

```js
"use strict";

if (true) {
  function f() {} // !!! syntax error
  f();
}

for (var i = 0; i < 5; i++) {
  function f2() {} // !!! syntax error
  f2();
}

function baz() {
  // kosher
  function eit() {} // also kosher
}
```
