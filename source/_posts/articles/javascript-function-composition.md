---
title: JAVASCRIPT FUNCTION COMPOSITION, WHAT’S THE BIG DEAL?
date: 2024-11-01 22:15:58
categories:
  - Articles
#tags:
---
함수형 프로그래밍 방식이 왜 중요한지를 요약하면 다음과 같습니다.

## function composition

직역하면 중고등학생 시절에 질리도록 공부했던 “합성함수”로, 다수의 함수들을 결합하여 만든 새로운 함수를 만들거나 연산을 수행하는 작업입니다.

## compose 구현하는 방법

인자로 전달된 합성 대상 함수들을 array로 받고 `reduceRight` 메서드로 역순으로 차례대로 수행하면서 반한된 값은 다음 함수의 인자로 전달합니다.

```js
const compose =
  (...fns) =>
  (x0) =>
    fns.reduceRight((x, f) => f(x), x0);

const processComment = compose(linkify, imagify, emphasize, headalize);
```

아니면 동일한 원리지만 반대로 합성할 때는 `reduceRight` 대신 `reduce` 메서드를 사용하면 됩니다.

```js
const composeRev =
  (...fns) =>
  (x0) =>
    fns.reduce((x, f) => f(x), x0);

const processComment = composeRev(linkify, imagify, emphasize, headalize);
```

위와 같이 재사용 가능한 함수가 아닌 일회성으로 사용하여 결과값만 얻고 싶다면 간단하게 수정하면 됩니다.

```js
const pipe = (x0, ...fns) => fns.reduce((x, f) => f(x), x0);

const result = pipe(r0, linkify, imagify, emphasize, headalize);
```

## 함수 단위로 코드를 작성한다면...

### 개연성을 높일 수 있다.

기존의 명령형(imperative) 코드에 비해서 선언형(declarative) 코드에 맞춰서 작성하면 코드를 더 쉽게 이해할 수 있습니다.

`pipe`만 보더라도 마치 `r0` 값이 `linkify` → `imagify` → `emphasize` → `headalize` 순으로 거친다는 점을 직관적으로 파악할 수 있습니다.

### 코드를 작성하기 위한 사고방식과 관점을 바꿀 수 있다.

**컴퓨터에게 실행할 일련의 명령어들**이 아닌 **작고 재사용 가능한 함수(기계 부품)들의 조합**으로 이해할 수 있습니다.

## imperative vs. declarative

### imperative code

```js
function removeOdd(items) {
  const results = [];
  
  for (let i = 0; i < items.length; i++) {
    if (items[i] % 2 === 0) {
      results.push(items[i])
    }
  
  }
  return results;
}
```

원하는 결과를 얻기까지의 과정을 코드로 작성하는 방식으로 코드가 길이가 길어질 뿐만 아니라 요구사항의 구현하는 방식이 각자 다르기 때문에 버그가 빈번하게 발생할 수 있다는 문제가 있습니다.

특정 함수의 역할과 기능을 이해하려면 정의를 한줄씩 해석해야 합니다.

### decalarative code

```js
function checkForOdd(n) {
  return n % 2 === 0;
}

function removeOdd(items) {
  return items.filter(checkForOdd);
}
```

원하는 결과만 선언하는 코드를 작성하는 방식으로 특정 기능을 수행 + 재사용이 가능한 함수를 정의하고 함수 단위로 코드를 작성합니다.

물론 이 함수를 정의할 때는 imperative code를 작성하겠지만 실제로 해당 함수를 사용할 때는 정의 부분을 볼 필요가 없습니다.

## 출처

[JAVASCRIPT FUNCTION COMPOSITION: WHAT’S THE BIG DEAL?](https://jrsinclair.com/articles/2022/javascript-function-composition-whats-the-big-deal/)
