---
title: 상황에 따라 달라지는 this
date: 2024-11-01 00:39:14
categories:
  - Books
  - 코어 자바스크립트
#tags:
---
1. global context → 전역객체
2. 메서드 내부 → 메서드를 포함한 객체
3. 독립적인 함수 내부 → 전역객체(JS 설계오류)
4. ES6 arrow function 내부 → scope chain 상 가장 가까운 context의 thisBinding
5. 콜백함수 내부 → 제어권을 가진 함수가 지정해주는 객체(default: 전역객체)
6. constructor 내부 → 생성된 인스턴스
