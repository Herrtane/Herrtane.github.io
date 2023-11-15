---
layout: post
title: <Network> 02. Background knowledge
date: 2020-09-04 13:29:23 +0900
category: Network
comments: true
---
이번 포스팅에서는 본격적으로 OSI 7Layer를 세부적으로 다뤄보기에 앞서, 네트워크에 있어서 필요한 기본적인 배경지식과 용어를 정리해보려고 한다.

## 배경지식 
<br/>

**프로토콜(protocol)** : 데이터를 송수신하는 과정을 주관하는 일련의 규칙들을 뜻한다. 예를 들어, 한국어를 하는 사람과 프랑스어를 하는 사람이 서로 통신하기 위해서는 공통된 언어가 필요한데, 영어가 이 역할을 하는 것과 유사하다.

<br/>

**Data Flow Direction** : 네트워크에서 정보를 주고 받는 방향별 명칭이 존재한다. 크게 3가지로 나뉜다.
1. Simplex : 한 방향으로만 진행되는 일방향 통신이다.
2. Half-duplex : 양방향으로 진행되기는 하지만, 동시에 진행되는 것이 아니라, 한 번 수신하면, 한 번 송신하는 식으로 주고 받는 형식이다.
3. Full-duplex : 실시간 양방향 통신이다.

<br/>

**Physical Structure** : 네트워크에서 기기 간의 연결 방식에 대한 명칭이 존재한다. 크게 2가지로 나뉜다.
1. Point-to-point : 두 기기가 서로 연결된 것을 의미한다.
2. Multipoint : 한 주축 케이블을 중심으로, 여러 기기가 쭉 연결된 것을 의미한다.

<br/>

**토폴로지(topology)** : 네트워크가 물리적으로 어떻게 배치되어 있는지를 나타내는 단어이다. 둘 이상의 장치들은 서로 **연결(link)**되어 있고, 둘 이상의 연결(link)들은 **토폴로지(topology)**를 형성한다. 크게 4종류로 나뉜다.
1. Star topology : 여러 기기가 하나의 hub에 연결되어 있는 구조이다. 어느 하나의 link가 끊어지더라도, 나머지 link들은 영향을 받지 않는다는 장점이 있지만, hub가 고장난다면 전체 통신이 망가질 수 있다는 단점이 존재한다. Tree topology는 Star topology의 응용된 버전이다.
2. Mesh topology : 모든 기기가 서로 연결되어 있는 구조이다. 가장 견고한 topology이고 보안이 보장되지만, 모든 기기마다 link를 걸어야 하기 때문에 경제적인 부담이 되는 단점이 있다.
3. Bus topology : 하나의 케이블에 각 기기가 연결된 multipoint형 구조이다. 설치가 쉽다는 장점이 있으나, 기기 수가 많아지면 충돌할 가능성이 생긴다.
4. Ring topology : 말 그대로 각 기기가 고리 구조로 연결되어 있다. 상대적으로 설치하기 쉽고 데이터를 안정적으로 전송하지만, ring의 일부가 고장나면 전체가 영향을 받기 쉽다.

<br/>

**인터넷(internetwork)** : LAN(Local Area Network), MAN(Metropolitan Area Network), WAN(Wide Area Network)이 복합적으로 모여서 인터넷을 형성한다.

<br/>

**성능(performance)** : 네트워크 통신에서 성능은 크게 두가지로 나뉜다. **대역폭(bandwidth)**은 link에서의 잠재적인 성능, 즉 네트워크에서 특정 시간 내에 전송될 수 있는 데이터의 최대 용량을 나타내고, 네트워크에서 초당 실제로 처리된 패킷의 양이 **출력(throughput)**이다.

<br/>

**Cyclic Redundancy Check(CRC)** : 네트워크에서 사용되는, 에러 검출 및 복구에 사용되는 코드이다.

<br/>

**Piggybacking** : ACK메시지를 보낼 때, 단순히 의미 없는 ACK메시지만 보내기 보다는, 이왕 보내는 김에 추가로 보낼 수 있는 메시지를 같이 뒤에 덧붙여서 보내는 것이 경제적이고 효율적인데, 이 과정을 의미한다.

<br/>
<br/>

## 마치며

다음 포스팅은, Physical Layer와 Data Link Layer에 대해 작성할 예정이다.
