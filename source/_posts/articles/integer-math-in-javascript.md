---
title: Integer math in JavaScript
date: 2024-11-01 22:15:13
categories:
  - Articles
#tags:
---
JS의 `number` 타입은 정수형, 부동소수점형 상관없이 무조건 8B(64bits)를 메모리에 할당합니다.

`number` 타입 데이터들을 [64bit 부동소수점형](https://en.wikipedia.org/wiki/Double-precision_floating-point_format)으로 메모리에 저장하기 때문에 평소에 알고 있던 정수연산과는 다른 결과가 나타날 수 있습니다.

```js
5 / 4 === 1; // false
```

그렇다면 실제 정수연산을 구현할 때는 어떻게 해야할까요?

결론부터 말하자면 다음과 같습니다.

> signed 32bit 정수 연산 결과값에는 "| 0" 연산을 적용합니다.

> unsigned 32bit 정수 연산 결과값에는 [">>> 0"](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Unsigned_right_shift) 연산을 적용합니다.

JS에서는 비트 연산의 두 피연산자들과 반환값들은 `signed 32bit` 라는 점을 이용한 겁니다🤨

```js
const a = (5 / 4) | 0;

a === 1; // true
```

참고로 32bit 정수 간의 곱셈연산 결과값은 길이가 32비트를 넘을 수 있기 때문에 `Math.imul` 함수를 사용합니다.

## 참고자료

[Integer math in JavaScript](https://james.darpinian.com/blog/integer-math-in-javascript)
