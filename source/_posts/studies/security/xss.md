---
title: XSS
date: 2024-11-03 18:49:27
categories:
  - Studies
  - Security
#tags:
---

## XSS란?

Cross-Site Scripting의 약자로, CSRF와 반대로 클라이언트가 특정 서버를 신뢰한다는 점을 노린 공격입니다.

클라이언트가 특정 서버 응답의 신뢰성을 의심할 수 없는 웹 취약점이 주원인입니다.

XSS 공격은 클라이언트가 서버부터 돌아온 응답에 내재된 악성 코드를 검증하지 않은 채 브라우저가 실행하여 쿠키, 세션 토큰 같은 개인정보를 탈취(Session Hijacking)하거나 사이트의 DOM 트리 구조를 망가뜨립니다.

악성 코드는 JS, HTML 등과 같이 브라우저에 의해서 실행되거나 파싱되는 언어로 작성됩니다.

## 공격유형

### Reflected(non-persistent) Attack

가장 흔한 공격형태로, 클라이언트가 전송한 요청이 그대로 응답 메시지로 전달되어 아무런 처리없이 렌더링되는 페이지에서 주로 발생합니다.

#### 예시

아래와 같은 요청 URL을 가지면 메시지가 그대로 `<p></p>` 안에 렌더링되는 페이지가 있다고 합시다.

```html
<!--https://insecure-website.com/status?message=All+is+well.-->
<p>Status: All is well.</p>
```

그렇다면 메시지로 `<script>...</script>`를 전달할 수 있지도 않을까요?

```html
<!--https://insecure-website.com/status?message=<script>/*+Bad+stuff+here...+*/</script>-->
<p>
  Status:
  <script>
    /* Bad stuff here... */
  </script>
</p>
```

그럼 공격자는 희생자에게 위와 같은 URL로 요청하도록 링크를 전달할 것이고, 사용자가 이 링크를 클릭하면 페이지 렌더링 중 악성 스크립트 파싱 및 실행으로 공격이 시작됩니다.

Chrome이나 Safari 등 현대 브라우저들은 Reflected XSS 공격에 대응하기 위한 보안기능이 내장되어 있습니다.

### Stored(persistent) Attack

주입된 코드를 DB에 영구적으로 저장하여 서버가 수신한 요청마다 악성 스크립트를 포함한 응답을 브라우저로 전송하는 방식입니다.

사용자의 입력값이 검증되지 않은 채 바로 DB에 저장하는 사이트에서 발생할 수 있습니다.

![Stored Attack](/images/xss_stored_attack.png)

### DOM-based Attack

Reflected XSS와 굉장히 유사하지만 악성 스크립트가 주입되는 시점이 다른 방식입니다.

Reflected XSS는 서버에서 요청 URL을 파싱하여 악성 스크립트가 주입된 페이지를 내려주지만 DOM-based XSS는 클라이언트가 페이지를 받은 뒤에 페이지 URL query나 fragment를 파싱하여 DOM에 악성 스크립트를 의도치않게 주입하고 실행합니다.

![DOM-based Attack](/images/dom_based_xss.png)

## XSS 예방하기

XSS 공격은 XSRF 공격과는 다르게 프런트 단에서 유효한 스크립트 코드를 판단하여 대응해야 합니다.

HTTP 응답이나 URL query, fragment로 전달된 악성 스크립트가 `.html`에 바로 삽입되는 것이 주원인이므로, 클라이언트에서 `<script></script>`를 비활성화시키면 문제를 해결할 수 있습니다.

### XSS 필터링

`<script></script>`의 태그로서 기능을 가지지 않도록 HTML context 상의 특수문자들을 [HTML Entity](https://developer.mozilla.org/en-US/docs/Glossary/Entity)로 인코딩합니다.

```html
<!-- before -->
<script></script>

<!-- after -->
&lt;script&gt; &lt;/script&gt;
```

HTML Entity는 prefix로 `&`, postfix로 `;`를 가지는 string으로, HTML reserved character 또는 non-visible character를 HTML 태그 아닌 텍스트로서 표시하거나 키보드에 없는 문자를 표시할 때 사용합니다(© ⇒ `&copy;`).

| 문자 | HTML entity |               설명               |
| :--: | :---------: | :------------------------------: |
|  &   |   `&amp;`   |        html entity의 시작        |
|  <   |   `&lt;`    |         html tag의 시작          |
|  >   |   `&gt;`    |          html tag의 끝           |
|  “   |  `&quot;`   | html tag의 attribute의 시작과 끝 |

### Unicode Escape
JS 코드 상에서 특수문자를 그저 문자로서, 또는 키보드로 표시할 수 없는 문자들을 사용하려면 escape 처리가 필요한데, back-slash(\\) 뒤에 대응되는 문자가 위치합니다.

여기서 escape 처리가 된 결과물을 escape sequence라고 합니다.

![Unicode Escape](/images/unicode_escape.png)

ES6에서 지원하는 escape 처리들은 다음과 같습니다.

- Hex escape(`\xHH`) : 2자리 16진수로 구성

```js
"\x7A" === "z"; // true
```

- Unicode escape(`\uHHHH`) : 4자리 16진수로 구성

```js
"\u007A" === "z"; // true
```

- Unicode code point escape(`\u{~}`) : 4자리 이상의 16진수로 구성

```js
"\u{7A}" === "z"; // true
```

여기서 escape 처리는 단순히 JS string이 아닌 변수명, 함수명 등의 코드 상에서 모든 문자들을 포괄해서 아래와 같은 코드도 작성할 수 있습니다.

```js
const az = 9;
console.log(az); // 9

console.log("\u{1F680}"); // 🚀
```

### 사용자가 입력한 데이터 검증하기

사용자로부터 전달된 input 데이터의 유효성을 검증하여 무효하다면 요청을 차단합니다.

여기서 유효성을 검증할 때는 [blacklisting보다는 whitelisting을 이용합니다](https://www.packetlabs.net/posts/blacklisting-whitelisting-greylisting/).

### 안전한 HTML 포스팅

블로그나 사이트에 code snippet으로 포스팅하는 경우가 흔한데 여기에 악성코드가 전달되지 않도록 관리해야 합니다.

### template engine, framework 설정

다수 프런트엔드 프레임워크에서 동적으로 DOM 트리를 구성할 수 있도록 template engine을 사용하는데, XSS 공격을 막을 수 있도록 적절한 설정이 필요합니다.

### CSP(Content Security Policy)

`Content-Security-Policy`라는 HTTP 응답 헤더에 값을 명시하여 브라우저가 받은 `index.html` 페이지로부터 요청할 수 있는 리소스의 URL의 종류 또는 해당 페이지가 iframe될 수 있는 parent page의 종류 등를 제한할 수 있습니다.

브라우저가 어떤 웹 페이지를 요청했는데 서버가 CSP header를 포함하여 운영지령(?)을 내리면 브라우저는 이 지령에 맞춰서 렌더링을 수행합니다.

## 참고자료

[XSS](https://owasp.org/www-community/attacks/xss/)

[Cross Site Scripting](https://portswigger.net/web-security/cross-site-scripting)

[XSS Attack](https://nordvpn.com/ko/blog/xss-attack/)

[DOM 기반 XSS(DOM based Cross Site Scripting) 공격과 방어](https://junhyunny.github.io/information/security/dom-based-cross-site-scripting/)

[Blacklisting, Whitelisting, GreyListing](https://www.packetlabs.net/posts/blacklisting-whitelisting-greylisting/)

[Content Security Policy](https://portswigger.net/web-security/cross-site-scripting/content-security-policy#mitigating-xss-attacks-using-csp)

[Unicode](https://exploringjs.com/es6/ch_unicode.html)