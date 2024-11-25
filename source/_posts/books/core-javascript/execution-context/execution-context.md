---
title: 실행 컨택스트
date: 2024-11-01 00:39:14
categories:
  - Books
  - 코어 자바스크립트
#tags:
---

JS 엔진이 실행하려는 코드로부터 확보한 환경정보들을 속성으로 가지는 객체입니다.

![Execution Context](/images/exec_context.png)

보통 코드를 실행할 때 처음으로 전역 컨택스트(global context)가 생성됩니다.

그 뒤에 함수를 호출할 때마다 또는 새로운 block(`{...}`, ES6 기준)을 만날 때마다 콜 스택에 새로운 실행 컨택스트(이하 context)가 push되고 top에 위치한 context를 기반으로 JS 엔진이 코드를 실행합니다.

도중에 새로운 context가 콜 스택에 push되면 기존 top에 위치했던 context와 관련된 코드의 실행이 중단되고 그 중단된 시점의 환경정보를 해당 context에 저장합니다.

top context와 관련된 코드의 실행을 마치면 콜 스택에서 pop되어 사라지고 바로 아래에 위치한 context가 top이 되면서 저장된 환경정보를 바탕으로 중단되었던 지점부터 다시 코드를 실행합니다.
