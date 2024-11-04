---
title: Resource
date: 2024-11-03 23:10:05
categories:
  - Studies
  - Web Term
#tags:
---
## 웹 서버 리소스

웹 서버에서 제공하는 정적자원과 동적자원을 의미합니다.

정적자원은 웹 서버 파일 시스템에 저장된 텍스트 파일, HTML 파일, MS Word 파일, JPEG 이미지 파일, AVI 영상파일 등 다양한 형태로 가진 파일을 가리킵니다.

동적자원은 자원을 요청한 주체나 요청시간 등 특정 조건에 따라 다르게 제공되는 자원으로, 카메라로부터 전송된 라이브 이미지, 실시간 주식이나 부동산 데이터 등이 있습니다.

## MIME(Multipurpose Internet Mail Extensions) type

HTTP 프로토콜에 의해서 전달된 데이터의 타입을 구분하기 위한 data format label로, `<primary object type> / <specific subtype>` 형식을 가집니다.

브라우저는 서버로부터 받은 HTTP 메시지의 헤더에 적힌 MIME type을 확인하여 데이터를 포맷에 맞게 처리합니다.

## URI(Uniform Resource Identifier)

클라이언트 입장에서 서버 리소스들을 구분하기 위한 ID입니다.

URI은 [URL](../url) 또는 [URN](../urn) 형태로 가집니다.

## URL

브라우저가 웹 상에서 리소스를 구분하기 위한 고유주소로, 유효한 URL은 HTML 페이지, CSS 문서, 이미지 등의 고유한 리소스를 가리킵니다.

URL 자체 문자열 값과 URL이 가리키는 리소스를 관리하는 것은 웹 서버가 담당합니다.

![URL 구조](/images/url_structure.png)

### scheme(protocol)

브라우저가 리소스를 요청하기 위해서 사용할 프로토콜 이름입니다.

### domain

목적지 웹 서버를 가리키는 주소로 HTTP 요청이 전송되기 전에 DNS 서버에 의해서 IP 주소로 변환됩니다.

### port

웹 서버 상에서 클라이언트가 요청하는 리소스가 위치한 프로세스를 가리키는 값입니다.

표준 포트번호를 사용할 경우에는 생략해도 됩니다(HTTP = 80, HTTPS = 443).

"scheme + domain + port"를 묶어서 origin 또는 authority라고 부릅니다.

### path

웹 서버 상에서 리소스의 경로입니다.

웹 초창기에 실제 서버의 물리적인 파일 시스템 경로값이었지만 지금은 물리적인 실제값이 아닌 서버가 다루기 위한 논리적인 값으로 지정됩니다.

### query(parameters)

웹 서버로 전달하는 부가적인 데이터로, `key=value` 형태의 문자열들을 `&`로 구분합니다.

![Query](/images/query.png)

### anchor

리소스 자체의 특정 영역을 가리키는 북마크 역할을 합니다.

리소스가 HTML 문서인 경우, 브라우저는 anchor가 가리키는 영역(fragment)으로 스크롤링을 해줍니다.

URL에서 `#` 이후의 부분 문자열을 fragment identifier(hash)라고 부르는데 실제 서버로의 요청 메시지 안에는 포함되지 않습니다.

`window.Location.hash`를 사용하면 현재 페이지의 URL의 hash를 읽거나 수정할 수 있습니다.

## URN

서버 리소스의 URL은 서버 내의 경로에 의해서 ID가 변하는 기존 단점을 해결하기 위한 방식입니다.

URN 방식을 사용하면 서버 리소스에 접근하는 프로토콜의 종류나 서버 내의 경로와 상관없이 ID가 1개만 가집니다.

아직 실험단계에 있고 흔하게 사용되는 방식이 아닐 뿐더러 리소스의 위치를 해석하기 위한 인프라가 필요하다는 단점이 있습니다.
