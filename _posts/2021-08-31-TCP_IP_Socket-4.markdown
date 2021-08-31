---
layout: post
title: <TCP/IP Socket> 06. listen(), connect(), accept()에 대하여
date: 2021-08-31 21:50:23 +0900
category: TCP/IP_Socket
comments: true
---

## listen()

```c
#include <sys/socket.h>

int listen(int sock, int backlog);
```

여기서 sock으로 전달되는 소켓이 바로 서버 소켓(리스닝 소켓)이 된다. backlog는 연결요청 대기 큐의 크기정보이다. 이 숫자만큼 클라이언트의 연결요청을 대기시킬 수 있다.

## accept()

```c
#include <sys/socket.h>

int accept(int sock, struct sockaddr *addr, socklen_t *addrlen);
```

여기서 두번째 인자는 연결요청을 한 클라이언트의 주소 정보를 담을 변수의 주소값이다. 이 함수가 호출되면, 이 주소값에 클라이언트의 주소가 채워진다. accept()함수가 호출되면, 내부적으로 **데이터 입출력에 사용할 소켓을 새로 생성**하고 (서버 소켓과는 다르다!), 연결요청을 한 클라이언트 소켓과 연결된다.

## connect()

```c
#include <sys/socket.h>

int connect(int sock, struct sockaddr *servaddr, socklen_t *addrlen);
```

이 함수는 클라이언트에서 실행하는 함수인데, 여기서 두번째 인자로 **서버 주소**를 대입한다. 또한, connect()함수가 호출되면 자동적으로 운영체제 커널에서 클라이언트 소켓에 클라이언트 IP 주소가 할당되므로, 클라이언트 프로그램에서는 bind()를 호출할 필요가 없다.

## 전체적인 흐름

서버가 listen() 함수를 호출한 이후에야 클라이언트 connect() 함수호출이 가능하다. 또한, 클라이언트가 connect()를 호출하기 전에 서버에서 미리 accept()를 호출할 수 있다. 대신, 이때는 클라이언트가 connect()를 호출할 때 까지는 accept() 함수가 호출된 위치에서 블로킹된다.

## 마치며

Iterative, Echo 코드 구현은 이미 예전에 이 카테고리 처음에 Linux와 Window버전으로 구현한 것을 포스팅했으므로 생략한다.