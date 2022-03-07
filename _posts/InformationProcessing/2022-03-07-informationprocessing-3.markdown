---
layout: post
title: <정보처리기사> 03. 세부용어/프로토콜/보안
date: 2022-03-07 14:41:23 +0900
category: InformationProcessing
comments: true
---

## 응용 계층의 주요 프로토콜

- HTTP (Hypertext Transfer Protocol) : HTML 문서를 송,수신하기 위한 표준 프로토콜.
- FTP (File Transfer Protocol) : 파일 전송 프로토콜.
- TELNET : 원격 가상 터미널 기능 수행 프로토콜.
- SMTP (Simple Mail Transfer Protocol) : 전자 우편을 교환하는 서비스.
- SNMP (Simple Network Management Protocol) : TCP/IP의 네트워크 관리 프로토콜.
- DNS (Domain Name System) : 도메인 네임을 IP 주소로 매핑하는 시스템.

## 전송 계층의 주요 프로토콜

- TCP (Transmission Control Protocol)
- UDP (User Datagram Protocol)
- RTCP (Real-Time Control Protocol) : 패킷의 전송 품질을 제어하기 위한 프로토콜로, 세션 각 참여자들에게 주기적으로 제어 정보 전송. 데이터 패킷과 제어 패킷의 Mux 제공.

## 인터넷 계층의 주요 프로토콜

- IP (Internet Protocol) : Datagram 방식
    - IPv6 : 128비트의 주소 사용, 인증성, 기밀성, 데이터 무결성 지원으로 보안 강화. Unicast, Multicast, Anycast 지원. (IPv4는 Anycast 대신 Broadcast)
    - Subnetting : 할당된 네트워크 주소를 다시 여러 개의 작은 네트워크로 나누어 사용하는 것. 4바이트의 IP주소를 네트워크 주소와 호스트 주소로 나누기 위해 Subnet Mask 사용. 203.241.132.82/27의 Subnet Mask는 255.255.255.224임. (자리당 8비트를 차지하므로 앞에서부터 27비트를 1로 처리하면 됨.)
    - A Class : 국가나 대형 통신망에 사용. 맨 앞 비트가 0인 Subnet Mask (0~127).
    - B Class : 중대형 통신망에 사용. 맨 앞 비트가 10인 Subnet Mask (128~191).
    - C Class : 소규모 통신망에 사용. 맨 앞 비트가 110인 Subnet Mask (192~223).  
    - D Class : 멀티캐스트용으로 사용. 맨 앞 비트가 1110인 Subnet Mask (224~239).
    - E Class : 실험적 주소이며 공용되지 않음 (240~255).
    - 예를 들면, 200.168.30.1은 C Class에 속한 IP 주소.
- ICMP (Internet Control Message Protocol) : Error, Ping, Traceroute 등 제어 메시지 관리, 전달, 진단. 즉, IP 패킷을 처리할 때 발생되는 문제를 알려줌.
- IGMP (Internet Group Management Protocol) : 멀티캐스트 그룹을 유지하고 멀티캐스트 실시간 전송을 위해 사용.
- ARP (Address Resolution Protocol) : IP주소를 MAC주소로.
- RARP (Reverse ARP) : MAC주소를 IP주소로.

## 라우팅 경로 제어 프로토콜

- RIP (Routing Information Protocol) : 소규모, IGP, 거리벡터 프로토콜. 최대 hop수를 15hop 이하로 제한.
- OSPF (Open Shortest Path First) : 대규모, IGP, 링크상태 알고리즘. RIP 단점 개선. Daijkstra 알고리즘 기반으로 최단 경로 설정. 멀티캐스팅 지원.
- BGP (Border Gateway Protocol) : EGP 단점 보완. 초기 BGP 라우터 연결 시 라우팅 테이블 교환.

## 네트워크 액세스 계층의 주요 프로토콜

- IEEE 802.3 : Ethernet. CSMA/CD 방식의 LAN.
- IEEE 802.4 : Token Bus.
- IEEE 802.5 : Token Ring.
- IEEE 802.11 : 무선 LAN. CSMA/CA 방식의 LAN.
- IEEE 802.15 : Bluetooth.
- HDLC (High-level DLC) : 비트 프레임 방식 (문자X)의 DLC 프로토콜. 3종류의 호스트 존재.
    - 컴퓨터가 1:1, 1:N로 연결된 환경에 데이터의 송수신 기능을 제공.
    - Half-Duplex, Full-Duplex 통신을 모두 지원.
    - 에러 제어를 위해 Go-Back-N ARQ 사용.
    - 흐름 제어를 위해 Sliding Window 방식을 사용.
    - 주국 (Primary Station) : 명령을 전송하는 호스트.
    - 종국 (Secondary Station) : 명령에 대한 응답을 회신하는 호스트.
    - 혼합국 (Combined Station) : 주국과 종국 기능을 모두 지닌 호스트.
- X25 : 패킷 교환망을 통해 패킷을 원활히 전달하기 위한 통신 프로토콜. TCP/IP보다 느리지만, 보안성은 더 뛰어남. 그러나 TCP/IP보다 사용자가 매우 적음.
    - 물리, 프레임, 패킷 계층으로 나뉨.
    - DTE (Data Terminal Equipment)와 DCE (Data Circuit-termination Equipment)간의 인터페이스 제공.
- RS-232C : 공중 전화 교환망 (PSTN)을 통한 인터페이스 제공.