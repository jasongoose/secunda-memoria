---
title: Message Structure
date: 2024-11-03 13:59:53
categories:
  - Studies
  - HTTP
  - HTTP1
#tags:
---
![HTTP1 Message](/images/http1_message.png)

## Start Line

### 요청

![Start Line - Request](/images/http1_start_line_req.jpeg)

#### Method

[Methods](../methods)

#### URI

URI는 아래와 같은 형태들이 가능합니다.

##### absolute path

`GET`, `POST`, `HEAD`, `OPTIONS`에서 주로 사용합니다.

형태 : path( + ‘?’ + query)

```bash
POST / HTTP/1.1
```

```bash
GET /background.png HTTP/1.0
```

```bash
HEAD /test.html?query=alibaba HTTP/1.1
```

```bash
OPTIONS /anypage.html HTTP/1.0
```

##### complete URL

HTTP Proxy로의 `GET` 요청에서 주로 사용합니다.

형태 : origin + path( + ? + query)

```bash
GET https://developer.mozilla.org/en-US/docs/Web/HTTP/Messages HTTP/1.1
```

##### authority component

주로 `CONNECT` 메서드에서 사용합니다.

형태 : domain + port

```bash
CONNECT developer.mozilla.org:80 HTTP/1.1
```

##### asterisk form(\*)

주로 `OPTIONS` 메서드에서 주로 사용합니다.

형태 : \*(가능한 모든 서버들을 가리킵니다)

```bash
OPTIONS * HTTP/1.1
```

#### HTTP Version

Start Line 아래로 작성되는 요청 메시지와 추후 응답 메시지의 구조를 나타냅니다.

### 응답

응답 메시지의 경우, 아래와 같이 HTTP version + Status Code로만 간단하게 구성됩니다.

![Start Line - Response](/images/http1_start_line_res.jpeg)

## Headers

요청/응답 메시지에 대한 추가적인 정보를 담는 부분으로 다음과 같은 구성을 가집니다.

```bash
<헤더명>: <헤더값>

// 헤더명은 case-insensitive
// 헤더값 앞의 공백문자는 무시됩니다.
```

### Request Header

요청 메시지의 context를 전달하여 서버가 클라이언트에 맞춰서 응답을 생성할 수 있도록 합니다.

cross-orign으로 리소스를 요청할 때, Preflight Request에서 생략할 수 있는 헤더들을 "CORS-safe Request Header"라고 합니다.

```bash
Accept
Accept-Language
Content-Language
Content-Type
```

- 헤더값은 아래와 같은 조건을 만족해야 합니다.

- 모든 헤더값은 최대 128개의 문자만 포함할 수 있다.

- `Content-Type` 으로 가능한 값들
  - `application/x-www-form-urlencoded`
  - `multipart/form-data`
  - `text/plain`

- `Accept`, `Content-Type`으로 불가능한 값들
    - ` " ( ) : < > ? @ [ \ ] { } 0x7F(DEL)`
    - CORS-unsafe Request Header byte를 포함한 경우
      - `0x00 ~ 0x1F // 단, 0x09(HT) 제외`

- `Accept-Language` , `Content-Language`으로 가능한 값들
  - `0 ~ 9`
  - `A ~ Z`
  - `a ~ z`
  - `' '`
  - `- . ; =`

### Response Header

응답 메시지의 내용과 상관없는 context를 전달합니다.

### Representation Header

요청/응답 메시지의 body에 담긴 데이터의 표현방식(언어, 압축여부, 인코딩 방식, MIME 타입 등) 정보를 가집니다.

클라이언트가 `Accept-*` 헤더를 통해 원하는 응답 format에 관한 정보를 서버로 전송하면(content-negotiation 단계) 서버는 해당 헤더를 통해 선택된 format에 관한 정보를 전송합니다.

가능한 헤더들은 다음과 같습니다.

```bash
Content-Type
Content-Encoding
Content-Language
Content-Location
```

### Payload Header

요청/응답 메시지의 안전한 전송과 복원을 위한 정보(payload 길이, payload의 순번, 전송방식, 고결성 검사 등)를 담고있는 헤더입니다.

```bash
Content-Length
Content-Range
Transfer-Encoding
Trailer
```

## Body

서버 또는 클라이언트로 전송할 데이터가 위치하는 부분입니다.

### 요청

보통 업데이트 목적으로 `POST` , `PUT`, `PATCH` 메서드에서 주로 사용됩니다.

### 응답

#### single file + known length

`Content-Type` 헤더와 `Content-Length` 헤더에 의해서 정의됩니다.

#### single file + unknown length

`Transfer-Encoding: chunked` 헤더에 의해서 여러 개의 chunk를 가집니다.

#### Multiple-resource bodies

하나의 body가 다수의 part로 구성됩니다.

`<form></form>` 에 입력한 데이터를 요청 메시지에 담을 때 사용되고, 응답 메시지에서는 거의 사용되지 않는 방식입니다.
