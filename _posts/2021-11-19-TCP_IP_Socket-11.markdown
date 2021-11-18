---
layout: post
title: <TCP/IP Socket> 13. 다중접속 서버 - 멀티쓰레딩 기반 (1)
date: 2021-11-19 15:50:23 +0900
category: TCP/IP_Socket
comments: true
---

## 쓰레드

멀티프로세스의 단점은 명확하다. 무거운 프로세스를 생성하는 과정과 프로세스 간의 통신을 위해 IPC 기법을 별도로 적용해야 한다는 점이다. 무엇보다도, 여러 프로세스가 동시에 실행될 때, context switching에 따른 오버헤드는 상당히 심각할 수 있다. 이러한 단점을 극복한 것이 바로 쓰레드이다. (context switching과 쓰레드에 관련된 설명은 OS관련 포스팅을 진행할 때 자세히 다루겠다.)

<br/>

리눅스에서 쓰레드를 사용한 코드를 gcc를 이용해서 컴파일 할 때는, 반드시 gcc ~.c -o ~ -lpthread처럼 -lpthread 옵션을 추가해서 쓰레드 라이브러리의 링크를 별도로 지시해야 한다.

## 쓰레드의 생성

쓰레드는 별도의 실행 흐름을 갖기 때문에, 쓰레드만의 main함수가 필요하다. 이 함수를 기반으로 쓰레드를 실행해줄 것을 운영체제에 요청하는 함수가 아래와 같다.

```c
#include <pthread.h>

int pthread_create(
    pthread_t* restrict thread, const pthread_attr_t* restrict attr, void* (*start_routine)(void*), void* restrict arg
);
```

첫번째 인자로 쓰레드의 ID 저장을 위한 변수를 전달하고, 두번째 인자는 보통 NULL을 전달하여 기본적인 특성의 쓰레드를 생성한다. 세번째 인자가 main역할을 하는 함수의 포인터를 전달하고, 네번째 인자로는 세번째 인자로 전달된 함수가 호출될 때 전달하고 싶은 인자의 정보를 담고 있는 변수의 주소 값을 전달한다.
예시는 아래와 같다.

```c
if(pthread_create(&t_id, NULL, thread_main, (void*)&thread_param) != 0){
    puts("pthread_create() error");
    return -1;
}
```

성공시, main 프로세스의 실행 중에 이 함수의 호출로 분기가 갈라지면서 새로운 흐름을 형성한다.

## 쓰레드의 실행흐름 조절

별도의 과정을 거치지 않는다면, pthread_create() 만으로는 main 프로세스가 쓰레드보다 먼저 종료되지 말라는 보장이 없다. 지금부터 다루는 함수를 사용하면 쓰레드가 종료될 때 까지 main 프로세스를 잠시 일시정지 시킬 수 있다.

```c
#include <pthread.h>

int pthread_join(pthread_t thread, void** status);
```

첫번째 인자로 전달되는 쓰레드가 종료될 때 까지 함수는 반환되지 않고, 두번째 인자를 통해 쓰레드의 main 함수가 반환하는 값이 포인터 변수의 주소 값으로 전달된다.

## 마치며

다음 포스팅에선 이어서 임계영역과 관련된 문제를 다루도록 하겠다.