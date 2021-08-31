---
layout: post
title: <TCP/IP Socket> 04. socket()에 대하여
date: 2021-08-31 20:48:23 +0900
category: TCP/IP_Socket
comments: true
---

## socket()의 인자들에 대한 설명

```c
#include <sys/socket.h>

int socket(int domain, int type, int protocol);
```

1. 첫번째 인자 : 소켓이 사용할 프로토콜 체계(Protocol Family) 정보 전달
    - 프로토콜 체계(Protocol Family) : PF_INET(IPv4), PF_INET6(IPv6) 프로토콜 체계 등이 있다.
2. 두번째 인자 : 소켓의 데이터 전송방식에 대한 정보 전달. **컴퓨터 네트워크 Network layer에서 배웠던 Virtual ciruit과 Datagram 방식이 연상된다.**
    1. 연결지향형 소켓(SOCK_STREAM)
        - 중간에 데이터가 소멸되지 않고 목적지까지 전송됨.
        - 전송 순서대로 데이터가 수신됨.
        - 전송되는 데이터의 경계가 존재하지 않음.
    2. 비 연결지향형 소켓(SOCK_DGRAM)
        - 전송된 순서에 상관없이 가장 빠른 전송을 지향.
        - 전송된 데이터는 손실의 우려가 있고, 파손의 우려가 있음.
        - 전송되는 데이터의 경계가 존재.
        - 한번에 전송할 수 있는 데이터의 크기가 제한됨. 
3. 세번째 인자 : 두 컴퓨터 간 통신에 사용되는 프로토콜 정보 전달.
    - IPPROTO_TCP, IPPROTO_UDP가 주로 사용됨.

## 마치며

확실히 컴퓨터 네트워크 전공과목과 내용의 접점이 많이 존재한다는 것을 느낀다.