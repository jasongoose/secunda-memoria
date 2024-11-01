---
title: Lexical Environment
date: 2024-11-01 00:41:42
categories:
  - Books
  - 코어 자바스크립트
#tags:
---
## environment record

context를 생성한 함수의 이름, 함수정의에 사용된 매개변수들의 이름, 내부에서 선언한 함수 객체, 선언된 변수명 등 context의 환경정보를 속성으로 가지는 객체로, context 내부를 위에서 아래로 횡단하며 순서대로 수집합니다.

global context의 경우, 런타임별 전역객체를 사용하여 필요한 선언정보들을 수집합니다.

코드를 실행하다가 중단하면 해당 시점의 환경정보를 lexEnv/envRec에 저장합니다.

context를 "lexical scope"라고 칭하는 문서들도 종종 있습니다.

### hoisting

JS 엔진이 context의 lexEnv/envRec 생성을 위해 변수와 함수 선언문들을 context의 최상단으로 끌어올린 후에 필요한 환경정보를 수집한다”라는 일종의 해석방식입니다.

가만히 있는 코드를 JS 엔진이 직접 수정한다는 의미가 전혀 아닙니다.

단순히 환경정보를 수집하는 과정을 쉽게 이해하도록 도입된 가상의 개념이기 때문에 hoisting을 적용하여 코드를 수정해서 실행한 결과와 원본 결과를 비교하면 동일하게 나옵니다.

hoisting 규칙은 다음과 같습니다.

- 함수 호출을 통해 전달한 매개변수들은 최상단에서 변수 선언과 할당이 동시에 이루어진 것으로 처리한다.
- 그 외에 변수선언과 함수선언들은 append하고 원래 할당하는 부분은 그대로 둔다.

### 함수 선언식

`function` 키워드로 함수를 정의하는 방식으로, **선언문 자체가 context 내에서 hoisting**됩니다.

```js
function name(parameter1, parameter2, ...parameterN) {
  // ...body...
}
```

### 함수 표현식

코드를 실행하는 중에 필요할 때 익명함수를 변수에 할당하여 정의하는 방식으로, **할당된 변수의 선언부만 hoisting**됩니다.

```js
let sayHi = function () {
  // 대입연산자를 사용하여 함수를 정의합니다.
  alert("Hello");
};
```

```js
let sayHi = () => {
  alert("Hello");
};

// 위의 sayHi에는 Function object가 할당됩니다.
```

## outer environment reference

콜 스택 내에서 직전 상위 context의 lexEnv를 참조하는 속성입니다.

현재 context 상에 특정 변수나 함수의 선언이 없다면 JS 엔진은 outerEnvRef가 참조하는 상위 context의 환경정보에서 일치하는 선언이 있는지 확인합니다.

outerEnvRef와 lexEnv가 있기 때문에 변수나 함수의 유효범위가 정해집니다.

### scope

선언된 변수에 접근할 수 있는 유효한 범위를 의미하는데, 보통 전역공간을 제외한 함수호출 또는 block(ES6 기준) 또는 모듈에 의해서 변수의 scope가 정해집니다.

> context → scope의 경계

> scope → 현재 context + scope chain으로 연결된 하위 context들

### scope chain

상위 context 방향으로 변수나 함수를 검색하는 과정을 가리킵니다.

여러 context에서 동일한 이름의 변수를 선언한 경우, **무조건 chain 상에서 가장 먼저 발견된 변수에만 접근이 가능**하도록 만듭니다.

scope chain이 일어나는 과정을 정리하면 다음과 같습니다.

1. 현재 lexEnv/envRec에서 찾으려는 변수, 함수의 정보가 있는지 확인한다.
2. 없다면 현재 lexEnv/outerEnvRef → 상위 lexEnv에 접근하여 envRec를 확인한다.
3. 위 과정을 global context의 lexEnv를 참조할 때까지 반복한다.

> scope chain 상에서 최상위 객체 → global context/lexEnv → global object

### 변수 은닉화

scope chain에 의해서 상위 context에 있는 동일한 이름의 변수로 접근할 수 없는 현상을 가리킵니다.

변수 은닉화를 방지하려면 상위 context에서 사용한 변수와 하위 context에서 사용하는 변수의 이름이 중복되지 않아야 합니다.

### 전역변수, 지역변수

전역변수, 지역변수의 여부는 hoisting 이후에 확인하면 됩니다.

> 전역변수 → global context의 lexEnv/envRec에 담긴 변수

> 지역변수 → 그외 context의 lexEnv/envRec에 담긴 변수
