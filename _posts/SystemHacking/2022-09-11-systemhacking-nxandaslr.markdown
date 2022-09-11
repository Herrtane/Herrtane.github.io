---
layout: post
title: <System Hacking> 11. BOF 보호기법 (2) - ASLR 과 NX
date: 2022-09-11 20:30:23 +0900
category: System_Hacking
comments: true
---

## ASLR

Address Space Layout Randomization은 바이너리가 실행될 때 마다 스택, 힙, 공유 라이브러리 등을 임의의 주소에 할당하는 보호 기법이다. 

```
$ cat /proc/sys/kernel/randomize_va_space
```

위의 명령어를 통해 해당 리눅스에서 ASLR이 어느정도 적용되는지 확인할 수 있다.

- No ASLR (0) 
- Conservative Randomization (1) : 스택, 힙, 라이브러리, vdso 등
- Conservative Randomization + brk (2) : (1)의 영역과 brk() 시스템콜로 할당한 영역 (즉, malloc등의 함수로 할당한 영역을 의미)

## NX

No-eXecute(NX)는 실행에 사용되는 메모리 영역과 쓰기에 사용되는 메모리 영역을 분리하는 보호 기법이다. 코드 영역에 쓰기 권한이 있으면 공격자는 코드를 수정하기 쉬워지고, 스택이나 데이터 영역에 실행 권한이 있으면 Return to Shellcode 같은 공격을 시도할 수 있으므로, 이를 방지하는 기법이다. CPU가 NX를 지원한다면, 컴파일러 옵션을 통해 NX를 바이너리에 적용할 수 있고, NX가 적용되면 **코드 영역외에는 실행 권한이 없다.**

<br/>

참고로, NX의 명칭은 다양한데, 인텔에서는 XD (eXecute Disable), 윈도우에서는 DEP (Data Execution Prevention), ARM에서는 XN (eXecute Never), 그리고 AMD에서는 지금까지 사용한대로 NX라고 한다.

## 마치며

이어서 Dynamic, Static Linking에 대해 다룰 예정이다. 개념적인 정리 이후, 실습 문제에 대해서 포스팅할 수 있으면 포스팅하겠다. (과정이 길어서 시간 소요가 적지 않아서 쉽지는 않지만..ㅜㅜ)