---
title: CORS
date: 2024-11-03 18:49:44
categories:
  - Studies
  - Security
#tags:
---
Cross-Origin Resource Sharing의 줄임말로, 서버 origin과 현재 클라이언트에 표시된 페이지의 origin이 다른 상황에서 리소스를 주고받기 위한 HTTP 헤더 기반 메커니즘입니다.

보통 클라이언트 페이지에서 [XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest) 또는 [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/fetch)로 리소스를 요청할 때, SOP(Same-Origin Policy) 정책에 의해서 origin 값이 동일할 서버로만 요청을 전송할 수 있습니다.

하지만 SOP에서는 다른 origin을 가지는 서버로도 리소스 요청을 보내기 위한 예외상황 몇 가지를 제시하는데 그 중에 하나가 "CORS 정책을 지킨 요청"인 경우입니다.

## 구동원리

1. 클라이언트에서 서버로 preflight request를 전송합니다.
2. 서버에서 요청처리가 가능한 클라이언트 origin 목록을 응답 헤더(`Access-Control-Allow-Origin`)에 명시하여 클라이언트로 전송합니다.
3. 클라이언트에서 요청 origin과 `Access-Control-Allow-Origin`에 명시된 origin을 비교했을 때 동일하지 않다면 응답 데이터의 status code가 `200 OK`여도 바로 폐기합니다.
4. 만일 origin 값이 동일하다면 원본 요청을 서버로 전송합니다.

여기서 서버가 아닌 클라이언트에서 두 origin을 비교하기 때문에 브라우저는 응답이 오고나서야 CORS 정책 위반 여부를 확인합니다.

따라서 클라이언트 쪽에 CORS 정책 위반이 일어났다고 해서 서버 쪽 로그를 분석해도 별다른 에러를 찾을 수 없습니다.

다르게 생각하자면 CORS 에러는 서버 간의 통신에서 발생하지 않는 것으로도 볼 수 있습니다.


## 적용되는 시나리오

### Preflight Request

![Preflight Request](/images/cors_preflight_request.png)

클라이언트에서 서버로 원본 요청을 전송하기 전에 [OPTIONS 메서드](../../http/http1/methods#options)를 사용한 예비 요청(preflight request)을 전송하여 서버가 허용하는 요청 설정(?)을 확인합니다.

#### 가능한 요청헤더들

- `Origin` : 요청을 전송하는 클라이언트의 origin
- `Access-Control-Request-Method` : 원본 요청에서 사용할 HTTP 메서드 타입
- `Access-Control-Request-Headers` : 원본 요청에서 사용할 HTTP 헤더(들)

#### 가능한 응답헤더들

- `Access-Control-Allow-Origin` : 서버의 리소스에 접근할 수 있는 클라이언트의 origin
- `Access-Control-Expose-Headers` : 클라이언트 단에서 JS API를 사용하여 접근할 수 있는 헤더(들)
- `Access-Control-Max-Age` : preflight request의 응답을 caching할 수 있는 최대 시간
- `Access-Control-Allow-Credentials` : 원본 요청에서 인증과 관련된 정보를 담을 수 있는지 여부
- `Access-Control-Allow-Methods` : 원본 요청에서 사용가능한 HTTP 메서드(들)
- `Access-Control-Allow-Headers` : 원본 요청에서 사용할 수 있는 헤더(들)

요청 메세지의 헤더나 일부 필드의 값에 따라서 응답이 동적으로 달라질 때는 [`Vary`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Vary) 헤더도 사용합니다.

### Simple Request

![Simple Request](/images/cors_simple_request.png)

예비요청 없이 클라이언트에서 원본 요청이 서버로 전송되면 서버의 `Access-Control-Allow-Origin`과 원본 요청의 `Origin`을 비교하여 CORS 정책 위반 여부를 검사하는 방식입니다.

간단하다는 장점이 있지만 [특정 조건(요청 메서드, 요청 헤더, Content-Type 등)을 만족하는 경우](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#simple_requests)에만 예비 요청을 생략할 수 있다는 까다로운 점이 있어서 실제 실무에서는 잘 사용되지 않습니다.

### Credentialed Request

기존의 preflight, simple 방식에 인증정보를 더 추가하여 다른 origin 간의 통신에서 보안을 강화하기 위한 방식입니다.

요청 메시지를 전송하기 전에 XMLHttpRequest나 Fetch API에서 `credential` 옵션을 설정하면 요청에 인증과 관련된 헤더(Cookie 같은)를 담을 수 있습니다.

`credential` 옵션으로 가능한 값으로 3가지가 있습니다.

- `same-origin(기본값)` : 같은 origin 간의 요청에만 인증정보를 담을 수 있다.
- `include` : 모든 요청에 인증정보를 담을 수 있다.
- `omit` : 모든 요청에 인증정보를 담지 않는다.

credential을 담은 요청 메시지를 전송하여 서버로부터 응답이 오면 브라우저는 응답 메시지로부터 다음 2가지를 확인합니다.

- `Access-Control-Allow-Origin` 헤더값이 "\*"가 아닌 명시적인 URL인가?
- `Access-Control-Allow-Credential` 헤더값이 `true`인가?

둘 중 하나라도 만족하지 않는다면 브라우저는 응답의 status code가 `200 OK`여도 바로 폐기하고 CORS 에러 메시지가 콘솔에 출력됩니다.

또한 응답 메시지에 `Set-Cookie` 헤더가 있다고 해도 브라우저에 쿠키가 저장되지 않습니다.

## 에러 핸들링

### 운영 중

API 서버 응답 메시지의 `Access-Control-Allow-Origin` 헤더에 알맞은 값을 셋팅하고 요청 헤더에도 적절한 `Origin` 값을 셋팅합니다. 그외의 다른 헤더들도 마찬가지입니다.

### 개발 중

webpack-dev-server을 사용하는 경우, 웹팩설정에 proxy 부분을 적절하게 셋팅하면 브라우저에서 요청을 전송했을 때 다른 `Origin`으로 교체하여 CORS 정책을 우회할 수 있습니다.

### 그 외 방법

SOP의 예외조항 중에 하나로 스크립트, 이미지, 스타일 시트의 소스 주소를 이용한 리소스 요청이 있습니다.

이 요청 메시지의 `Sec-Fetch-Mode` 헤더값은 `no-cors`가 되어 브라우저에서 응답 메시지의 CORS 정책 위반 여부를 검사하지 않습니다.

하지만 클라이언트 쪽에서 응답 데이터에 접근할 수 없다는 단점이 있습니다.

## 참고자료

[About CORS](https://evan-moon.github.io/2020/05/21/about-cors/)

[CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)