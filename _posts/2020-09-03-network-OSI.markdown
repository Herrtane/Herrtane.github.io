---
layout: post
title: OSI 7Layer
date: 2020-09-03 23:09:23 +0900
category: Network
comments: true
---
지난 학기, 그러니까 3학년 1학기에 컴퓨터 네트워크 과목에서 배웠던 내용이 중요하다고 판단하여서,
이번 기회에 복습하는 겸 해서 컴퓨터 네트워크에 대한 중요한 이론을 차차 정리해나가려고 한다.
배웠던 기억과 필기에 의존하는 만큼, 약간의 오류가 있을 수 있다.
<br/>
<br/>

## OSI 7Layer란?  

컴퓨터 네트워크가 점점 발전하면서, 네트워크 통신 기능을 수행하는 기기를 만드는 회사마다 네트워크 통신에 대한 기준이 제각각 생겨났었는데, 서로 다른 기준을 하나로 통일해서 네트워크 통신을 원활히 하기 위해서 고안된 계층 구조가 바로 **OSI 7계층(Open System Interconnection)** 구조이다.
<br/>
<br/>
각 계층은 이렇게 구성되어있다.

- Layer 7 : Application Layer
- Layer 6 : Presentation Layer
- Layer 5 : Session Layer
- Layer 4 : Transport Layer
- Layer 3 : Network Layer
- Layer 2 : Data Link Layer
- Layer 1 : Physical Layer

낮은 계층일수록 물리적, 기계적이고, 높은 계층일수록 사용자와 가까운 특징을 가진다.
<br/>
<br/>

## OSI 7Layer의 구조 및 특징

그렇다면 각 계층은 어떤 특징을 가지고 있을까?

1. Physical Layer : 물리적인 통신을 담당한다. Ethernet, Wifi 처럼 직접 통신 신호를 주고 받게끔 하는 역할을 한다. 눈에 보이는 인터넷 선으로 생각하면 쉽다.
2. Data Link Layer : Physical Layer에서 통신을 주고 받으면서 발생할 수 있는 에러나 누락을 검출하고 수정하는 역할을 한다. 크게 두가지 역할로 나뉘는데, 오류를 검출하고 수정하는 Data Link Control과, 다수의 노드(각 사람이라고 생각하면 된다)끼리 통신할 때 통신 질서와 접근을 조절해주는 Multiple Access Control이 그것이다.
3. Network Layer : 통신하려는 노드가 많아지고 거리가 멀어질수록 Physical Layer가 감당하기 힘들 수 있다. Network Layer는 노드와 노드간 통신을 넘어서서, 복잡한 Network 안에서도 송신하는 주체와 수신하는 주체간의 통신이 원활히 이루어지도록 하는 역할을 맡는다. 주로 **IP주소**를 이용하여 이 과정을 진행한다.
4. Transport Layer : Network Layer는 혹여나 에러가 발생하거나 흐름이 이상해져도 이를 검출하고 수정할 능력이 부족하다. Transport Layer는 이를 담당하여 통신의 신뢰성을 높인다. 뿐만 아니라, Network Layer를 통해 올바른 IP주소로 통신이 도달했다면, 해당 컴퓨터 내에서 어떤 포트를 통과해야 하는지를 정해주는 역할도 한다. 참고로, IP주소와 포트주소를 하나로 묶어서 만든 것이 바로 **소켓**이다. (포트와 관련된 부분은 추후에 설명하겠다.) **TCP 프로토콜**은 너무나도 유명한 Transport Layer의 예시이다.
5. Session Layer : 이 계층부터 7계층까지는 한꺼번에 합쳐서 Application Layer라고 취급하기도 한다.
Session Layer는 통신에 필요한 권한 검사나 허가를 하거나, 세션을 복구하는 기능을 한다.
6. Presentation Layer : 같은 영상을 저장하더라도, 저장 방식에 따라서 파일의 형식이 avi, wma, mp4등으로 달라지는데, Presentation Layer는 이런 파일의 형식을 복호화, 암호화하는 역할을 주로 담당한다.
7. Application Layer : 웹브라우저처럼 사용자의 손길이 직접적으로 닿는 대부분의 경우가 이 계층에 해당한다.

세부적인 내용은 앞으로 포스팅별로 차차 다루어보겠다. (이미 전체적인 흐름은 다 나왔지만..)
<br/>
<br/>

## 마치며

처음으로 마크다운 형식을 이용해서 본격적인 포스팅을 진행해보는데, 그동안 네이버 블로그에서 해왔던 형식으로 진행하는 것이 익숙해서 우선 이런 방식으로 진행해보려고 한다. 앞으로 이 블로그를 얼마나 많은 사람들에게 공개할 지는 모르겠지만, 가장 큰 목적은 내가 공부해온 흔적들을 기록하는 것이므로, 초심을 잃지 않고 열심히 적어나가보려고 한다.
<br/>
<br/>

다음 포스팅은, Physical Layer와 Data Link Layer에 대해 작성하고, 추가적인 네트워크 관련 배경지식 역시 작성할 예정이다.
