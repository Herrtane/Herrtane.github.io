---
layout: post
title: <Network> 03. Data link layer (1) - MAC
date: 2020-09-05 12:10:23 +0900
category: Network
comments: true
---
이번 포스팅에서는 본격적으로 OSI 7계층 중 두 번째 계층인 Data link layer에 대해서 일부를 다루어 보겠다. 

사실 첫 번째 계층인 Physical layer에 대해서는 지난 배경지식 포스팅에서 다룬 것이나 마찬가지이기 때문에, 바로 Data link layer부터 다루도록 한다.


## Data link layer의 첫 번째 역할 : Media Access Control(MAC)

Data link layer에 대한 개념은 Network 카테고리의 첫 번째 포스팅에서 다루었었다. 이번 포스팅에서는 이어서 Data link layer의 첫 번째 역할인 Media Access Control에 대해서 이야기해보려고 한다.

<br/>

하나의 네트워크에서 오직 두 개의 기기만 서로 link를 형성하여 통신을 한다면 복잡하지 않겠지만, 만약 세 개 이상의 기기가 통신을 한다면 어떨까? 만약 세 기기가 동시에 신호를 보내려다가 신호끼리 충돌한다면 이는 어떻게 해결할까?

<br/>

이런 문제를 해결하기 위해서 고안된 것이 바로 Media Access Control이다. 그리고 Access 방식에 따라서 다시 크게 3가지 방식으로 나뉜다. 아래의 그림을 참고하자.

![mac_overview]({{site.url}}/img/mac_overview.jpg)

각각의 방식에 대해서 하나씩 살펴보자.

<br/>

## Random-access protocols

이 방식에서는 통신하는 그 어떤 기기도 통신하는 과정에서 더 많은 권리를 가지지 않는다. 오직 상호간의 경쟁에 기반(contention based access)하여 통신을 진행한다. 그렇기에 random이라는 명칭이 붙었다고 볼 수 있다. 

<br/>

신호가 서로 충돌하는 것을 막기 위해서, random-access protocol에서는 크게 3가지 구체적인 방법이 고안되었다.

1. ALOHA : 가장 오래된 다중접속 방식이다. 충돌방지를 위해 임의로 대기했다(backoff time)가 신호를 송신한다.
![aloha]({{site.url}}/img/aloha.jpg)
2. CSMA/CD : Carrier Sense (회선이 사용중인지 확인) Multiple Access (다중 접근) / Collision Detection으로서, 만약 같은 회선에 존재하는 서로 다른 컴퓨터가 동시에 신호를 보내게 되면 충돌이 일어날 수 있는데, CSMA/CD에서는 충돌을 미리 감지하고 jamming signal을 다른 기기에 보내어서 전송을 일정 시간 중단시킴으로써 회선이 낭비되지 않도록 한다. 
![csma_cd]({{site.url}}/img/csma_cd.jpg)
3. CSMA/CA : 무선 네트워크에서는 CSMA/CD방식을 적용하기가 어렵다. 그래서 이 방식이 고안되었는데, 충돌을 감지하는 것이 아니라, 미리 충돌을 피하는 방식이다. 방식은 이러하다.
> 회선에 아무런 신호가 잡히지 않는 idle한 상태에서, 한 기기가 통신을 하기 전에 IFS(Interframe Space)만큼 기다리면서 혹시 원거리의 다른 기기의 회선 사용을 예방한다. 그 뒤에도 idle한 상태라면, contention window에서 임의의 slot time을 배정받고 다시 한번 기다린다. 그 후, 최종적으로 idle한 상태를 확인하면 마침내 신호를 전송한 뒤에 수신측으로부터 ACK메시지를 기다린다. ACK메시지가 잘 도착하면 성공적으로 송수신이 이루어진 것이다.

![csma_ca]({{site.url}}/img/csma_ca.jpg)

세부적인 용어에 대한 설명은 우선 생략하고, 추후에 기회가 되면 추가하겠다.

<br/>

## Controlled-access protocols

이 방식에서는 어느 한 기기가 우선권을 가지고 다른 기기들의 통신을 관리하기 때문에, controlled라는 명칭이 붙었다고 볼 수 있다.

이 방식에서는 Reservation, Polling, Token passing의 방법이 존재하지만, 세부적인 설명은 우선 생략하도록 하겠다.

<br/>

## Channelization protocols

이 방식에서는 신호나 회선 자체를 기술적으로 나누어서 통신을 하게끔 한다. 크게 어려운 개념은 아니므로 간단하게 짚고 넘어가겠다.

1. FDMA(Frequency-Division Multiple Access) : 신호를 보내는 시간에 구애받지 않도록, 각각 기기마다 서로 다른 주파수 영역을 분배하여 그 영역을 통해서 신호를 자유롭게 보내는 방식이다.
2. TDMA(Time-Division Multiple Access) : 신호를 보내는 시간을 세분화해서, 각각 기기마다 시간을 배분받아서 그 시간에만 통신망에 신호를 보내도록 하는 방식이다.
3. CDMA(Code-Division Multiple Access) : 각각 기기마다 신호를 다르게 암호화하고 복호화하여서, 신호의 방식을 달리 하여서 통신망을 분배하는 방식이다.

나는 개인적으로 CDMA 방식에 대해서는 추상적이라는 느낌을 받았다. 이후 이 방식에 대해서 접할 일이 생기면, 조금 더 자세하게 공부해보려고 한다.

<br/>
<br/>

## 마치며

수업 시간에 다루었던 모든 내용을 다 서술하자니 글도 길어지고, 포스팅하는 텐션도 떨어지는 것 같아서, 어느 정도 반드시 기억해야 되는 내용을 중심으로 포스팅해나가고 있다. 혹여나 더 보충해야될 부분이 있다면, 우선 중요한 전체 내용에 대해서 모두 포스팅 한 이후에 수정해나가도록 하겠다.

<br/>
다음 포스팅은 Data link layer의 나머지 역할에 대해서 다루어볼 예정이다.
