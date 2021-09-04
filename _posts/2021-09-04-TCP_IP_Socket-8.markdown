---
layout: post
title: <TCP/IP Socket> 10. 다중접속 서버 - 멀티프로세스 기반
date: 2021-09-04 14:50:23 +0900
category: TCP/IP_Socket
comments: true
---

## 다중접속 서버의 종류

한 서버에 한 클라이언트씩만 줄서서 들어가는 서버는, 사실상 효율성이 매우 낮다. 그래서 동시에 여러 클라이언트가 접속할 수 있게끔 해야하는데, 이를 다중접속 서버라고 한다. 그 종류는 아래와 같다.

1. 멀티프로세스 기반 서버
2. 멀티플렉싱 기반 서버
3. 멀티스레딩 기반 서버

우선 멀티프로세스 기반 서버를 살펴보도록 하겠다. 프로세스에 대한 기본 지식은 OS와 시스템프로그래밍에서도 그대로 겹치는 내용이다.

## 프로세스

메모리 공간을 차지한 상태에서 실행중인 프로그램을 프로세스라고 함은 너무나 잘 알려진 사실이다. 이 프로세스를 생성하는 방법이 몇가지 있는데, fork()함수만 여기선 다루겠다. 참고로, 시스템 프로그래밍에서는 exec()함수도 깊게 다룬다. 이는 다른 카테고리에서 다루겠다.

```c
#include <unistd.h>

pid_t fork(void);
```

이 함수를 통해 부모프로세스에서 완전히 동일한 (메모리 영역까지 복사한) 자식 프로세스가 분기되어 생성된다. **두 프로세스의 차이점은 fork()함수의 반환값으로 구분할 수 있는데, 자식프로세스는 0, 부모프로세스는 자식프로세스의 pid가 반환값**이다. 아래와 같이 구분할 수 있다.

```c
pid = fork();
if(pid==0)
    exit(10);
else
    return;
```

## 좀비프로세스

자식프로세스가 종료되면, **그 반환값이나 exit함수 인자값이 부모에게 전달되어야 완전히 자식프로세스가 소멸**된다. 반대로 이야기하면, 이 값이 전달되기 전까지는 소멸되지 않고 남아있는데, 이를 **좀비프로세스**라고 한다. 이를 방지하기 위해, 아래의 함수를 사용한다.

```c
#include <sys/wait.h>

pid_t wait(int *statloc);
pid_t waitpid(pid_t pid, int *statloc, int options);
```

wait()는 자식프로세스가 종료되기 전까지 블로킹되는 단점이 있기 때문에, waitpid()를 주로 사용한다. 블로킹을 막고 함수를 빠져나오기 위해선, 세번째 인자로는 WNOHANG을 사용한다. 그리고 이를 실제로 사용하는 예시는 아래와 같다.

```c
waitpid(-1, &status, WNOHANG);
if(WIFEXITED(status)){
    puts("Normal termination");
    printf("Child pass num: %d", WEXITSTATUS(status));
}
```

## 시그널 핸들링1 : signal()

자식프로세스를 좀비프로세스로 만들지 않기 위해 wait()류의 함수를 호출하는 것은 좋지만, 자식프로세스가 언제 종료될지 모르는 채로 계속 기다리고 있을 수는 없다. 이때, **시그널(signal)**을 사용하면 이 문제를 쉽게 해결할 수 있다. 자식프로세스가 종료되면 SIGCHLD라는 시그널이 발생하는데, 이를 운영체제가 확인하고 부모프로세스에게 알려줄 수 있다. 이때, 아래의 함수를 사용하여 **시그널을 등록**하면, 해당 시그널이 발생하면 운영체제가 특정 함수를 호출해준다.

```c
#include <signal.h>

void (*signal(int signo, void(*func)(int)));
// 함수 포인터가 반환형이므로, 유의해서 다루어야한다.
```

첫번째 인자로 전달할 수 있는 것은 시그널 상수인데, SIGINT, SIGALRM, SIGCHLD 등이 있다. 이외의 상수들은 시스템프로그래밍 카테고리에서 자세히 다루겠다. 두번째 인자로 전달되는 함수는 **시그널 핸들러**라고도 부르며, 해당 시그널이 발생시 호출될 함수를 등록한다. 이를 사용한 예시 코드는 아래와 같다.

```c
#include <stdio.h>
#include <unistd.h>
#include <signal.h>

void timeout(int sig){
    if(sig == SIGALRM)
        puts("Time out");
    alarm(2);
}

int main(void){
    int i;
    signal(SIGALRM, timeout);
    alarm(2);

    sleep(100);

    return 0;
}
```

## 시그널 핸들링2 : sigaction()

signal함수는 유닉스 계열 운영체제별로 약간의 차이를 보이지만, sigaction함수는 그렇지 않고 더 안정적이다. 그래서 sigaction함수의 사용을 더 권장한다.

```c
#include <signal.h>

int sigaction(int signo, const struct sigaction *act, struct sigaction *oldact);

struct sigaction{
    void (*sa_handler)(int);
    sigset_t sa_mask;
    int sa_flags;
}
```

아래의 구조체가 별도로 사용되는 것이 이 함수의 특징이다. 구조체의 첫번째 맴버에 **시그널 핸들러**로 쓰일 함수를 등록하면 된다. sa_mask는 모든 비트를 0으로, sa_flags는 0을 전달하면 되는데, 더 세부적인 사용법은 역시 시스템 프로그래밍 카테고리에서 다루도록 하겠다. 이 함수의 사용법은 아래와 같다.

```c
struct sigaction act;
act.sa_handler = timeout;   // timeout 핸들러는 signal 예시의 그것과 동일하다.
sigemptyset(&act.sa_mask);  // sa_mask의 모든 비트를 0으로 초기화하는 함수이다.
act.sa_flags = 0;

sigaction(SIGALRM, &act, 0);
```

## 마치며

멀티프로세스 서버 구현은 카테고리 첫 포스팅에 게시한 링크에 있으므로, 별도의 코드 작성은 생략한다. 그 서버 구현에 필요한 지식들이 이번 포스팅에서 다룬 내용들이다!