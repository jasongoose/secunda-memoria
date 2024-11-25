---
title: Variable Environment
date: 2024-11-01 00:40:10
categories:
  - Books
  - 코어 자바스크립트
#tags:
---
lexEnv와 동일한 구조와 기능을 가지는데 2가지 차이점을 가집니다.

- context를 생성할 때 varEnv에 가장 먼저 환경정보가 저장되고 lexEnv에 복사된다.
- 초기 context에서의 환경정보 snapshot이 그대로 유지되고, lexEnv의 환경정보는 코드 실행 중 변경사항이 바로 반영된다.
