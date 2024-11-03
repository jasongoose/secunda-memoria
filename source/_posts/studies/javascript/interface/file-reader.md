---
title: FileReader
date: 2024-11-03 16:49:49
categories:
  - Studies
  - JavaScript
  - Interface
#tags:
---
로컬 호스트에 저장된 파일이나 raw data를 브라우저에서 비동기적으로 읽기 위한 인스턴스를 제공하는 생성자로, `Blob` 또는 `File`로 표현된 데이터를 읽습니다.

`FileReader` 인스턴스로 읽은 파일의 내용은 `result` 속성으로 알 수 있는데, `result`의 데이터 타입은 사용하는 읽기 메서드들(`readAs-*` prefixed)마다 다릅니다.

- `.readAsArrayBuffer` : file 데이터를 가진 `ArrayBuffer`
- `.readAsBinaryString` : binary 데이터를 가진 `String`
- `.readAsDataUrl` : file 데이터를 참조하는 `data: URL`
- `.readAsText` : file 내용을 표현한 `String`

## 참고자료

[FileReader](https://developer.mozilla.org/en-US/docs/Web/API/FileReader)