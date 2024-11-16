---
title: What is FP?
date: 2024-10-29 22:17:58
categories:
  - Series
  - Composing Software
  - Introduction
#tags:
---
소프트웨어를 순수함수 단위로 구성하는 선언형 프로그래밍 기법입니다.

## Pure Function

다음 2가지 조건을 충족하는 함수를 가리킵니다.

- 동일한 input에 대해서 언제나 동일한 output이 나온다.
- side-effect가 발생하지 않는다.

## Function Composition

다수의 함수들을 합성하여 새로운 함수를 만들거나 연산을 수행하는 작업을 의미합니다.

## Shared State

공유된 scope 내에 존재하는 모든 변수, 객체 등이 있는 메모리 공간입니다.

Shared State는 다음과 같은 문제점을 가집니다.

- 함수를 이해하려면 해당 함수에 의해 mutate되는 모든 Shared State들의 종류와 관련 히스토리들을 모두 알아야 한다.
- 일련의 함수들이 mutate하는 순서와 타이밍에 따라 최종 state가 달라지는 race condition이 발생한다.

## Immutable Object

처음 생성된 뒤로 속성을 수정할 수 없는 객체입니다.

보통 `Object.freeze` 메서드로 root-level freezing을 구현할 수 있는데, deep-level까지 freeze하려면 [Immutable.js](https://immutable-js.com/) 또는 [Mori](https://swannodette.github.io/mori/) 등의 라이브러리를 활용합니다.

## Side Effect

호출된 함수의 외부에 있는 state에서 발생하는 변화로, 아래와 같은 상황에서 주로 발생합니다.

- 전역 scope 또는 부모 scope에 위치한 외부 변수나 객체의 속성을 수정한 경우
- 파일을 작성한 경우(Disk I/O)
- 네트워크를 요청한 경우(Network I/O)
- side-effect가 있는 함수를 호출한 경우

## Higher Order Function

함수를 인자 또는 반환값으로 가지는 고차함수를 의미하는데, 아래와 같은 사용이점들이 있습니다.

- 데이터 타입에 일반화하여 재사용하기 용이한 함수들을 만들 수 있다.
- 콜백함수와 Promise로 함수 내의 비동기 흐름을 제어할 수 있다.
- 부분적용함수, [커리함수](../../concepts/curry-function)로 function composition을 구현할 수 있다.
