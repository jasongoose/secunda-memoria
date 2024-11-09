---
title: Why FP In JS?
date: 2024-10-29 22:18:21
categories:
  - Books
  - Composing Software
  - Introduction
#tags:
---
## 왜냐하면...

JS는 아래와 같은 FP 언어의 주요 특징들을 가지고 있습니다.

- 함수가 일급객체이다.
- [lambda syntax](https://www.w3schools.com/python/python_lambda.asp)처럼 함수를 정의할 수 있다(arrow function).
- closure가 가능하다.

## 하지만...

기존의 FP 언어에 비해 부족한 부분들을 짚어보면 다음과 같습니다.

### Purity는 선택적이다.

FP에서 모든 함수들은 pure function이어야 하는데 JS에서 모든 함수들이 pure function으로 만들어지지는 않습니다.

### Immutability는 선택적이다.

FP에서는 원본 객체를 수정하는 대신 수정된 버전의 객체를 새로 생성하지만 JS는 원본 객체의 속성을 수정할 수 있습니다.

### Recursion은 선택적이다.

FP에서 순회할 때는 recursion만 가능하지만 JS는 for-loop, while 문으로도 가능합니다.

## 그래도...

JS가 FP를 공부하고 개발하기에 적합한 언어인 이유는 다음과 같습니다.

### general-purpose

웹앱 뿐만 아니라 서버 사이드, 임베디드, AI 등 다양한 분야에서 사용되므로 다양한 직군의 개발자들이 동일한 언어로 생각을 공유하는 생태계가 형성될 수 있습니다.

### multi-paradigm

회사 또는 팀 내부에서 선호하는 어떤 방식으로도 소스를 작성할 수 있어서 도입장벽이 높지 않습니다.

- 프로토타입 기반(prototypal)
- 절차지향형(procedural)
- 객체지향형(obect-oriented)
- 함수형(functional)
- …
