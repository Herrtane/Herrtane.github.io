---
layout: post
title: <TCP/IP Socket> 09. DNS (Domain Name System), Time-wait
date: 2021-09-04 13:50:23 +0900
category: TCP/IP_Socket
comments: true
---

## DNS의 개념

사실 워낙 DNS는 유명한 시스템이라, 간단히만 짚고 넘어가고자 한다. 우리가 인터넷 상에서 어떤 사이트에 접속하기 위해서는 IP주소를 입력해야 하는데, 매번 여러 숫자로 이루어진 IP주소를 입력할 수는 없는지라, 그 IP주소에 대응되는 도메인 주소를 입력해도 그 사이트에 접속할 수 있게끔 하는 것이 바로 **DNS**이다. 우리가 사이트에 www.abcd.com 이라고 입력할 경우, 인터넷 브라우저는 이 주소를 우리 컴퓨터에 깔려있는 Default DNS Server에 질의한다. 하지만 Default DNS Server에 세상의 모든 주소가 다 대응되어 있지는 않기 때문에, 만약 이 곳에 해당 주소가 없을 경우, 상위 DNS Server에 해당 DNS Server가 다시 질의한다. 이런 식으로, 최종적으로 Root DNS Server까지 질의해서 그 주소에 대응되는 IP 주소를 찾아낸다. 그래서 DNS 역시 일종의 **분산 데이터베이스 시스템**이다.

## Time-wait 상태

우리는 TCP 연결 종료 과정에서 Four-way handshaking에 대해서 다루었었다. 먼저 FIN 메시지를 보내는 호스트 A에게 호스트 B가 ACK와 FIN 메시지를 차례로 보낸 뒤에, 호스트 A가 ACK메시지로 마무리하는 과정이었다. 

<br/>

그런데, 이 과정을 서버와 클라이언트 관계에서 살펴보면, 만약 서버와 클라이언트가 서로 메시지를 주고받는 와중에, 서버측에서 급하게 강제종료를 할 경우를 살펴보아야 한다. 만약, **서버가 클라이언트에 마지막 4번째로 보낸 ACK메시지가 제대로 전달이 안될 경우**, 클라이언트는 서버에게 재차 FIN 메시지를 보낼 것인데, 서버가 이것을 고려하지 않고 ACK메시지 전송 후에 바로 연결을 끊는다면, 영원히 클라이언트는 서버의 ACK메시지를 수신할 수 없게된다. 이를 방지하기 위해, **서버가 연결을 끊기 전에, 일정시간 대기시간을 가지면서 클라이언트로부터 재차 FIN메시지가 오는지 확인하는 것을 Time-wait** 이라고 한다. 

<br/>

그래서, 서버와 클라이언트를 실행하고 서버를 강제종료 시킬 경우, 곧바로 다시 서버를 동일한 PORT 번호로 열려고 하면 bind() 에러가 발생하는데, 이 Time-wait 때문인 것이다. 만약, 급하게 다시 서버를 동일 PORT 번호에서 재실행해야 할 경우, 아래의 코드를 사용해야 한다.

```c
#include <sys/socket.h>

optlem = sizeof(option);
option = TRUE;
setsockopt(serv_sock, SOL_SOCKET, SO_REUSEADDR, (void*)&option, optlen);
```

SO_REUSEADDR의 디폴트값은 FALSE이다. 이를 TRUE로 바꾸면 Time-wait 상태더라도, 동일 PORT 번호를 재사용할 수 있게끔 해준다.

## 마치며

다음 포스팅부터는, 본격적인 다중접속 서버 구현 방법들에 대해 적을 예정이다.