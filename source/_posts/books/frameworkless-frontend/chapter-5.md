---
title: 5장. HTTP 요청
date: 2024-11-10 20:57:31
categories:
  - Books
  - 프레임워크 없는 프론트엔드
#tags:
---
## 간단한 역사: AJAX의 탄생

- 1999년 이전에는 서버에서 데이터를 가져올 필요가 있는 경우, 전체 페이지를 다시 로드해야 했습니다.
- 이후 최초 페이지 로드 후 필요한 데이터만 서버에서 로드하는 새로운 기술을 사용하기 시작했습니다.
- 제임스 가레트는 2005년에 자신의 블로그에 이 기술을 "Asynchronous JavaScript  and XML"의 약어인 "AJAX"로 명명했습니다.
- W3C는 2006년 AJAX의 핵심인 `XMLHttpRequest` 객체의 표준 규격 초안을 작성했습니다.
  - 당시 서버에서 내려주는 데이터는 XML 형식으로 전달했습니다.

## todo 리스트 REST 서버

### REST

- 도메인을 리소스로 분할해야 하며 각 리소스는 특정 URI로 접근해 읽거나 조작할 수 있어야 합니다.
- 리소스를 추가, 삭제, 업데이트하고자 동일한 URI가 사용되지만 HTTP 메서드는 달라집니다.

## 코드 예제

### 기본 구조

- 컨트롤러에서 HTTP 클라이언트를 직접 사용하는 대신 모델 객체에 래핑하면 아래와 같은 장점을 가집니다.
  - 테스트 가능성
    - 테스트 코드에서 정적 데이터 세트를 반환하는 mocking 대상이 될 수 있습니다.
    - 컨트롤러를 독립적으로 테스트할 수 있습니다.
  - 가독성
    - 모델 객체는 코드를 좀 더 명확하게 만듭니다.
- HTTP 클라이언트를 사용하기 위한 공개 API는 아래와 같이 만들 수 있습니다.
  - `http[verb](url, config)`: GET, DELELTE
  - `http[verb](url, body, config)`: POST, PUT, PATCH
  - `http(url, verb, body, config)`

### XMLHttpRequest

- W3C가 비동기 HTTP 요청의 표준 방법을 정의한 첫 번째 시도였습니다.
- 2006년 정의된 API로 콜백을 기반으로 합니다.
- HTTP 클라이언트의 공개 API는 Promise를 기반으로 하는데, `XMLHttpRequest` 요청을 새로운 Promise 객체로 묶습니다.

```js
const request = (params) => {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();

	// 특정 URL로 요청을 초기화
    xhr.open(params.method ?? 'GET', params.url);

	// 요청 헤더 설정, 타임아웃 등을 구성
    setHeaders(xhr, params.headers);

	// 요청 전송
    xhr.send(JSON.stringify(params.body));

	// 요청이 오류로 끝나는 경우
    xhr.onerror = () => {
	  reject(new Error('HTTP Error'));
    }

	// 요청이 타임아웃으로 끝나는 경우
	xhr.ontimeout = () => {
	  reject(new Error('Timeout Error'));
	}

	// 요청이 성공적으로 끝나는 경우
	xhr.onload = () => {
	  resolve(parseResponse(xhr));
	}
  });
}
```

### Fetch

- 원격 리소스에 접근하고자 만들어진 새로운 API입니다.
- `Request`, `Response` 같은 네트워크 객체에 대한 표준 정의를 제공하는 것이 목적입니다.
- `ServiceWorker`, `Cache` 등 다른 API와 상호 운용할 수 있습니다.

```js
const parseResponse = async (response) => {
  let data;

  if (response.status !== 204) {
    data = await response.json();
  }

  return {
    status,
    data,
  }
}

const request = async (params) => {
  const config = {
    method: params.method ?? 'GET',
    headers: new window.Headers(params.headers ?? {}),
    // contentType: ''
  };

  if (body) {
    config.body = JSON.stringify(body);
  }

  const response = await window.fetch(url, config);

  return parseResponse(response);
}
```
### Axios

- [HTTP 클라이언트 라이브러리](https://axios-http.com/docs/intro)

```js
const request = async (params) => {
  const config = {
    url: parmas.url,
    method: params.method ?? 'GET',
    headers: params.headers ?? {},
    data: params.body,
  };

  return axios(config);
}
```

### 아키텍처 검토

- `XMLHttpRequest`, `Fetch`, `axios` 버전의 개별 API는 동일한 공용 API를 가집니다.
- 이런 아키텍처 특성으로 인해 최소한의 노력으로 HTTP 요청에 사용되는 라이브러리를 변경할 수 있습니다.
- 자바스크립트는 동적 타입 언어지만 모든 클라이언트는 HTTP 클라이언트 인터페이스를 구현합니다.

> 구현이 아닌 인터페이스로 프로그래밍하라
> - Gang of Four

## 적합한 HTTP API를 선택하는 방법

### 호환성

- IE 지원이 중요하다면 `axios`, `XMLHttpRequest`를 사용합니다.
- IE 지원을 중단할 계획이 있다면 `Fetch` API에 polyfill을 적용하여 사용합니다.

### 휴대성

- `XMLHttpRequest`, `Fetch` API는 모두 브라우저에서만 동작합니다.
- 브라우저 외 런타임 환경에서 코드를 실행해야 하는 경우 `axios`가 매우 좋은 솔루션입니다.

### 발전성

- `Fetch` API에서 제공하는 `Request`, `Response` 같은 네트워크 관련 객체의 표준 정의는 `ServiceWorker` API나 `Cache` API와 잘 맞기 때문에 코드베이스를 빠르게 발전시킬 때 유용합니다.

### 보안

- axios에는 XSS, XSRF에 대한 보호 시스템이 내장되어 있어서 유용합니다.

### 학습 곡선

- Promise 기반의 `axios`, `Fetch` API가 초보자들이 이해하기 쉽습니다.