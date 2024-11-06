---
title: 13-2장. TCP/IP 주소 결정 프로토콜
date: 2024-10-31 23:33:43
categories:
  - Books
  - TCP/IP 완벽 가이드
  - Chapter 13
#tags:
---
3계층 프로토콜 IP와 이더넷을 비롯한 2계층 프로토콜간의 동적 주소 결정을 위해서 개발된 ARP(Address Resolution Protocol)입니다.

## ARP 메시지 포맷

![ARP Format](/images/arp_format.png)

### Harware Type

2계층에서 사용하는 프로토콜 유형이자 주소지정 방식을 명시하는 필드입니다.

![HRD](/images/hrd.png)

### Protocol Type

3계층에서 사용하는 프로토콜 유형이자 주소지정 방식을 명시하는 필드입니다.

### HLN

2계층 주소의 바이트 단위 길이를 명시하는 필드입니다.

이더넷의 경우, HLN은 48비트이므로 6을 값으로 가집니다.

### PLN

3계층 주소의 바이트 단위 길이를 명시하는 필드입니다.

IPv4의 경우, PLN은 32비트이므로 4를 값으로 가집니다.

### OpCode

ARP 메시지의 유형을 지정하는 필드입니다.

![ARP OpCode](/images/arp_opcode.png)

### SHA

송신장비의 2계층 주소가 위치하는 필드입니다.

### SPA

송신장비의 3계층 주소가 위치하는 필드입니다.

### THA

수신장비의 2계층 주소가 위치하는 필드입니다.

### TPA

수신장비의 3계층 주소가 위치하는 필드입니다.

## ARP 주소 명세와 일반 운영

![ARP Operation](/images/arp_operation.png)

1. 출발지 장비가 캐시를 검사
2. 캐시되지 않은 경우, 출발지 장비가 ARP 요청 메시지를 생성 

   - 요청 메시지 필드값
     - SHA : 출발지 장비의 2계층 주소
     - SPA : 출발지 장비의 3계층 주소
     - THA : 빈 값
     - TPA : 목적지 장비의 3계층 주소

3. 출발지 장비가 ARP 요청 메시지를 로컬 네트워크에서 broadcast
4. 로컬 장비들이 ARP 요청 메시지를 처리
5. 목적지 장비가 ARP 응답 메시지를 생성

   - 응답 메시지 필드값
     - SHA : 목적지 장비의 2계층 주소
     - SPA : 목적지 장비의 3계층 주소
     - THA : 요청 메시지의 SHA
     - TPA : 요청 메시지의 THA

6. 목적지 장비가 ARP 캐시를 갱신
7. 목적지 장비가 ARP 응답 메시지를 출발지 장비로 unicast
8. 출발지 장비가 ARP 응답 메시지를 처리
9. 출발지 장비가 ARP 캐시를 갱신
