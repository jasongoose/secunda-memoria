---
title: Blob
date: 2024-11-03 16:49:41
categories:
  - Studies
  - JavaScript
  - Interface
#tags:
---
read-only raw data를 가지는 file-like object로, 데이터 내용은 text, binary data, stream으로 변환하여 확인할 수 있습니다.

JS native 타입을 가질 필요가 없는 데이터가 `Blob`로 많이 표현됩니다.

`<form></form>`을 통해서 입력받은 file 데이터가 `Blob`로 변환되어 `POST` request body로 전달됩니다.
