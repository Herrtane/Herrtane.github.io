---
layout: post
title: <System Hacking> 22. GDB 사용법 및 함수 프롤로그 분석 팁 정리
date: 2026-04-29 10:30:23 +0900
category: System_Hacking
comments: true
---

## GDB 사용법

이 카테고리에는 정말 오랜만에 글을 작성하는 것 같다. 작년까지 대학원에서는 주로 포렌식과 리버싱을 위주로 보안을 다루다보니 포너블 영역은 한동안 공부를 쉬었었는데, 올해 회사 보안팀에 들어오면서 다시 골고루 공부를 해야할 필요성을 느껴서 예전에 공부했던 드림핵을 펼쳐보고 있다.

<br/>

그러다보니 GDB를 자연스레 다시 자주 사용하게 되어서, 아예 GDB에서 자주 사용하거나 자주 까먹는 내용들을 이 곳에 한꺼번에 저장하고 그때그때 펼쳐보려고 한다. 물론 리버싱을 할 때도 동적분석을 위해 GDB를 사용하긴 했지만, 확실히 사용법을 아는 만큼 GDB가 편해지는 것 같아서 이곳에 한꺼번에 정리한다.

### 유용한 명령어

```
file ./debugee : 파일 불러오기
(No debugging symbols found in ./debugee) : -g 옵션 없이 컴파일된 경우
gdb -p <PID> : 프로세스 디버깅
entry : 진입점부터 프로그램을 분석
start : main부터 프로그램을 분석 (단, main을 찾을 수 없는 경우 entry와 동일하게 동작)
finish : 함수가 길어서 ni로 끝까지 가기 어려울 경우, 이 명령어를 통해 함수의 끝까지 한 번에 실행 가능
info r : 레지스터 정보
info b : breakpoint 정보
disable [bp num] : 해당 bp 비활성화
enable [bp num] : 해당 bp 활성화
delete [bp num] : 해당 bp 삭제
x/<포맷 및 크기 조합> : examine의 약자. 특정 주소에서 원하는 길이만큼의 데이터를 원하는 형식으로 인코딩하여 볼 수 있음.
ex) x/10gx $rsp, x/5i $rip, x/s 0x40000
telescope : pwndbg 기능. 특정 주소의 메모리가 참조하고 있는 주소를 재귀적으로 탐색하여 값을 보여줌
vmmap : 가상메모리의 레이아웃 출력
bt : backtrace. Call stack 출력
set <주소/레지스터> = <변경할 값> : 프로세스의 메모리 상태를 변경할 수 있는 명령어
print <함수명> : 함수의 주소 반환
```

### python 스크립트로 짠 코드에 대한 디버깅 방법

디버깅에는 여러 가지 방법이 있지만, 나는 **PID 기반 GDB attach 디버깅** 방식을 선호한다. 총 2개의 터미널 창이 필요하다. 우선, pwntools를 활용하여 작성한 python 스크립트의 상단에 다음과 같이 pause()를 추가한다.

```python
from pwn import *

p = process('./rop')
pause()
...
```

그리고 평소처럼 이 스크립트를 실행한다.

```
$ python3 rop_sol.py
```

그 상태로 또 다른 터미널 창에서는 아래와 같이 입력하여 일시정지된 rop_sol.py 프로세스에 Attach하는 방식으로 디버깅을 할 수 있다.

```
$ pwndbg -p $(pidof rop)
```

## 함수 프롤로그 분석 팁

종종 b main으로 중단점을 설정하고 start를 하면, main 함수의 완전 첫부분은 자동으로 지나치는 경우가 있다. 초반 함수의 프롤로그부터 보고 싶은 경우가 있는데 (특히 배열, 변수의 크기 할당을 직접 확인하고 싶거나, Stack Canary의 설정값을 확인하고 넘어가고 싶을때) 그럴 때를 대비해서 b main 말고 b* main+0로 중단점을 설정해서 더 정확히 멈추게 하자.

<br/>

한편, 이건 습관으로 들여놓으면 좋은데, 초반 main의 return address (0x7ffff7c2a1ca (__libc_start_call_main+n))와 rbp를 잘 관찰하고 해당 값들을 기억해두면, 이후 Canary leak이나 Stack을 확인하는 등의 작업에서 헷갈리지 않고 안정적으로 분석이 가능하더라. 그러니 프롤로그 부분도 신경써서 분석하도록 하자.

```s
b+ 0x4006f7 <main>       push   rbp
   0x4006f8 <main+1>     mov    rbp, rsp                            RBP => 0x7fffffffdd30 —▸ 0x7fffffffddd0 —▸ 0x7fffffffde30 ◂— ...
   0x4006fb <main+4>     sub    rsp, 0x40                           RSP => 0x7fffffffdcf0 (0x7fffffffdd30 - 0x40)
 ► 0x4006ff <main+8>     mov    rax, qword ptr fs:[0x28]            RAX, [0x7ffff7fb4768] => 0xb6f364cd65e18e00
   0x400708 <main+17>    mov    qword ptr [rbp - 8], rax            [0x7fffffffdd28] <= 0xb6f364cd65e18e00
   0x40070c <main+21>    xor    eax, eax                            EAX => 0
   0x40070e <main+23>    mov    rax, qword ptr [rip + 0x20095b]     RAX, [stdin@@GLIBC_2.2.5] => 0x7ffff7e038e0 (_IO_2_1_stdin_) ◂— 0xfbad2088
   0x400715 <main+30>    mov    ecx, 0                              ECX => 0
   0x40071a <main+35>    mov    edx, 2                              EDX => 2
   0x40071f <main+40>    mov    esi, 0                              ESI => 0
   0x400724 <main+45>    mov    rdi, rax                            RDI => 0x7ffff7e038e0 (_IO_2_1_stdin_) ◂— 0xfbad2088
```

마지막으로, 이 Disassembled code에서 Canary 설정 부분을 놓치지 말자. 저게 지나가야 Canary 값이 저장되는 것이다.
