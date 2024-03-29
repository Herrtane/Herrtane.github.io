---
layout: post
title: <TCP/IP Socket> 12. 다중접속 서버 - 멀티플렉싱 기반
date: 2021-10-02 15:50:23 +0900
category: TCP/IP_Socket
comments: true
---

## 멀티플렉싱

멀티프로세스는 오버헤드가 적지 않은 방법임은 쉽게 알 수 있다. 멀티플렉싱은 통신채널을 최소한으로 줄여서 최대한의 데이터를 전달하기 위해 사용되는 기술이다. 이를 클라이언트 - 서버 개념에 적용한다면, 접속해있는 클라이언트의 수에 상관없이, 서비스를 제공하는 프로세스의 수는 오로지 하나이다. 윈도우와 리눅스 공용으로 사용되는 멀티플렉싱 기법은 select() 함수를 사용하는 방법이 대표적이다.

## select()

select() 함수를 사용하면 여러 개의 파일 디스크립터를 모아서 한꺼번에 관찰하면서, 어느 디스크립터에 변화가 생겼는지 그 여부를 확인할 수 있다. 이 **파일 디스크립터의 묶음을 fd_set형 변수**라고 한다. 0과 1로 이루어진 비트 단위의 배열이며, 왼쪽부터 파일 디스크립터 0부터 순서대로 커지면서 저장된다. 아래의 매크로 함수들로 이 fd_set형 변수를 조작한다.

```c
fd_set set;

FD_ZERO(&set);      // 모든 비트를 0으로 초기화
FD_SET(1, &set);    // 첫번째 인자로 전달된 파일 디스크립터(여기서는 파일 디스크립터 1)를 fd_set에 등록한다. (0에서 1로 비트를 변환)
FD_CLR(2, &set);    // 첫번째 인자로 전달된 파일 디스크립터(여기서는 파일 디스크립터 2)를 fd_set에서 해당하는 정보를 삭제한다. (1에서 0으로 비트를 변환)
```

다음으로 select() 함수를 보이겠다.

```c
#include <sys/select.h>
#include <sys/time.h>

int select(int maxfd, fd_set *readset, fd_set *writeset, fd_set *exceptset, const struct timeval *timeout);

struct timeval{
    long tv_sec;
    long tv_usec;
}
```

첫번째 인자는 파일 디스크립터의 관찰 범위이다. 파일 디스크립터의 값은 생성될 때마다 1씩 증가하기 때문에 가장 큰 파일 디스크립터의 값에 1을 더해서 인자로 전달한다. 마지막 인자는 타임아웃 인자이다. select() 함수는 관찰중인 파일 디스크립터에 변화가 생겨야 반환을 하므로, 그 전에는 블로킹 상태에 빠진다. 이를 방지해주는 역할이다. select() 함수 호출 이후 0이 아닌 양수가 반환이 되면, 그 수 만큼 파일 디스크립터에 변화가 발생했다는 뜻이다. **select() 함수 호출 이후에는 1로 설정된 모든 비트가 다 0으로 변경되지만, 변화가 발생한 파일 디스크립터에 해당하는 비트는 1로 남아있게 되고, 이 남아있는 비트를 확인해서 어떤 파일 디스크립터에 변화가 생겼는지 확인하는 방식**을 사용한다.

## epoll()

select()함수는 꽤나 오래 전에 개발된 멀티플렉싱 기법이다. 윈도우와 리눅스 모두에 호환된다는 큰 장점이 있지만, 아래와 같은 성능상의 큰 단점이 존재한다.

1. select() 호출 이후에 등장하는, 모든 파일 디스크립터를 대상으로 하는 반복문으로 인한 성능 저하
2. select() 호출마다 관찰대상에 대한 정보를 매번 운영체제에게 전달해야 하는 과정에서의 성능 저하

select()는 소켓의 변화를 관찰하는 함수이고, 소켓은 운영체제가 관리하는 영역이기 때문에, 2번의 문제가 발생한다. 이러한 단점들을 개선한 방식은 리눅스의 epoll 방식이 있다. 위에서 언급한 두가지 단점을 극복한 함수이다.

### epoll_create

epoll 방식에서는 fd_set형 변수를 선언해서 파일 디스크립터 관찰 대상을 저장할 필요가 없다. 운영체제가 직접 저장을 담당하기 때문에, epoll 방식에서는 운영체제에 파일 디스크립터의 저장을 위한 저장소의 생성을 요청하는 epoll_create를 사용한다. 이 함수호출 시 생성되는 파일 디스크립터의 저장소를 **epoll 인스턴스** 라고 한다. 이 함수는 epoll 파일 디스크립터를 반환하는데, 이는 epoll 인스턴스를 구분하는 목적으로 사용된다. 

```c
#include <sys/epoll.h>

int epoll_create(int size); // 최근 버전의 리눅스 커널부터는 size 전달인자가 완전히 무시된다.
```

### epoll_ctl

FD_SET, FD_CLR 대신 운영체제에 파일 디스크립터의 추가 및 삭제를 요청하는 epoll_ctl를 사용한다. epoll 인스턴스 생성 후에, 관찰대상이 되는 파일 디스크립터를 등록하는 용도로 사용한다. 

```c
#include <sys/epoll.h>

int epoll_ctl(int epfd, int op, int fd, struct epoll_event* event);
```

예시를 들면 이해하기 쉽다. epoll_ctl(A, EPOLL_CTL_ADD, B, C); 라는 문장을 해석하면, A라는 epoll인스턴스에 B라는 파일 디스크립터를 관찰대상으로 추가(등록)하고, C 구조체를 통해 전달된 이벤트를 관찰하는 것을 목적으로 한다는 뜻이다. epoll_ctl(A, EPOLL_CTL_DEL, B, NULL); 이면 반대로 삭제한다는 뜻이겠지.
실제적인 예시는 아래와 같다. event의 종류는 별도의 레퍼런스를 참조하자. 

```c
struct epoll_event event;
...
event.events = EPOLLIN; // 수신할 데이터가 존재하는 상황(이벤트) 발생시
event.data.fd = sockfd;
epoll_ctl(epfd, EPOLL_CTL_ADD, sockfd, &event);
```

### epoll_wait

파일 디스크립터의 변화를 기다리기 위해 select()를 호출하는 select방식과 달리, epoll_wait 함수를 호출한다. 

```c
#include <sys/epoll.h>

int epoll_wait(int epfd, struct epoll_event* events, int maxevents, int timeout);
```

이벤트가 발생한 파일 디스크립터가 채워질 버퍼의 주소값이 두번째 인자(동적으로 할당한다)로 전달된다. 네번째 인자로 -1을 전달하면, 이벤트가 발생할 때까지 무한 대기한다. 함수호출 후에는 이벤트가 발생한 파일 디스크립터의 수가 반환된다.
구체적인 사용 예시는 아래와 같다.

```c
int event_cnt;
struct epoll_event* ep_events;
...
ep_events = malloc(sizeof(struct epoll_event)*EPOLL_SIZE);
...
event_cnt = epoll_wait(epfd, ep_events, EPOLL_SIZE, -1);
```

### epoll_event

fd_set형 변수의 변화를 통해서 관찰대상의 상태변화를 관찰하는 select방식과 달리, epoll_event라는 아래의 구조체를 기반으로 상태변화가 발생한 파일디스크립터가 별도로 묶인다. 이 구조체 기반의 배열을 넉넉한 길이(변화가 생긴 fd가 하나씩 저장되므로 충분한 공간이 필요)로 선언한다. 

```c
struct epoll_event{
    __uint32_t events;
    epoll_data_t data;
}

typedef union epoll_data{
    void* ptr;
    int fd;
    __uint32_t u32;
    __uint64_t u64;
} epoll_data_t;
```

## 마치며

역시 자세한 서버 구현 코드는 처음 포스팅한 github 링크를 참고하기 바란다.
윈도우 상에서는 IOCP기법이 존재하지만, 이는 추후에 필요한 경우에 다루도록 하겠다.