---
layout: post
title: <Network> 07. Transport layer (2) - TCP
date: 2020-09-13 23:10:23 +0900
category: Network
comments: true
---
이번 포스팅에서는 Transport layer의 중요한 역할을 담당하는 TCP에 대해서 이어서 다루어 보겠다. TCP의 중요도와 분량상 내용이 평소에 비해 길다.

### 2. Transmission Control Protocol (TCP)

TCP는 UDP와는 사뭇 다른 특징을 가졌다. Datagram방식이었던 UDP와는 반대로 virtual-circuit을 형성하는 connection-oriented이고, UDP보다 훨씬 더 깊은 flow and error control을 갖추었다. 또한, **수신단까지 전송만 하는 IP와 다르게, TCP는 connection establishment, data transfer, connection termination의 단계를 요구**하며, full-duplex 모드를 통해 데이터 전송이 이루어진다.  

<br/>

TCP는 독특한 특성을 가지는데, stream-oriented protocol이라는 것이다. 서로 통신하려는 두 process 사이에 가상의 "tube"를 논리적으로 형성하는데, 이 "tube"를 통해 데이터를 교환한다. 또한, 이 과정에서 송신측과 수신측 process가 동일한 속도로 데이터를 주고받을 확률은 낮기 때문에, 별도의 버퍼를 지니게끔 한다. 

<br/>

또한, TCP가 "tube"를 통해서 데이터를 주고받을 때, 데이터의 단위는 byte로 구분하는데, 이 byte들을 몇개 묶어서 패킷으로 전송하는 것이 보통이며, 이 패킷들을 **segment**라고 부른다. 아래의 사진에서 이 내용들을 도식화해보자.

![TCP_buffer_segment]({{site.url}}/img/TCP_buffer_segment.jpg)

더불어서, TCP는 full-duplex service이고, 통신 과정에서 piggybacking을 사용한다.

<br/>

TCP에서 오고 가는 segment들의 세부적인 특징을 살펴보자.

- 각 segment들은 sequence number와 acknowledgement number 필드가 존재한다.
- 각 segment들은 별도로 1부터 넘버링되지 않는다. 대신에, 랜덤하게 생성된 번호를 기준으로 그 번호부터 넘버링된다. 더 구체적으로 예를 들면, 송신측이 보내는 segment에서 첫 번째 byte의 번호가 segment 번호가 되는데, segment 1, segment 2가 아니라, segment 1057, segment 2057 이런 식으로 매겨진다. 
- acknowledgement number는 segment를 수신 받은 후, 다음으로 수신받아야 하는 byte들(즉, 다음 segment)의 첫 byte 번호로 부여한다. 예를 들어, 송신측에서 segment 1057을 보냈고, 각 segment의 size가 1000byte라고 하자. 이를 수신측에서 받았다면, 수신측에서 다음에 이어서 받을 segment는 1057 + 1000 = 2057 번째 byte가 되어야 한다. 따라서, 수신측에서 송신측으로 보내는 segment의 ack 번호는 2057이 된다. 


#### Connection Establishment (연결 설정)

역시 자세한 과정은 그림을 통해서 정리하도록 하자. 아래의 그림을 미리 설명하자면, TCP에서 connection을 establish하는 과정으로, 송신 측이 먼저 수신측에게 연결을 요청하는 메시지로 SYN을 보내고, 수신측이 알겠다는 ACK메시지와 함께, SYN을 같이 요청하는 SYN 메시지를 보낸다. 마지막으로 송신측에서 확인했다는 ACK메시지를 보냄으로써 connection established된다. 이를 중요한 말로 **3-Way-Handshake**라고 한다.

![TCP_3wayhandshake]({{site.url}}/img/TCP_3wayhandshake.jpg)

참고로, 이를 악용한 보안 공격이 **SYN Flooding Attack**인데, 연결을 설정하려는 메시지를 기하급수적으로 많이 보내서 수신측의 서버를 과부하시키는 공격을 말한다.

#### Connection Termination (연결 종료)

마찬가지로, 연결을 종료하는 connection termination과정도 거의 유사하다. 그림은 아래와 같다.

![TCP_termination]({{site.url}}/img/TCP_termination.jpg)

<br/>

그런데, **만약에 한쪽은 연결을 종료하고 싶은데, 한쪽에서는 아직 보내야 할 데이터들이 남아있다면?** 어떻게 해야할까? 이를 **half-close**라고 하는데, 그 과정은 아래와 같다. 해결책은, 아직 보내야 할 데이터들이 남은 쪽에서 FIN메시지를 보내기 전에, 데이터를 모두 보낸 뒤에 마지막 segment와 함께 FIN메시지를 보내는 것이다. 

![TCP_termination2]({{site.url}}/img/TCP_termination2.jpg)

### TCP의 다양한 성능 개선 방법들

#### 1. Fast retransmission

일반적인 송수신 상황에서 TCP는 한번 segment를 받을 때마다 ACK메시지를 답하는 것은 낭비라고 판단하여, ACK-delaying timer를 사용한다. 이는 일정 시간동안 timer를 실행하여, 그 시간동안 오는 segment들을 모두 수신한 뒤에, 정상적으로 수신하였다면 한꺼번에 segment들을 처리하고, 그 다음에 올 segment들을 ACK메시지로 답하는 방식이다.

![TCP_normaltimer]({{site.url}}/img/TCP_normaltimer.jpg)

그렇다면, 만약 이 과정에서 송신측의 segment 하나가 소실된다면 어떻게 대처해야 할까? 

![TCP_lostsegment]({{site.url}}/img/TCP_lostsegment.jpg)

이에 대한 해결책으로 고안된 방법은 다음과 같다. 수신측에서 수신 받지 못한 segment를 다시 보내달라는 ACK메시지를 3번 연속 보내게 되면, 송신 측에서 즉시 누락된 segment를 다시 보내주는 방식이다. 이를 **Fast retransmission**방식이라고 한다. 자세한 과정은 아래의 그림을 참고하자.

![TCP_fastretransmission]({{site.url}}/img/TCP_fastretransmission.jpg)

#### 2. Slow start

복잡한 네트워크에서는 다른 네트워크 전송에 피해를 주거나 영향을 주지 않기 위해서, 시작부터 대량의 데이터를 전송하는 것을 피한다. TCP의 개선 방안 중에는 **slow start** 방식이 있는데, 소량의 데이터 전송을 시작으로 하여, ACK메시지가 잘 도착할수록, 점점 기하급수적으로 데이터 전송을 늘림으로써 네트워크 상의 피해와 영향을 최소화하는 방식이다.

#### 3. Congestion avoidance

점점 기하급수적으로 데이터 전송을 늘리다보면, 어느 순간 수신측의 버퍼보다 데이터의 양이 많아져서 데이터 전송에 누락이 발생하거나 실패하는 경우가 존재할 수 있다. 따라서, 어느 정도까지는 slow start 방식으로 데이터를 전송하다가, 어느 정도부터는 기하급수적이 아닌, 조금씩 데이터 전송량을 늘리는 additive increase 방식을 사용하는데, 이를 **congestion avoidance**라고 한다.

<br/>

위의 예시들을 바탕으로 실전에서 합쳐서 적용한 두 방식을 소개한다.

![TCP_taho]({{site.url}}/img/TCP_taho.jpg)

<br/>

위의 방식은 **Taho**라고 하는데, 이 방식에서는 송신에 문제가 발생하면 다시 전송속도를 원점부터 시작하는 비효율적인 단점이 존재한다. 이 단점을 보완해서, 일정 속도의 전송 속도부터 이어서 전송하는 방식이 생겨났는데, 이를 **Reno**방식이라고 하며, 아래와 같다. 

![TCP_reno]({{site.url}}/img/TCP_reno.jpg)

<br/>
<br/>

## 마치며

이번 포스팅은 TCP를 다루는 중요한 부분이라, 유독 세밀하고 생략없이 포스팅하다보니 사진의 양도 어마어마하고 내용도 어마어마하다. 작성하는 동안 손가락 아파서 혼났다. 그래도, 이제 OSI 7layer 중에 비중 있게 다루어야 할 계층들은 마무리되었기에 고지가 눈앞에 보인다.

<br/>

다음 포스팅은 Application layer에 대해서 다룰 예정이다. 흔히 Session layer, Presentation layer, Application layer를 하나로 합쳐서 Application layer로 통칭한다. 다음 포스팅은 이 세 층을 합쳐서 다룰 예정이다. 이제 거의 다 왔다..! 
