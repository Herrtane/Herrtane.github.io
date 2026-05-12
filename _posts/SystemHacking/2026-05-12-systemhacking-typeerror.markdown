---
layout: post
title: <System Hacking> 27. Type Error, SIGSEGV
date: 2026-05-12 10:32:23 +0900
category: System_Hacking
comments: true
---

## Example Code

Dreamhack의 예시 코드이다.

```c
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>
#include <unistd.h>

void alarm_handler()
{
    puts("TIME OUT");
    exit(-1);
}

void initialize()
{
    setvbuf(stdin, NULL, _IONBF, 0);
    setvbuf(stdout, NULL, _IONBF, 0);

    signal(SIGALRM, alarm_handler);
    alarm(30);
}

void get_shell()
{
    system("/bin/sh");
}

int main()
{
    char buf[256];
    int size;

    initialize();

    signal(SIGSEGV, get_shell);

    printf("Size: ");
    scanf("%d", &size);

    if (size > 256 || size < 0)
    {
        printf("Buffer Overflow!\n");
        exit(0);
    }

    printf("Data: ");
    read(0, buf, size - 1);

    return 0;
}
```

### SIGSEGV의 발생 조건

**SIGSEGV (Segmentation Fault) 는 프로세스가 접근 권한이 없는 메모리 주소에 접근할 때 OS가 보내는 신호**이다. 주로 다음과 같은 경우이다.

1. 유효하지 않은 주소로 점프 : return address를 0x4141414141414141로 덮음
2. NULL 포인터 역참조 : `*ptr (ptr == NULL)`
3. 읽기 전용 메모리에 쓰기 : 코드 영역(.text)에 write
4. 스택/힙 경계 벗어남 : 너무 깊은 재귀로 스택 소진

### read()의 size 자료형

```c
ssize_t read(int fd, void *buf, size_t count);
//                               ^^^^^^
//                            부호 없는 정수 (unsigned)
```

size는 int (부호 있는 정수)인데, size = 0을 넣으면 -1이 되고, 이 -1이 `size_t`로 캐스팅되는 순간 -1 언더플로우가 발생하는 문제가 터진다.

```
-1 (signed)  →  0xFFFFFFFFFFFFFFFF (unsigned 64bit)
             →  18446744073709551615 바이트
```

### 너무 큰 input도 안됨

신난다고 너무 큰 input을 Payload로 넣었다가 정작 쉘 자체가 동작을 안했는데, 스택 아래 메모리 영역(다른 함수 프레임, 환경변수 등)까지 덮어쓰면 read() 실행 중 자체적으로 crash가 나거나 signal handler가 실행되기 전에 프로세스가 죽어버리기 때문이다.

<br/>

따라서, 딱 Return Address를 덮어쓸 정도로만 Payload를 잘 작성하도록 하자. 구체적인 풀이는 생략한다.
