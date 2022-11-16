---
layout: post
title: <System Hacking> 11. BOF 보호기법 (2) - ASLR 과 NX
date: 2022-09-11 20:30:23 +0900
category: System_Hacking
comments: true
---

## Introduction

Canary에 이어서 ASLR과 NX, PIE, RELRO 등의 보호기법등은, 한마디로 프로그램이 **공격당할 수 있는 표면 자체를 여러 겹에 걸쳐서 줄여나가는 목적**으로 고안되었다. 후술하겠지만, 각각의 보호기법들에 대해 정리하자면,

1. Canary : Stack의 Buffer와 SFP 사이에 임의의 값을 추가하여 **Return Address를 보호** -> Return Address Overwrite 방어. Return To Shellcode 공격으로 카나리 우회
2. ASLR : 바이너리가 실행될 때 마다 **스택, 힙, 공유 라이브러리 등을 임의의 주소에 할당** 
3. NX : **Code segment외에 Stack, Heap, Data segment에는 실행 권한을 제거** -> Return To Shellcode 방어. Return To Library 공격으로 NX, ASLR 우회 (실행 권한이 남아있는 Code segment인 바이너리의 코드 영역과 라이브러리의 코드 영역. 비록 ASLR로 라이브러리의 주소는 무작위 주소로 매핑되나, PIE가 없다면 PLT의 주소는 고정. Return gadget 등을 활용하여 레지스터 값을 원하는 대로 조정하고, 연쇄 체인 흐름 형성.) 이를 확장하여 Return Oriented Programming 공격. 
4. RELRO : 쓰기권한이 불필요한 **Data segment에 쓰기 권한을 제거 (Partial과 Full로 나뉨)**
5. PIE : **ASLR을 Code segment에도 적용**

## ASLR

Address Space Layout Randomization은 바이너리가 실행될 때 마다 **스택, 힙, 공유 라이브러리 등을 임의의 주소에 할당**하는 보호 기법이다. 

```
$ cat /proc/sys/kernel/randomize_va_space
```

위의 명령어를 통해 해당 리눅스에서 ASLR이 어느정도 적용되는지 확인할 수 있다.

- No ASLR (0) 
- Conservative Randomization (1) : 스택, 힙, 라이브러리, vdso 등
- Conservative Randomization + brk (2) : (1)의 영역과 brk() 시스템콜로 할당한 영역 (즉, malloc등의 함수로 할당한 영역을 의미)

```
$ gcc addr.c -o addr -ldl -no-pie -fno-PIE

$ ./addr
buf_stack addr: 0x7ffcd3fcffc0
buf_heap addr: 0xb97260
libc_base addr: 0x7fd7504cd000
printf addr: 0x7fd750531f00
main addr: 0x400667

$ ./addr
buf_stack addr: 0x7ffe4c661f90
buf_heap addr: 0x176d260
libc_base addr: 0x7ffad9e1b000
printf addr: 0x7ffad9e7ff00
main addr: 0x400667

$ ./addr
buf_stack addr: 0x7ffcf2386d80
buf_heap addr: 0x840260
libc_base addr: 0x7fed2664b000
printf addr: 0x7fed266aff00
main addr: 0x400667
```

위의 실행 결과는 Dreamhack의 ASLR 강의 예시이다. 보면, Code segment에 위치한 main함수의 주소 말고는 바이너리 실행마다 주소가 바뀐다. 여기서 두가지 기억해야 할 것은,
1. 바이너리를 반복해서 실행해도 **libc_base 주소 하위 12비트 값과 printf 주소 하위 12비트 값은 변경되지 않는다**. 리눅스는 ASLR이 적용됐을 때, 파일을 page 단위로 임의 주소에 매핑하므로, page의 크기인 12비트 이하로는 주소가 변경되지 않는다.
2. **libc_base와 printf의 주소 차이는 항상 같다**. ASLR이 적용되면, 라이브러리는 임의 주소에 매핑되지만, **라이브러리 파일을 그대로 매핑하는 것이므로** 매핑된 주소로부터 라이브러리의 다른 심볼들 까지의 거리(Offset)는 항상 같다.

## NX

No-eXecute(NX)는 실행에 사용되는 메모리 영역과 쓰기에 사용되는 메모리 영역을 분리하는 보호 기법이다. 코드 영역에 쓰기 권한이 있으면 공격자는 코드를 수정하기 쉬워지고, **스택이나 힙, 데이터 영역에 실행 권한이 있으면 Return to Shellcode 같은 공격을 시도할 수 있으므로, 이를 방지하는 기법**이다. CPU가 NX를 지원한다면, 컴파일러 옵션을 통해 NX를 바이너리에 적용할 수 있고, **NX가 적용되면 코드 영역외에는 실행 권한이 없다.**

<br/>

참고로, NX의 명칭은 다양한데, 인텔에서는 XD (eXecute Disable), 윈도우에서는 DEP (Data Execution Prevention), ARM에서는 XN (eXecute Never), 그리고 AMD에서는 지금까지 사용한대로 NX라고 한다.

## 마치며

이어서 Dynamic, Static Linking에 대해 다룰 예정이다. 개념적인 정리 이후, 실습 문제에 대해서 포스팅할 수 있으면 포스팅하겠다. (과정이 길어서 시간 소요가 적지 않아서 쉽지는 않지만..ㅜㅜ)

<br/>

2022-11-16. System Hacking 기초 커리큘럼을 모두 마쳐서, 기존의 내용들을 복습하고 내용들을 보충하는 중이다!