---
layout: post
title: <System Hacking> 18. 포인터와 malloc에 대한 완전한 복습
date: 2022-09-30 21:30:23 +0900
category: System_Hacking
comments: true
---

## hook 소스코드

```c
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>
#include <unistd.h>

void alarm_handler() {
    puts("TIME OUT");
    exit(-1);
}

void initialize() {
    setvbuf(stdin, NULL, _IONBF, 0);
    setvbuf(stdout, NULL, _IONBF, 0);
    signal(SIGALRM, alarm_handler);
    alarm(60);
}

int main(int argc, char *argv[]) {
    long *ptr;
    size_t size;

    initialize();

    printf("stdout: %p\n", stdout);
    printf("Size: ");
    scanf("%ld", &size);

    ptr = malloc(size);

    printf("Data: ");
    read(0, ptr, size);
    *(long *)*ptr = *(ptr+1);
    free(ptr);
    free(ptr);
    system("/bin/sh");
    return 0;
}
```

Dreamhack의 hook 실습 문제이다.

<br/>

우선, ASLR은 스택, 힙, 라이브러리 등의 영역을 실행마다 바꾸는 것인데, 코드 영역에는 PIE가 적용되지 않으면 그대로이다. 그리고 PIE는 컴파일 시에 적용여부를 결정할 수 있지만, ASLR은 따로 서버의 설정 파일에서 보호 기법의 적용 여부를 결정한다. (/proc/sys/kernel/randomize_va_space) 이 문제는 no_PIE이기 때문에, 코드 영역의 주소값은 변하지 않으므로, 이를 이용할 수 있다고 분석할 줄 알아야한다.

## 포인터와 malloc에 대한 오개념 바로잡기

요새 20학점에 풀전공을 듣고, 저녁에는 따로 3시간정도씩 보안공부를 하다보니, 오늘따라 유난히 체력적으로 지친다..ㅜㅜ 그래서 내가 공부하면서 메모해둔 내용들을 정리하지 않고 그냥 옮겨적도록 하겠다.

<br/>

malloc의 반환형은 void* '포인터'이다. 그리고, read(0, buf, size) 에서 buf 자체가 어떤 주소를 가리키는 포인터여야하므로, read(0, *ptr, size)가 아니라, read(0, ptr, size)가 맞는 표현이다. 그럼 ptr이 가리키는 공간 (아래의 a와 대응되는 공간, 이 문제에선 heap의 공간)에 값을 입력하는 것이다. 계속 헷갈리는 부분이니까 잘 짚고 넘어가자.

<br/>

포인터 복습 : *(ptr + 1)을 *(ptr변수의 주소 + 0x8)으로 혼동한다.. 여기서 한가지 개념을 제대로 확립하고 가는데, **포인터도 하나의 변수이기 때문에, 포인터 자체의 메모리 주소가 존재한다.** 예를 들어서,

```c
int a = 10;	
int* ptr = &a;
```

이고, a의 주소가 0x1000이라 하자. ptr은 a를 가리키는 포인터 변수이고, ptr의 값은 0x1000이다. 여기서 'ptr의 스택에서의 주소가 0x1000이다'라고 혼동하면 큰일나는 것이다. ptr도 엄밀히 변수이기 때문에, ptr의 스택에서의 주소는 또 따로 존재한다. 그리고 **이것이 바로 포인터의 포인터, 즉 이중포인터이다. (드디어 이중포인터의 존재이유를 알게되었다.)** ptr 변수의 주소를 알기위해서 C에선 이중포인터로 접근해야하며,

```c
int** pptr = &ptr;
```

로 설정하면 된다. ptr의 주소가 0x2000이라 하면, 이처럼 ptr의 값과 ptr의 stack에서의 주소값은 다르다! 엄밀히 말하면 pptr도 역시 변수니까, 이에 해당하는 주소가 따로 있겠지? 이는 역시 삼중 포인터로 접근가능할것이다.

<br/>

본론으로 들어가서, *(ptr + 1)을 *(&ptr + 0x8)로 혼동하면 안된다. C언어의 약속에 따라, ptr + 1은 'ptr이 가리키고 있는 a의 주소 0x1000에서 다음 바이트인 0x1008를 참조하는 것이다. 사실 포인터 자료형에 따라서 값이 달라지는데, 여기선 개념만 짚고 넘어가자.

%p : 포인터의 주소값을 16진수로 출력

## 마치며

그때그때 공부하다가 꺠닫거나 새로 알게된 점들을 임시로 메모해두고 이를 포스팅할때 다듬어서 쓰는데, 특히 오늘 도움을 많이 받았다. 오늘 하루종일 쉬지않고 공부를 달렸더니 온몸이 뻐근하네...