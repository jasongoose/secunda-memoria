---
title: 6-3장. 네트워크 계층(3계층)
date: 2024-10-31 23:32:11
categories:
  - Books
  - TCP/IP 완벽가이드
  - Chapter 6
#tags:
---
개별 2계층 네트워크(로컬 네트워크)들을 연결하여 원격 네트워크와 통신할 수 있는 인터네트워크를 형성하는 계층입니다.

이 계층에 속하는 프로토콜로 IP(Internet Protocol), ICMP(Internet Control Message, Protocol) 등이 있습니다.

### 논리적 주소지정

로컬 네트워크에 속하는 개별 장비는 물리적 위치에 상관없이 인터네트워크 상에서 고유한 id인 IP 주소를 가집니다.

송신장비는 패킷의 header에 목적지, 출발지 장비를 IP 주소로 지정합니다.

### 라우팅 네트워크

3계층 장비인 라우터가 임의의 패킷을 받으면 패킷 header에 명시된 **최종 목적지 주소**를 읽고 다음 경로를 결정하여 [재전송합니다](../chapter-5-3).

그래서 인터네트워크 상에서의 통신은 라우터에서 라우터(hop-to-hop)로의 통신으로도 이해할 수 있습니다.

라우터들은 트래픽을 효율적으로 보내기 위한 최적의 경로를 찾기 위해서 라우팅 프로토콜을 사용하여 서로 통신합니다.

### 패킷 캡슐화

4계층 PDU(3계층 SDU)를 인터페이스로 받아서 3계층 PDU인 "패킷"으로 캡슐화하여 2계층으로 전달합니다.

### 단편화와 재조합

송신장비에서 2계층 SDU 사이즈에 맞게 3계층 PDU(패킷)를 단편화하고 수신장비 3계층에서 재조합하여 원본 패킷을 복원합니다.

### 에러 처리와 진단

인터네트워크 상에서 연결된 라우터들은 네트워크나 장비 상태 정보를 교환할 수 있는 특수 프로토콜을 사용합니다.