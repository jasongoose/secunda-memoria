---
title: The Dao Of Immutability
date: 2024-10-29 22:17:24
categories:
  - Books
  - Composing Software
  - Introduction
#tags:
---
[The Dao of Immutability](https://medium.com/javascript-scene/the-dao-of-immutability-9f91a70c88cd)

## Immutability

> _The true constant is change._

함수를 호출한 전과 후로 context 상의 상태는 유지되어야 한다.

> _Hidden change manifests chaos. Therefore, the wise embrace history._

함수 내에서 context mutation을 직접 수행하는 대신 변경사항이 적용되었을 때 상태를 반환하여 context의 전후 상태를 비교하고, 필요한 mutation은 오직 함수를 호출한 context 상에서만 일어나야만 한다.

[//]: # (여기서 `const`로 선언된 변수는 처음으로 참조하는 메모리 주소의 값이 변하지 않는 즉, 재할당이 안되는 성질을 가질 뿐 불변성과는 전혀 관련이 없습니다.)

## Seperation

> _Logic is thought. Effects are action. Therefore the wise think before acting, and act only when the thinking is done._

로직을 따라서 줄줄이 코드를 작성하면(명령형 방식) 의도치 않은 side-effect가 발생할 수 있으므로 로직을 먼저 생각하고 단일 기능을 가진 함수 단위로 구현한다(선언형 방식).

## Composition

> _Therefore the wise mix ingredients together to make their soup taste better._

고차함수들을 조합하여 로직을 구현한다.

## Conversation

> _Therefore the wise reuse their tools as much as they can. The foolish create special tools for each new creation._

처음 정의한 함수의 인자를 특정 타입만이 아닌 다수 타입에도 동작할 수 있도록 일반화하여 함수의 재사용성을 고려한다.

## Flow

> _A program can be reasoned about and outcomes predicted only when data flows freely through pure functions._

pure function은 immutability를 따르는 함수로, 임의의 context에서 인자를 전달하여 여러번 또는 연속적으로 호출해도 상태는 변하지 않는다.

로직을 일련의 pure function들의 호출로 구현한다.

**기능 자체는 함수이고 기능은 함수들로 구성된다.**
