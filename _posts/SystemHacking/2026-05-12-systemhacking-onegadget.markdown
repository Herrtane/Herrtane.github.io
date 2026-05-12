---
layout: post
title: <System Hacking> 25. One gadget, stdout 전역변수
date: 2026-05-12 10:30:23 +0900
category: System_Hacking
comments: true
---

## Introduction

Hoot overwrite 부분에서 복습을 진행하다보니 one gadget에 대한 부분은 따로 포스팅해두겠다고 하고 안한걸 발견해서, 이번 포스팅에서는 one_gadget에 대해서 다뤄보려고 한다.

## One Gadget

### One Gadget이란?
- **one_gadget**은 Glibc 바이너리 내부에 존재하는 특정 코드 시퀀스로,  
  `execve("/bin/sh", ...)`를 실행하여 쉘을 획득할 수 있는 가젯을 의미한다.
- 공격자는 이 가젯의 주소로 제어 흐름(RIP)을 이동시켜 **단 한 번의 점프만으로 쉘을 실행**할 수 있다.
- 작은 정수밖에 입력할 수 없는 상황 등으로 인해 **함수에 인자를 전달하기 어려울 때, (특히 `/bin/sh`같은 인자)** 매우 유용하다.

### 특징
- 단일 가젯으로 쉘 획득 가능
- ROP 체인을 길게 구성할 필요 없음

### Constraints (제약 조건)
각 one_gadget은 정상적으로 실행되기 위해 특정 조건을 요구한다.

예:
- 특정 레지스터 값 조건
  - `rax == NULL`
- 특정 스택/메모리 상태
  - `[rsp+0x30] == NULL`
  - `[rsp+0x50] == NULL`

이러한 조건이 만족되지 않으면 쉘이 실행되지 않고 프로그램이 중단된다.

### 익스플로잇 과정

1. **취약점 식별**
   - 반환 주소를 제어할 수 있는 취약점 확보
   - 예: BOF, FSB, OOB 등

2. **one_gadget 위치 찾기**
   - `one_gadget` 도구를 사용하여 libc 내부 가젯 탐색

3. **libc_base 계산**
   - leak을 이용해 libc base 주소 계산

4. **RET 덮어쓰기**
   - `libc_base + one_gadget` offset으로 반환 주소 overwrite

5. **조건 만족시키기**
   - gadget 실행 시 요구되는 constraint를 만족하도록 환경 구성

### one_gadget 사용 예시

```bash
$ one_gadget libc.so.6

0x45216 execve("/bin/sh", rsp+0x30, environ)
constraints:
  rax == NULL

0x4526a execve("/bin/sh", rsp+0x30, environ)
constraints:
  [rsp+0x30] == NULL

0xf02a4 execve("/bin/sh", rsp+0x50, environ)
constraints:
  [rsp+0x50] == NULL

0xf1147 execve("/bin/sh", rsp+0x70, environ)
constraints:
  [rsp+0x70] == NULL
```

### oneshot 예제 및 stdout 전역변수에 대하여

워게임 문제는 다음과 같다.

```c
// gcc -o oneshot1 oneshot1.c -fno-stack-protector -fPIC -pie

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
    char msg[16];
    size_t check = 0;

    initialize();

    printf("stdout: %p\n", stdout);

    printf("MSG: ");
    read(0, msg, 46);

    if(check > 0) {
        exit(0);
    }

    printf("MSG: %s\n", msg);
    memset(msg, 0, sizeof(msg));
    return 0;
}
```

문제에 적용된 보호 기법을 확인해보면, NX, PIE는 적용되어있으나, Canary가 적용되어있지 않음을 주목해보자.

```bash
$ checksec oneshot
[*] '/home/Study/Dreamhack/oneshot'
    Arch:       amd64-64-little
    RELRO:      Partial RELRO
    Stack:      No canary found
    NX:         NX enabled
    PIE:        PIE enabled
    Stripped:   No
```

문제 코드의 초반부에서 익숙한 코드가 등장하는데, 딱 보면 libc_base 주소를 구할 수 있게끔 도와주려고 하는 것을 알 수 있다.

```c
printf("stdout: %p\n", stdout);
```

여기서 주의할 점은, libc_base 주소를 계산하기 위해 추후 파이썬 코드로 stdout offset을 구할 때, **`libc.symbols['stdout']`으로 구하면 안된다**는 것이다. `stdout`은 **libc 내부의 _IO_2_1_stdout_ 를 가리키는 포인터 전역변수**이기 때문에, 다음과 같이 표현할 수 있다.

```c
FILE *stdout = &_IO_2_1_stdout_;
```

그러므로, `libc.symbols['_IO_2_1_stdout_']`으로 구해야 정확한 libc 내 offset값이 나온다.

<br/>

그 외의 풀이는 `check`변수의 값만 0으로 유지해주도록 Payload를 구성해주는것만 신경써주고, 나머지는 난이도가 쉬워서 구체적인 풀이는 생략하도록 한다.
