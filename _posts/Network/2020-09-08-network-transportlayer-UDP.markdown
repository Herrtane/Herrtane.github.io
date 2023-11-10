---
layout: post
title: <Network> 06. Transport layer (1) - UDP
date: 2020-09-08 23:10:23 +0900
category: Network
comments: true
---
이번 포스팅에서는 Transport layer에 대해서, 특히 UDP에 대해서 다루어 보겠다.

## 복습

그동안 포스팅했던 내용을 복습하면서 OSI Layer의 전체적인 구조와 흐름을 파악해보자.

- Data link layer는 이웃한 **node**간의 **frame** 전송을 책임지는 역할을 한다. 이 계층에서는 **node-to-node** 전달이 이루어진다. 그리고 통신에 사용되는 주소는 **MAC** 주소이다.
- Network layer는 두 **host**간의 **datagram** 전송을 책임지는 역할을 한다. 이 계층에서는 **host-to-host** 전달이 이루어진다. 그리고 통신에 사용되는 주소는 **IP** 주소이다.
- 다시 말해, node가 3개 이상일 때를 **network**라 하고, 수 많은 node 사이에서 송신측과 수신측의 통신을 연결해주는 것이 network layer가 하는 일이다. network에서는 node가 아니라 host라고 한다.

<br/>

## Transport layer의 특징과 역할

그렇다면 Transport layer는 어떤 역할을 할까? 위의 두 layer의 연장선이다. 두 **process**간의 **packet** 전송을 책임지는 역할을 한다. 그렇기때문에, **process-to-process** 전달이 이루어진다. **client**와 **server** 패러다임을 사용하는 계층이며, 나의 쪽을 **localhost**, 상대편 쪽을 **remote host** 라고 부른다. 또한, 통신에 사용되는 주소는 **Port** 숫자이며, 여러개의 프로세스중에 선택하기 위한 주소이다.

![transport_layer]({{site.url}}/img/transport_layer.jpg)

지금까지의 layer들의 특징과 과정을 종합해놓은 좋은 그림이다.

<br/>

IP주소와 Port숫자를 합친 주소를 **socket address**라고 한다.

<br/>

## Connectionless VS Connection-Oriented

이 부분은 앞서 Network layer에서 datagram과 virtual-circuit 부분을 다룰 때 설명했다. 이 방식이 본격적으로 적용되는 곳이 바로 Transport layer의 UDP와 TCP이다. 하나씩 살펴보자.

1. Connectionless service : 각각의 패킷들이 넘버링되어있지 않고, 통신하면서 ACK메시지 또한 존재하지 않는다. 보통 빠르게 전송하는 것을 목표로 하기 때문에, flow와 error control을 깊이 하지 않거나 아예 하지 않는다. 따라서 Unreliable하기도 하다. 대표적으로 UDP가 있다.
2. Connection-Oriented service : 우선 송신측과 수신측에 먼저 연결이 설정되고 난 뒤에, 데이터가 전송된다. 이후 연결이 해제된다. 보통 더 신뢰성을 높이기 위해 느리고 복잡한 서비스를 채택하고, 따라서 reliable하다. 대표적으로 TCP가 있다.

참고로, Network layer에서 flow와 error control을 하긴 하지만, 충분한 신뢰성을 가지지 않기 때문에, Transport layer에서 이를 보강하여 확실한 reliability를 가진다. 그렇다면, 지금부터 UDP와 TCP를 하나씩 살펴보자.

<br/>

### 1. User Datagram Protocol (UDP)

앞서 UDP가 connectionless하고 unreliable하다는 것을 충분히 살펴보았다. 강조하자면, UDP는 빠른 통신을 지향한다. 특히, 크기가 작은 메시지를 전송할 때, 굳이 송신측과 수신측 사이에 연결을 설정하고 하는 복잡한 과정을 해야할까? UDP는 이럴때 강점을 나타낸다. UDP의 큰 특징을 정리하면 아래와 같다.

1. Connection service (Independent datagram)
2. No flow and error control
3. Bulk data를 sending하는 데에는 부적합 ex) FTP
4. Simple data를 sending하는 데에 적합, Routing Information Protocol에 사용되기도 함
5. Connectionless 하므로 송신측에서는 메시지가 소실되었거나 중복되었는지 알지 못하며, 수신단은 오로지 **checksum**을 통해서 메시지의 오류 여부를 알 수 있음. 만약 checksum에 문제가 있다면, 그 메시지는 그냥 버려짐.

![UDP]({{site.url}}/img/UDP.jpg)

위의 그림은 UDP의 통신 과정을 도식화한 것이다. 이후 살펴볼 TCP보다 훨씬 간단한 과정이며, 직관적으로 이해하기 쉽다.

<br/>

### 2. Transmission Control Protocol (TCP)

TCP는 UDP에 비해서 설명할 것이 많으므로, 별도의 포스팅으로 진행하려고 한다.



<br/>
<br/>

## 마치며

다음 포스팅은 Transport layer의 나머지 부분인 TCP에 대해서 다루어볼 예정이다.
