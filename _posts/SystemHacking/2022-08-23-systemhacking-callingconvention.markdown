---
layout: post
title: <System Hacking> 06. Calling convention
date: 2022-08-23 20:30:23 +0900
category: System_Hacking
comments: true
---

## 함수호출규약 (Calling convention)

함수를 호출할 때는 반환된 이후를 위해 
1. Caller의 Stack frame 및 Return Address를 저장
2. Caller는 Callee가 요구하는 인자를 전달
3. Callee의 실행이 종료될 때는 Return Value를 전달

일반적으로 이는 컴파일러의 몫이므로 프로그래머가 High Level Language로 코드를 작성할 때는 Calling Convention을 명시하지 않아도 되지만, 컴파일러의 도움 없이 어셈블리 코드를 작성하거나 읽을 때는 반드시 알아야 한다.

x86 architecture와 x86-64 architecture는 각각 다른 Calling Convention을 사용하며, x86은 대표적으로 **cdecl**, x86-64는 대표적으로 **SYSTEM V**를 사용한다.

### x86 : cdecl

x86 architecture는 레지스터의 수가 적으므로, 

1. 스택을 통해 인자를 전달 : 마지막 인자부터 첫 번째 인자까지 거꾸로 스택에 push
2. Caller가 이 스택을 정리

### x86-64 : SYSTEM V (SYSV)

1. 6개의 인자를 rdi, rsi, rdx, rcx, r8, r9에 순서대로 저장하여 전달
2. 더 많은 인자를 사용할 때는 스택을 추가로 이용
3. Caller가 이 스택을 정리
4. Return Values는 rax로 전달

이를 직접 gdb를 통해 분석할 수 있다.

## 마치며

GDB에 대한 사용법이나 어셈블리어에 대한 설명은 당장 포스팅하지는 않을 예정이다. 추후 필요에 의해 필요한 부분에 한해서 포스팅하도록 하겠다.

