---
layout: post
title: <System Hacking> 12. Dynamic and Static Link / PLT, GOT (2026.04.30 수정)
date: 2022-09-11 21:30:23 +0900
category: System_Hacking
comments: true
---

## 잠시 적어둘 내용들

- libc : 리눅스에서의 표준 C라이브러리
- checksec : 내가 보고자 하는 바이너리에 어떤 보호 기법이 적용되어있는지 알려주는 도구. 아주 좋은 도구.
- gdb 사용시에 무턱대로 start하지 말고, disassemble main 명령어로 전체 명령어를 쉽게 확인하는 습관을 가지자.
- Compile 전과정에 대한 정리
    1. pre-compiler를 통해 i파일 생성
    2. compiler를 통해 s파일 생성
    3. assembler를 통해 o파일 (우리가 아는 오브젝트 파일) 생성
    4. linker를 통해 라이브러리 등 필요한 다른 o파일을 link시켜서 최종적인 실행파일 (exe) 생성

## Static Link vs Dynamic Link

정적 링크는 링크때 바이너리에 정적 라이브러리의 모든 함수가 포함된다. 따라서 해당 함수를 호출할 때, 라이브러리를 참조하는 것이 아니라, 자신의 함수를 호출하는 것처럼 호출할 수 있게 된다.다만, 여러 바이너리에서 라이브러리를 사용할 경우, 그 라이브러리의 복제가 중복되어 여러번 이루어지므로 용량을 낭비하게 된다.

<br/>

동적 링크된 바이너리를 실행하면 동적 라이브러리가 프로세스의 메모리에 매핑된다. 그리고 실행 중에 라이브러리의 함수를 호출하면, 매핑된 라이브러리에서 호출할 함수의 주소를 찾아서 실행한다. 이 과정에서 **PLT(Procedure Linkage Table)와 GOT(Global Offset Table)**이 사용된다. 다음의 코드 예시를 보자.

```c
#include <stdio.h>

int main() {
  printf("Hello world!\n");
  return 0;
}
```

이 코드를 다음과 같이 동적, 정적 링크를 하여 GDB로 비교를 해보면, puts를 호출하는 부분에서 차이가 발생한다.

```
gcc -o static hello.c -static
gcc -o dynamic hello.c -no-pie
```

정적 링크를 통해 컴파일한 바이너리의 경우, puts을 호출하는 부분에서 실제 puts가 존재하는 주소를 직접 호출한다.

```s
b+ 0x4018bd <main+8>         lea    rax, [rip + 0x7d74c]     RAX => 0x47f010 ◂— 'Hello world!'
   0x4018c4 <main+15>        mov    rdi, rax                 RDI => 0x47f010 ◂— 'Hello world!'
 ► 0x4018c7 <main+18>        call   puts                        <puts>
        s: 0x47f010 ◂— 'Hello world!'

   0x4018cc <main+23>        mov    eax, 0                      EAX => 0
   0x4018d1 <main+28>        pop    rbp
   0x4018d2 <main+29>        ret
```

반면, 동적 링크를 통해 컴파일한 바이너리의 경우, puts를 호출하는 부분에서 실제 puts의 주소가 아닌 puts의 plt 주소를 호출한다.

```s
b+ 0x40113e <main+8>      lea    rax, [rip + 0xebf]     RAX => 0x402004 ◂— 'Hello world!'
   0x401145 <main+15>     mov    rdi, rax               RDI => 0x402004 ◂— 'Hello world!'
 ► 0x401148 <main+18>     call   puts@plt                    <puts@plt>
        s: 0x402004 ◂— 'Hello world!'

   0x40114d <main+23>     mov    eax, 0                 EAX => 0
   0x401152 <main+28>     pop    rbp
   0x401153 <main+29>     ret
```

## Runtime resolve

바이너리가 실행되면 ASLR에 의해 라이브러리가 임의의 주소에 매핑되고, 이 상태로 라이브러리 함수를 호출하면 함수의 이름을 바탕으로 라이브러리에서 심볼들을 탐색하고, 해당 함수의 정의를 발견하면 그 주소로 실행 흐름을 옮기게 된다. 이 과정을 **runtime resolve**라고 한다. 다만, 이 과정이 매번 이루어진다면 비효율적일 것이므로, ELF는 GOT라는 테이블을 두고, 한번 resolve된 함수의 주소는 이 테이블에 등록했다가 추후 다시 호출되면 저장된 주소를 꺼내서 사용한다. 아래의 코드는 Dreamhack의 강의자료에서 가져온 코드이다. 이 코드를 보면서 설명을 이어가겠다.

```c
#include <stdio.h>

int main() {
  puts("Resolving address of 'puts'.");
  puts("Get address from GOT");
}
```

```s
 ► 0x40113e <main+8>     lea    rax, [rip + 0xebf]     RAX => 0x402004 ◂— "Resolving address of 'puts'."
   0x401145 <main+15>    mov    rdi, rax               RDI => 0x402004 ◂— "Resolving address of 'puts'."
   0x401148 <main+18>    call   puts@plt                    <puts@plt>

   0x40114d <main+23>    lea    rax, [rip + 0xecd]     RAX => 0x402021 ◂— 'Get address from GOT'
   0x401154 <main+30>    mov    rdi, rax               RDI => 0x402021 ◂— 'Get address from GOT'
   0x401157 <main+33>    call   puts@plt                    <puts@plt>

   0x40115c <main+38>    mov    eax, 0                 EAX => 0
   0x401161 <main+43>    pop    rbp
   0x401162 <main+44>    ret
```

### resolve되기 전

먼저 got.c를 컴파일하고, 실행한 직후에 GOT를 확인해보면 아직 puts의 주소를 찾기 전이므로, 함수의 주소가 아닌 PLT 내부의 주소가 적혀있다. (참고로 GOT는 코드가 아니라 주소값이 저장된 데이터 영역이므로, 제대로 보려면 x/gx 0x404000 로 보아야한다.)

```
pwndbg> got
Filtering out read-only entries (display them with -r or --show-readonly)

State of the GOT:
GOT protection: Partial RELRO | Found 1 GOT entries passing the filter
[0x404000] puts@GLIBC_2.2.5 -> 0x401030 ◂— endbr64

pwndbg> x/gx 0x404000
0x404000 <puts@got.plt>:        0x0000000000401030
```

여기서 한가지 유의깊게 봐야할 부분이 생기는데, .got는 global 변수를 저장하는 주소여서 지금 우리가 찾는 목적과는 관련이 없고, .got.plt가 우리가 찾고자 하는 Lazy binding용 GOT 영역이라는 것이다. 최근 커널로 바뀌면서 .got와 .got.plt가 별도로 존재하기 때문에, 헷갈려서는 안된다.

```
.text
  puts@plt

.got
  global 변수 주소

.got.plt
  puts 실제 주소 (lazy binding 대상)
```

<br/>

PLT에서는 먼저 puts의 GOT에 쓰인 값으로 실행 흐름을 옮긴다. 이후 **GOT에 아직 puts에 대한 실제 주소가 없기 때문에, 다시 puts@plt+n의 주소로 이동**한다. 아래의 GDB 내용을 보면, puts의 GOT 주소인 rip + 0x2fb6 (= 0x404000)으로 jmp를 하였지만, 0x404000에는 아직 실제 put의 주소가 없기 때문에 다시 puts@got.plt 주소인 0x401030으로 이동하는 것을 확인할 수 있다.

```s
   0x401040       <puts@plt>                        endbr64
   0x401044       <puts@plt+4>                      jmp    qword ptr [rip + 0x2fb6]    <0x401030>
    ↓
 ► 0x401030                                         endbr64
   0x401034                                         push   0
   0x401039                                         jmp    0x401020                    <0x401020>
    ↓
   0x401020                                         push   qword ptr [rip + 0x2fca]
   0x401026                                         jmp    qword ptr [rip + 0x2fcc]    <_dl_runtime_resolve_xsavec>
```

그 이후 **dl_runtime_resolve_xsavec라는 함수가 실행**되는데, 이 함수에서 puts의 주소가 구해지고 (**PLT테이블이 필요한 이유**이다. GOT로 JMP하는 기능 외에도 함수의 주소를 구하는 데에 필요한 코드가 구현되어있다.), **최종적으로는 GOT에 실제 주소를 저장한 후, 해당 주소로 점프**하게 된다.

### resolve된 후

```s
b+ 0x40113e <main+8>     lea    rax, [rip + 0xebf]     RAX => 0x402004 ◂— "Resolving address of 'puts'."
   0x401145 <main+15>    mov    rdi, rax
   0x401148 <main+18>    call   puts@plt                    <puts@plt>

   0x40114d <main+23>    lea    rax, [rip + 0xecd]     RAX => 0x402021 ◂— 'Get address from GOT'
   0x401154 <main+30>    mov    rdi, rax               RDI => 0x402021 ◂— 'Get address from GOT'
 ► 0x401157 <main+33>    call   puts@plt                    <puts@plt>
        s: 0x402021 ◂— 'Get address from GOT'

   0x40115c <main+38>    mov    eax, 0                 EAX => 0
   0x401161 <main+43>    pop    rbp
   0x401162 <main+44>    ret
```

두 번째로 puts를 호출할 때는 GOT에 puts의 주소가 쓰여있어서 바로 puts가 실행될 수 있다. 아래 디버깅 결과를 보면 자명하다.

```s
   0x401040       <puts@plt>      endbr64
 ► 0x401044       <puts@plt+4>    jmp    qword ptr [rip + 0x2fb6]    <puts>
    ↓
   0x7ffff7c87be0 <puts>          endbr64
   0x7ffff7c87be4 <puts+4>        push   rbp
   0x7ffff7c87be5 <puts+5>        mov    rbp, rsp     RBP => 0x7fffffffdd30 —▸ 0x7fffffffdd40 —▸ 0x7fffffffdde0 —▸ 0x7fffffffde40 ◂— ...
```

이번에는 GOT에 명확하게 puts의 주소가 담긴 것을 확인할 수 있다.

```
pwndbg> x/gx 0x404000
0x404000 <puts@got.plt>:        0x00007ffff7c87be0
pwndbg> got
Filtering out read-only entries (display them with -r or --show-readonly)

State of the GOT:
GOT protection: Partial RELRO | Found 1 GOT entries passing the filter
[0x404000] puts@GLIBC_2.2.5 -> 0x7ffff7c87be0 (puts) ◂— endbr64
```

## 마치며

오늘 내용은, 조만간 다룰 GOT overwrite라는 공격기법에서 중요하게 사용되는 개념이다. Return Oriented Programming에서 이 공격기법을 다루기 때문에, 개념을 따로 정리해두었다. 다음 포스팅부터는 내 체감상 이해하는데 시간이 걸린 부분들이 대거 등장할 예정이다.

<br/>

2022.11.18 추가. [https://bob3rdnewbie.tistory.com/190](https://bob3rdnewbie.tistory.com/190)

.plt, .got 섹션이 과거에는 정확히 각각 Text, Data section에 존재했지만, 지금은 .got plt, .rel plt 가 생기면서 개념이 애매해졌는데, 위의 링크에서 해당 내용이 잘 설명되어있어서 추가한다. 그리고, 실제 readelf 명령어를 통해 해당 섹션의 모습을 잘 분석했고, Dynamic linking 과정을 gdb를 통해 잘 분석해놓아서 추후 종종 참고하기 위해 링크를 첨부하였다.

2026.04.30 추가. 

다시 복습을 해보니, 실제 GDB로 분석한 사례가 있으면 이해하기가 더 좋겠다고 생각하여 자세하게 GDB를 통해 실습한 내용을 추가하였다. 훨씬 보기 편해지고, 실습 내용을 추가하면서 더 확실하게 내용을 파악할 수 있어서 좋았다.
