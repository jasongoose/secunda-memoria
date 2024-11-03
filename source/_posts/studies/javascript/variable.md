---
title: Variable
date: 2024-11-03 17:59:33
categories:
  - Studies
  - JavaScript
  - Variable
#tags:
---
## Declare → Initialize → Assign

### 선언

변수공간에 변수명만 지정하고 특정 데이터 공간의 주소가 저장되지 없은 상태(`null`)입니다.

### 초기화

선언된 변수공간에 `undefined`를 저장한 데이터 공간의 주소를 지정하는 단계로, 변수에 저장된 초기값이 `undefined`인 상태입니다.

### 할당

특정 데이터 공간에 실제 데이터를 저장하고 그 주소를 변수공간에 저장하는 단계로, 변수를 참조할 때마다 주소가 가리키는 공간에 저장된 데이터를 읽습니다.

## var

function 또는 global scope 변수를 선언할 때 사용하는 키워드입니다.

global context에서 `var`로 선언된 변수들은 모두 전역객체의 read-only property로 처리됩니다.

CommonJS 또는 ESM 모듈 내에 top-level 변수들은 global이 아닌 module scope를 가집니다.

변수를 선언하면 초기화가 동시에 이루어집니다.

hoisting에 의해서 변수의 선언부와 초기화부가 분리되지 않으므로 TDZ가 없어서 실제 선언부 전에 해당 변수를 사용할 수 있는 이상한(?) 성질이 있습니다.

## const, let

block scope를 가지는 변수를 선언할 때 사용하는 키워드입니다.

`var`와 다르게 전역으로 선언되었을 때 전역객체의 property로 처리되지 않습니다.

block statement(`{...}`)을 사용하여 closure를 사용하지 않고 private 변수를 구현할 수 있습니다.

block scope 내에서 hoisting으로 최상단에 변수 선언만 일어납니다.

원본 변수 선언문에 도달해야 변수 초기화(초기값이 없다면 `undefined`)가 수행되는데, scope 최상단과 변수 선언문 사이의 변수를 참조할 수 없는 공간을 TDZ라고 합니다.

`let`, `const`, `var`에 의한 변수 재선언은 불가합니다.

`const`가 `let`과 다른 점들은 아래와 같습니다.

1. 변수를 선언하면서 대입연산자로 할당하여 초기화시켜야 합니다.
2. 선언문 이후로 할당이 불가합니다.

## 참고자료

[📜 var](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/var)

[📜 let](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let)

[📜 const](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const)
