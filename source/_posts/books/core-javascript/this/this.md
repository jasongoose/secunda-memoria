---
title: this
date: 2024-11-01 00:37:26
categories:
  - Books
  - 코어 자바스크립트
#tags:
---
**현재 context를 생성한 상위 context 상의 객체**를 가리키는 키워드입니다.

`this`가 가리키는 객체의 정보는 context가 만들어질 때 context/thisBinding에 저장됩니다.

함수 내부의 `this`가 무엇을 가리키는지는 함수가 호출될 때 결정되므로 함수의 정의만 분석해서 유추해서는 안되고 함수가 언제, 무엇에 의해서 실행되는지를 확인해야 합니다.
