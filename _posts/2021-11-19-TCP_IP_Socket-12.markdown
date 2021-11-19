--- 
layout​: ​post 
title​: ​<TCP/IP Socket> 14. 다중접속 서버 - 멀티쓰레딩 기반 (2) 
date​: ​2021-11-19 20:50:23 +0900 
category​: ​TCP/IP_Socket 
comments​: true 
--- 

## 쓰레드의 문제점과 임계영역

둘 이상의 쓰레드를 동시에 실행되면 임계영역(critical section)이 발생할 수 있으므로, 이를 각별히 신경써야 한다. (임계영역에 대해선 OS 카테고리에서 자세히 다루겠다.) 이러한 임계영역에 대해, 쓰레드에 안전한 함수가 있고, 쓰레드에 불안전한 함수가 있는데, 대부분의 표준함수들은 쓰레드에 안전하거나, 불안전한 함수들은 별도로 안전한 함수가 따로 정의되어 있으므로, 편하게 표준함수를 사용하면 된다. 혹여나 쓰레드에 불안전한 함수일 경우, **함수의 이름 뒤에 _r** 를 추가할 경우 안전한 함수로 변경된다. 매번 함수를 변경하는 수고로움을 덜기 위해서, gcc 컴파일시 **-D_REENTRANT** 옵션을 추가해주는 방법을 사용하자. 
 
## 쓰레드 동기화
 
​쓰레드 동기화(synchronization)는 다음과 같은 상황을 해결하는 것을 말한다.
1. 동일한 메모리 영역으로의 동시접근 발생
2. 동일한 메모리 영역에 접근하는 쓰레드의 실행순서를 지정해야 하는 상황

### 뮤텍스(Mutex)

쓰레드의 동시접근을 허용하지 않는 대표적인 방법으로 Mutual Exclusion이 있다. 윤성우 선생님의 비유에 따르면 화장실 칸의 예시와 유사하다. 사용중인 화장실은 한명씩만 들어갈 수 있고, 들어갈 때 문을 잠그고 나올 때 문을 여는 방식이다. 기다리는 사람들은 기다리는 순서에 따라 차례대로 화장실에 들어간다.

<br/>

뮤텍스도 마찬가지로 critical section부분에서 자물쇠와 같은 역할을 한다. 

```c
#include <pthread.h>

int pthread_mutex_init(pthread_mutex_t* mutex, const pthread_mutexattr_t* attr);
int pthread_mutex_destroy(pthread_mutex_t* mutex);
```

attr는 별도의 특성을 지정하지 않을 경우 NULL을 전달해주면 된다. 다음으로, 자물쇠를 잠그고 푸는 함수는 아래와 같다.

```c
#include <pthread.h>

int pthread_mutex_lock(pthread_mutex_t* mutex);
int pthread_mutex_unlock(pthread_mutex_t* mutex);
```

예시코드는 아래와 같다.

```c
...
pthread_mutex_t mutex;
...
pthread_mutex_init(&mutex, NULL);
...
pthread_mutex_lock(&mutex);
// cirtical section
pthread_mutex_unlock(&mutex);
...
pthread_mutex_destroy(&mutex);
...
```

### 세마포어

세마포어(semaphore)도 뮤텍스와 거의 유사하다. 다만, 바이너리 세마포어가 아닌 경우, 한번에 한 쓰레드만 임계영역에 들어가는 것이 아닌, 다중 처리가 가능하다. 이 역시 OS에서 자세히 다루도록 하겠다. 지금 다룰 것은 바이너리 세마포어이다.

```c
#include <semaphore.h>

int sem_init(sem_t* sem, int pshared, unsigned int value);
int sem_destroy(sem_t* sem);
```

위에서 pshared 인자를 0으로 전달시 바이너리 세마포어가 되고, 0 이외의 값 전달시 둘 이상의 쓰레드(프로세스)에 의해 접근 가능한 세마포어가 생성된다. value는 생성되는 세마포어의 초기 값을 지정한다.

```c
#include <semphore.h>

int sem_post(sem_t* sem);
int sem_wait(sem_t* sem);
```

init을 통해 만들어진 세마포어 오브젝트에는 세마포어 값이라는 정수가 하나 기록되는데, 이 값은 post시 1 증가, wait시 1 감소한다. 이 값은 0보다 작아질 수 없으므로, 0일때 wait호출 시 블로킹 상태에 놓인다. 예시 코드는 아래와 같다.

```c
sem_wait(&sem);
// critical section
sem_post(&sem);
```

참고로, 세마포어를 sem_one, sem_two 처럼 두개 이상 두어야 되는 경우도 흔하다. 이는 쓰레드간의 순서를 지키기 위해 구현한다. 이에 대한 예시는 생략하겠다. 

## 쓰레드의 소멸

쓰레드를 소멸시키는 방법은 2가지가 있다. 그중 우리가 앞에서 다뤘던, pthread_join()을 이용한 방법은 자명하게도 쓰레드의 종료를 대기한 이후 쓰레드의 소멸까지 유도가 된다. 다만, 이는 그 동안 블로킹상태로 놓이기 때문에, 보통은 다음의 방법을 사용한다.

```c
#include <pthread.h>

int pthread_detach(pthread_t thread);
```

## 마치며

자세한 멀티스레드 기반 서버 구현은 역시 별도로 게시한 링크에 게시해놓았다. 이상으로 리눅스 기반의 소켓 프로그래밍에 대한 포스팅을 일단락지으려고한다. 1년 전에 학습을 끝냈던 소켓 프로그래밍에 대한 내용을 언제쯤 포스팅하나 고민하고 있었는데, 이렇게 마무리짓고나니 후련하다. 소켓 프로그래밍에 대한 포스팅은 앞으로 프로젝트를 진행하거나 별도로 필요한 지식들이 생기게 되면 그때그때 추가할 계획이다.

<br/>

다음 포스팅 주제는 1년동안 쭉 혼자서 공부해왔던 다양한 과목들에 대해 다룰 예정이다. OS, 리눅스 시스템 프로그래밍, 데이터베이스 등 많은 책들을 공부했기 때문에, 작성할 포스팅 주제들이 많다.. 하하.