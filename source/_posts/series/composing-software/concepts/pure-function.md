---
title: Pure Function
date: 2024-10-28 08:41:01
categories:
  - Series
  - Composing Software
  - Concepts
#tags:
---
## 함수의 용도

### Mapping

input에 대응되는 output을 생성합니다.

### Procedure

재사용될 일련의 명령어들을 묶은 단위로, C와 같은 절차지향적 언어로 개발할 때 활용됩니다.

### I/O

네트워크 요청과 같이 시스템의 다른 부분들과 통신하기 위한 수단입니다.

## 순수함수란?

위에서 Mapping과 관련된 개념으로 아래 성질들을 모두 만족하는 함수입니다.

### 동일한 input에 대해서 언제나 동일한 output이 나온다.

객체 인자가 전달되면 수정된 버전의 새로운 객체를 반환하거나 `Object.assign` 메서드로 중첩된 객체만 다른 객체로 대체합니다.

함수 호출 전후로 기존 context의 state는 유지되므로 순수함수의 동작은 예측이 용이하고 테스트 단위로 적절합니다.

### side-effect가 발생하지 않는다.

상위 context에 위치한 state(매개수가 참조하는 외부 객체)에 전혀 영향이 없습니다.

이러한 순수함수는 여러 함수들의 비동기적인 접근이나 병렬작업으로 shared state의 결과를 예측할 수 없는 race-condition 문제를 해결합니다.
