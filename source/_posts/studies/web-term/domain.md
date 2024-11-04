---
title: Domain
date: 2024-11-03 23:09:13
categories:
  - Studies
  - Web Term
#tags:
---
## 구조

![Domain 구조](/images/domain_structure.png)

도메인은 서버 주소의 가독성을 높이기 위해서 사용하는 주소지정 방식으로, 보통 오른쪽에서 왼쪽 방향으로 읽습니다.

### TLD(Top Level Domain)

도메인의 가장 오른쪽에 위치하여 해당 도메인의 전반적인 운영목적을 보여줍니다.

`.com`, `.org`, `.net`과 같은 TLD는 아무 도메인에 붙여도 되지만 일부는 정해진 특수목적을 위해서 허가된 조직에서만 사용할 수 있습니다.

- 특정 언어로 서비스를 제공하거나 특정 나라에서 호스팅하는 경우 : `.us`, `.fr`, `.kr` 등
- 정부 부처에서 운영하는 서버인 경우 : `.gov`
- 교육기관에서 운영하는 서버인 경우 : `.edu`

### Label

TLD 왼쪽으로 붙은 문자열로 `.`으로 구분합니다.

대소문자를 구분하지 않고 최대 63자의 알파벳 + 0부터 9까지의 한자리 숫자 + `-` 또는 `_`를 포함할 수 있습니다.

도메인이 가질 수 있는 label의 개수는 제한되지 않습니다.

### SLD(Secondary Level Domain)

TLD 바로 왼쪽에 위치한 label을 가리킵니다.

## 관리

서버의 도메인은 한번 구매해서 계속 사용하는 것이 아닌 1년 단위로 사용기간을 정하여 대여비용을 지불하고 사용하는 방식으로, 사용기간이 지날 때마다 갱신해야 합니다.

registrar(레지스트라)라고 불리는 기관/회사에서 도메인 이름 등록과 관리를 담당합니다.

## DNS Refreshing

![DNS Refreshing](/images/dns_refreshing.jpeg)

전세계적으로 퍼져 있는 DNS 서버들은 계층구조의 형태를 가지고 각자 담당하는 도메인에 속하는 서브 도메인들의 IP 주소들을 관리합니다.

만일 registrar에서 특정 도메인에 관한 정보를 생성하거나 수정하면 이 정보는 해당 도메인의 정보를 가지고 있는 다른 도메인들에게 반영이 되도록 업데이트가 이루어집니다.

물론 최신 상태로 업데이트하기까지 시간은 좀 걸립니다.

## DNS Process

![DNS Process](/images/dns_process.png)

브라우저에서 도메인 주소를 입력하고 엔터키를 누르면 브라우저는 먼저 local DNS cache을 먼저 확인해봅니다.

만일 대응되는 IP 주소가 없다면 root level DNS에서부터 시작하여 트리를 타고 내려오면서 이 도메인을 담당하는 DNS 서버의 주소를 알아냅니다.

해당 DNS 서버에서 도메인에 해당하는 IP주소를 알아내어 브라우저로 전달하면 세션연결을 시작합니다.

## Page vs. Site

### Page

![Web Page](/images/web_page.jpg)

브라우저에 의해서 화면에 그려지는 하나의 HTML 문서로 스타일(`.css`), 스크립트(`.js`), 미디어(이미지, 영상, 음성 등) 등 다양한 리소스들을 포함합니다.

웹 페이지를 보려면 브라우저 주소창에 해당 페이지의 URL을 입력하거나 목적지 페이지로 이동하는 링크를 클릭하면 됩니다.

### Site

![Website](/images/web_site.jpg)

서로 연결된 웹 페이지들과 관련 리소스들의 집합으로 고유한 도메인을 가집니다.

브라우저 주소창에 도메인을 입력하면 웹 사이트에 접근할 수 있습니다.

## Web Server

1개 이상의 웹 사이트들을 호스팅하는 물리적인 하드웨어와 소프트웨어를 동시에 가리킵니다.

![Web Server](/images/web_server.png)

Hosting이란 임의의 웹 사이트로 사용자의 요청이 들어올 때마다 요청한 웹 페이지와 관련 리소스를 브라우저로 제공한다는 뜻입니다.

웹 서버는 웹 사이트를 호스팅하고, 웹 사이트의 호스트는 웹 서버입니다.

### HW

물리적인 하드웨어로서 서버 소프트웨어를 실행하고 웹 사이트의 구성 파일들(`.html`, `.css`, `.js`, image 등)을 저장하는 컴퓨터입니다.

인터넷과 연결되어 웹 상에 연결된 다른 기기들과 데이터를 주고받을 수 있습니다.

### SW

호스팅하고 있는 파일들을 어떻게 사용자들에게 제공할지를 제어하는 소프트웨어입니다.

클라이언트가 웹 서버에서 호스팅되는 파일들을 얻고자할 때는 웹 서버로 HTTP 요청을 전송합니다.

그 요청이 물리적인 웹 서버에 도착하면 내부에 위치한 HTTP 서버 소프트웨어가 요청을 분석하고 적절하게 응답합니다.

## Search Engine

사용자가 다른 사이트의 페이지를 찾을 수 있도록 도와주는 사이트입니다.

브라우저는 웹 페이지를 요청하고 보여주는 기반 소프트웨어인 반면, 검색 엔진은 브라우저 위에서 동작하는 서비스입니다.

![Search Engine](/images/search_engine.jpg)

## 참고자료

[What is the difference between webpage, website, web server, and search engine?](https://developer.mozilla.org/en-US/docs/Learn/Common_questions/Web_mechanics/Pages_sites_servers_and_search_engines)