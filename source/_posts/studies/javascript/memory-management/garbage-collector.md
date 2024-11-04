---
title: Garbage Collector
date: 2024-11-03 17:27:45
categories:
  - Studies
  - JavaScript
  - Memory Management
#tags:
---
JS에서 메모리 관리는 [JS 엔진](../../ecma/ecma-spec#JavaScript-Engine)의 가비지 컬렉터(Garbage Collector, GC)가 전담합니다.

GC는 할당된 메모리를 모니터링하면서 더 이상 필요가 없다고 판단한 메모리 공간을 회수하는게 주기능인데, 프로그램에 의해서 언제 사용하게 될지도 모르니 대상 메모리 공간을 회수하기 위한 기준은 엄격합니다.

GC에 내장된 메모리 회수 알고리즘은 Reference Count와 Mark & Sweep 알고리즘이 있습니다.

## Reference Count

대상 객체를 참조하는 횟수를 의미합니다.

할당된 메모리를 회수하는 기준은 대상 객체 자체 또는 대상 객체의 속성이나 메서드가 다른 객체에 의해서 참조되는지 여부로, 코드 상에서 참조하는 방법은 변수나 객체의 속성, 함수의 매개변수 등이 있습니다.

GC는 대상 객체의 참조 카운트 값이 0이면 메모리를 회수합니다.

IE6, IE7에서 DOM 요소들의 메모리 회수에서 활용된 이 방식은 순환 참조(circular reference) 문제가 있습니다.

순환참조는 임의의 두 객체가 자신의 속성을 통해서 서로를 참조하는 현상을 가리키는데 이 현상에 의해서 두 객체는 참조 카운트가 항상 0보다 크기 때문에 메모리 회수가 되지 않고 공간만 차지하는 메모리 누수(memory leak)를 일으키면서 브라우저 성능을 저하시킬 수 있습니다.

## Mark & Sweep

전역객체로부터 도달할 수 있는지 여부를 기준으로 가지는데, 도달할 수 있다면 회수하지 않고 도달할 수 없다면 해당 공간을 GC가 비워냅니다.

### Root

JS 코드에서 태생부터 도달가능한 변수들을 루트(root)라고 하는데 관련 예시는 다음과 같습니다.

- 함수의 지역변수 및 매개변수
- scope chain상 함수에서 사용되는 변수와 매개변수
- 전역변수

### 동작방식

JS 엔진은 코드를 실행하면서부터 끊임없이 root로부터 참조할 수 있는 값들을 확인합니다.

여기서 참조하는 방법은 Reference Count 방식과 마찬가지로 변수나 객체의 속성, 함수의 매개변수 등이 가능합니다.

첫 번째 줄에서 전역객체로부터 도달가능한 여부를 따진 이유는 JS에서 전역변수나 함수의 정의는 모두 전역객체의 속성으로 저장되기 때문입니다.

내부적으로 알고리즘의 실행과정은 그래프 이론의 너비 우선 탐색(BFS)과 동일합니다.

먼저 GC가 가능한 root들의 정보를 모으고 각 root를 표시한 뒤, 각 root로부터 도달가능하고 아직 방문하지 않은 모든 객체들을 방문하여 표시하는 과정을 계속 반복합니다.

최종적으로 방문하지 않은 객체들의 메모리 공간은 이제 회수됩니다.

## 참고자료

[Memory Management](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Memory_Management)

[가비지 컬렉션](https://ko.javascript.info/garbage-collection)

[Feature Preview: Incremental Garbage Collection](https://blog.unity.com/technology/feature-preview-incremental-garbage-collection)