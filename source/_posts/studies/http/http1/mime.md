---
title: MIME
date: 2024-11-03 14:00:06
categories:
  - Studies
  - HTTP
  - HTTP1
#tags:
---
IANA에서 공식적으로 관리하는 문서, 파일, 바이트 모음 등의 media type입니다.

웹 서비스 같은 경우 서버에서 전달하는 응답 메시지의 `Content-Type` 헤더의 값으로 사용되고, 클라이언트는 해당 값에 따라서 응답을 다르게 처리합니다.

## Structure

type과 subtype 2가지로 구성되고 추가적인 세부사항을 위해서 paramter도 추가할 수 있습니다.

통상적으로 모두 소문자로 지정합니다.

```bash
type/subtype
type/subtype; parameter=value
```

## Types

### discrete

말 그대로 하나의 리소스만을 표현할 때 사용합니다.

### multipart

여러 개의 MIME type으로 구성된 단일 documnet 종류 또는 동일한 transaction에서 주고받는 파일들을 나타낼 때 사용합니다.

#### multipart/form

`POST` 메서드로 `<form></form>`에 입력된 값들을 클라이언트에서 서버로 전달할 때 사용됩니다.

여러 개의 다른 파트들로 구성되고 구분자는 `boundary` parameter 값으로 지정합니다.

각 파트는 개별적인 HTTP 헤더로 `Content-Disposition`과 `Content-Type`를 가집니다.

```bash
Content-Type: multipart/form-data; boundary=aBoundaryString
(other headers associated with the multipart document as a whole)

--aBoundaryString
Content-Disposition: form-data; name="myFile"; filename="img.jpg"
Content-Type: image/jpeg

(data)
--aBoundaryString
Content-Disposition: form-data; name="myField"

(data)
--aBoundaryString
(more subparts)
--aBoundaryString--
```
