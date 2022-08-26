---
layout: post
title: <System Hacking> 07. Syscall과 execve, 그리고 Shell code
date: 2022-08-26 15:30:23 +0900
category: System_Hacking
comments: true
---

## Shell Code란?

내가 Dreamhack에서 몇일의 시간에 걸쳐서 shell code 개념 전체의 흐름을 이해한 내용을 정리해보려고 한다.

<br/>

x86-64 architecture의 어셈블리어에는 다양한 명령어들이 있다. 그 중, syscall이라는 명령어는 유저모드에서 커널모드의 시스템 소프트웨어에게 특정 동작을 요청하기 위해 사용된다. 소프트웨어 대부분은 커널의 도움이 필요하기 때문에 이 명령어는 상당히 자주 사용된다.

<br/>

syscall은 함수이기 때문에, 각 레지스터의 값을 인자로 사용하는데,
rax를 통해 read(0x00), write(0x01), execve(0x3b) 등의 요청 종류를,
rdi, rsi, rdx, rcx, r8, r9, stack 순으로 필요한 인자를 전달한 뒤에,
syscall 명령어를 어셈블리어로 호출하면 함수처럼 실행이 된다.

<br/>

이 중에, execve 요청은 인자로 전달된 filename이 가리키는 파일을 실행하는 요청이다.
이 요청을 통해서 sh, bash등의 shell을 실행하게되면 해당 컴퓨터의 shell을 그대로
획득하게 되는데, 이를 통해 원하는 목적을 달성할 수 있게 된다.

<br/>

이 일련의 과정을 어셈블리어로 작성한 것을 (필요에 따라 이후 nasm 등으로 기계어로
바꾼 16진수의 숫자의 조합을) shell code라고 한다.

```
test
```