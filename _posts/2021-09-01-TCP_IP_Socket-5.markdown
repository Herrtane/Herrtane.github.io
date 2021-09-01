---
layout: post
title: <TCP/IP Socket> 07. UDP
date: 2021-09-01 21:50:23 +0900
category: TCP/IP_Socket
comments: true
---

## UDP의 개념

TCP는 전화에 비유할 수 있고, UDP는 편지에 비유할 수 있다. UDP는 패킷을 보낼 때 마다 편지처럼 보내는 호스트와 받는 호스트의 주소를 입력해야 하고, 상호간의 별도 연결 과정 없이 보내기만 하면 된다. 매번 **흐름 제어를 하는 TCP**와는 다르게, UDP는 그렇지 않기 때문에 상대적으로 구현이 용이하지만, 그만큼 신뢰성이 낮다. 이런 특징으로 인해, 송수신하는 데이터의 양이 작으면서 연결을 자주 해야할 경우 TCP보다는 UDP가 효과적이고 빠르게 동작한다.

<br/>

또한, UDP는 TCP와 달리 데이터의 경계가 뚜렷한 프로토콜이기 때문에, 반드시 입력함수의 호출횟수와 출력함수의 호출횟수가 일치해야 데이터 전부를 수신할 수 있다.

<br/>

Host by Host로 패킷이 보내지는 과정은 IP의 역할이고, Host로 수신된 패킷을 PORT를 통해 UDP소켓까지 전달하는 것이 UDP의 역할이다. 이는 컴퓨터 네트워크에서 자세히 다뤘었다. 

## UDP 기반 서버/클라이언트

UDP는 연결 설정 과정이 필요없다고 했다. 따라서 listen()함수와 accept()함수가 사용되지 않는다. 또한, 서버에서 여러 클라이언트에게 서비스를 제공하기 위해 TCP는 여러 소켓이 필요했지만, UDP는 편지함의 개념과 비슷하기 때문에 서버든 클라이언트든 하나의 소켓만 생성하면 된다.

<br/>

단, 한번의 연결 설정 과정 이후에는 주소 정보를 따로 입력할 필요가 없는 TCP와는 다르게, UDP는 데이터를 전송할 때마다 주소 정보를 별도로 추가해야 한다. 그래서 UDP에서는 sendto() 함수와 recvfrom()함수를 별도로 사용하여, 호출 때마다 주소 정보를 추가로 입력한다.

```c
#include <sys/socket.h>

ssize_t sendto(int sock, void *buff, size_t nbytes, int flags, struct sockaddr *to, socklen_t addrlen);
ssize_t recvfrom(int sock, void *buff, size_t nbytes, int flags, struct sockaddr *from, socklen_t *addrlen);
```

## UDP에서의 connect()

```c
#include <sys/socket.h>

int connect(int sock, struct sockaddr *servaddr, socklen_t *addrlen);
```

일반적으로 UDP에서는 클라이언트에서 connect()함수를 호출하지 않는다. 그렇다면, **클라이언트가 데이터를 보낼 때 주소 정보는 언제 할당될까?** 보통 아래처럼 2가지 방법이 있다.

1. sendto()호출 이전에 bind()를 통해 주소를 할당
2. bind()를 호출하지 않아도, sendto()가 호출되는 시점에 자동으로 할당됨

이후 데이터가 전송되면, UDP 소켓에 등록된 목적지 정보를 삭제하는 것으로 하나의 데이터 전송 과정이 끝난다. 이 과정을 매 전송때마다 반복하는 것이 UDP이다. 이렇게 목적지 정보가 등록되어 있지 않은 소켓을 **unconnected 소켓**이라 한다. 반대로, **한 호스트를 대상으로 여러번 데이터를 전송해야 되는 경우, 위의 과정을 매번 반복하는 건 비효율적**이다. 이럴때는 connect()함수를 사용하여 **connected 소켓**을 생성하면 된다. 방법은 아래와 같다.

```c
sock = socket(PF_INET, SOCK_DGRAM, 0);
memset(&adr, 0, sizeof(adr));
adr.sin_family = AF_INET;
adr.sin_addr.s_addr = ...
adr.sin_port = ...

connect(sock, (struct sockaddr*)&adr, sizeof(adr));
```

잘 보면, TCP 소켓 생성과정과 거의 유사하다. SOCK_DGRAM을 인자로 사용한다는 차이가 존재할 뿐이다. 이렇게 connected 소켓이 생성되면, 대상이 정해졌으므로 write, read함수의 호출도 가능해진다.

## 마치며

이상으로 UDP에 대한 설명도 마무리한다.