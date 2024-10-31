---
title: 13-4장. 프록시 ARP
date: 2024-10-31 23:33:52
categories:
  - Books
  - TCP/IP 완벽가이드
  - Chapter 13
#tags:
---
동일한 네트워크지만 서로 다른 서브네트워크(로컬 네트워크, 물리 네트워크)에 위치한 장비간의 ARP 주소결정에는 라우터가 중간 Proxy 역할을 합니다.

![ARP Proxy](/images/arp_proxy.png)

장비 A가 ARP 요청 메시지를 broadcast를 해도 다른 물리 네트워크에 있는 장비 B의 2계층 주소를 알 수 없습니다.

그래서 라우터가 다른 물리 네트워크에 있는 장비 B 대신에 자신의 2계층 주소를 ARP 응답 메시지를 장비 A로 unicast합니다.
