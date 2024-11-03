---
title: Array
date: 2024-11-03 16:18:09
categories:
  - Studies
  - JavaScript
  - Data Structure
#tags:
---
reference 타입 중 하나인 Array는 단일 변수만으로 여러 개의 요소들을 순서에 맞게 저장할 수 있습니다.

## 특징

- 길이가 가변적이고 여러 타입의 요소들을 저장할 수 있습니다.
  - 단일 타입만 저장하고 싶다면 [TypedArray](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypedArray)를 사용하면 됩니다.
- index를 통해서만 요소를 read/write할 수 있습니다.
- zero-indexed
- array-copy 연산은 항상 "shallow-copy"만 생성합니다.

## 참고자료

[Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)