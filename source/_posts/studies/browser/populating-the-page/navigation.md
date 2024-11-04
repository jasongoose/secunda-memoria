---
title: Navigation
date: 2024-11-02 00:05:54
categories:
  - Studies
  - Browser
  - Populating The Page
#tags:
---
웹 페이지를 화면에 표시하기 위한 첫 번째 단계로, 주소창에 URL을 입력하고 엔터 키를 눌러서 서버로 요청하면 응답이 돌아오는 과정이 바로 Navigation입니다.

## DNS Lookup

브라우저가 요청 URL에 대응되는 IP를 로컬 또는 원격 DNS 서버로부터 검색하는 과정입니다.

DNS Lookup은 한번도 방문하지 않은 페이지에 접속하려는 경우 실행됩니다.

추후에 동일한 서버로의 요청을 더 빠르게 처리할 수 있도록 해당 IP를 브라우저에서 일정 시간동안 캐싱할 수 있습니다.

하나의 페이지를 표시하기 위한 asset들이 동일한 서버에 위치하여 DNS Lookup이 한번만 일어나면 좋겠지만 만일 폰트파일, 이미지, 스크립트, 광고 등 데이터가 각기 다른 서버에 위치한다면 lookup이 여러 번 일어나야 합니다.

이와 같은 DNS Lookup에 의한 지연은 웹 성능에 영향을 주므로 최대한 줄여야 합니다.

## TCP Handshake

DNS Lookup으로 목적지 서버 IP를 알아내면 브라우저와 웹 서버 사이에서는 TCP 3-way HandShake를 통해서 connection을 생성합니다.

이 과정을 통해 2개의 통신주체(브라우저, 웹 서버)는 데이터를 주고받기 전에 서로 협상하여 TCP 통신 소켓의 설정을 결정합니다.

3-way HandShake라는 이름은 TCP 세션을 시작하기 전에 클라이언트와 서버 사이에서 3개의 메세지(SYN, SYN+ACK, ACK)를 주고받는다는 점에서 유래됩니다.

## TLS Negotiation

![TLS Negotiation](/images/tls_negotiation.jpg)

TCP HandShake를 거쳐서 connection을 생성한 뒤, 통신에 적용할 암호화 방식 및 인증수단을 정하는 HandShake를 거쳐야 합니다.

이 HandShake는 [TLS 프로토콜(구 SSL)](https://developer.mozilla.org/en-US/docs/Glossary/TLS)을 따르는데, 해당 프로토콜은 Internet Protocol Suite에서 최상위 Application Layer에 위치하거나 HTTPS 프로토콜의 Security Layer에 위치합니다.
