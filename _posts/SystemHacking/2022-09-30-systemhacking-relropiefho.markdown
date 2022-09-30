---
layout: post
title: <System Hacking> 17. RELRO, PIE, 그리고 HO(Hook Overwrite)
date: 2022-09-30 20:30:23 +0900
category: System_Hacking
comments: true
---

## RELRO (Relocation Read-Only)

프로세스의 데이터 세그먼트를 보호하는 RELocation Read-Only(RELRO)는 쓰기 권한이 불필요한 데이터 세그먼트에 쓰기 권한을 제거하는 방어 기법이다.

<br/>

RELRO는 RELRO를 적용하는 범위에 따라 두 가지로 구분되는데, 하나는 RELRO를 부분적으로 적용하는 **Partial RELRO**이고, 나머지는 가장 넓은 영역에 RELRO를 적용하는 **Full RELRO**이다.

1. Partial RELRO : .init_array, .fini_array, .got만 쓰기권한 제거. .got.plt, .data, .bss는 쓰기권한 유지. (.got.plt는 Partial RELRO에서만 생성)
2. Full RELRO : .got전체 쓰기권한까지 제거. .data, .bss에만 쓰기권한 유지. 라이브러리 함수들의 주소가 바이너리의 로딩 시점에 모두 바인딩되므로 Lazy binding을 사용하지 않고, .got에 쓰기권한이 부여되지 않음.

.init_array, .fini_array영역에는 프로세스의 시작, 종료에 실행할 함수들의 주소가 저장되어있으므로, 공격자가 임의로 값을 넣는다면 프로세스의 실행 흐름이 조작될 위험이 있다.

<br/>

.got는 전역변수 중에서 실행되는 시점에 바인딩 되는 변수 (now binding)들이 위치해있으므로, Partial RELRO를 통해 쓰기 권한을 제거한다. .got.plt는 실행중에 바인딩되는 변수 (lazy binding)들이 저장되고, Partial RELRO가 적용된 바이너리에서는 대부분의 함수들의 GOT가 여기에 저장된다.

## PIE (Position-Independent Executable)

PIE는 ASLR이 코드영역에도 적용되게 해주는 기술이다. 원래는 보안성 향상 목적으로 고안된 기술이 아니지만, 어쩌다보니 ASLR과 함께 보안을 강화하는 기술이 되었다. 

<br/>

리눅스의 ELF는 지난 포스팅에서 Executable object와 Shared Object가 있다고 했다. 이중, 후자는 기본적으로 재배치가 가능하며, rip를 기준으로 데이터를 Relative Addressing하기 때문에 어느 주소에 적재되어도 코드의 의미가 훼손되지 않는다. 이를 **PIC (Position-Independent Code)** 라고 부른다.

<br/>

ASLR이 도입되기 전에는 리눅스 실행파일들이 재배치를 고려하지 않고 만들어졌지만, 도입 이후에는 재배치를 원했고, 그러나 기존의 파일들을 뜯어고칠수는 없었기 떄문에, SO를 실행파일로 사용하기로 한 것이다. 이들을 PIE라고 한다.

## HO (Hook Overwrite)

위에서 Full RELRO의 경우, 기존의 GOT overwrite 기법이 막히게 되므로, 새로운 공격방법을 모색해야 하는데, 그 중 라이브러리에 위치한 **hook**을 이용하는 공격기법이 생겨났다. 대표적으로 malloc, realloc, free함수를 호출할 때, 함수의 초반부에서 동적 메모리의 할당과 해제 과정에서 발생하는 버그를 디버깅하기 위해 __malloc_hook, __free_hook 등의 포인터 존재여부를 검사한다. 이 변수들은 **libc.so에서 쓰기 가능한 영역에 위치**하므로, libc가 매핑된 주소를 알때 이 변수를 조작하고 malloc, realloc, free 등을 호출하여 실행흐름을 조작할 수 있다.

```c
// Name: fho.c
// Compile: gcc -o fho fho.c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() {

  char buf[0x30];
  unsigned long long *addr;
  unsigned long long value;

  setvbuf(stdin, 0, _IONBF, 0);
  setvbuf(stdout, 0, _IONBF, 0);

  puts("[1] Stack buffer overflow");
  printf("Buf: ");
  read(0, buf, 0x100);
  printf("Buf: %s\n", buf);

  puts("[2] Arbitrary-Address-Write");
  printf("To write: ");
  scanf("%llu", &addr);
  printf("With: ");
  scanf("%llu", &value);
  printf("[%p] = %llu\n", addr, value);

  *addr = value;

  puts("[3] Arbitrary-Address-Free");
  printf("To free: ");
  scanf("%llu", &addr);

  free(addr);

  return 0;
}
```

위의 코드는 Dreamhack의 Free Hook Overwrite 실습 예제이다. 이 포스팅에서는 이 문제를 해결하는 코드보다는, 문제를 해결하는 과정에서 내가 고민했던 내용들을 적어보려고 한다. 익스플로잇 코드는 생략한다.

<br/>

**PIE 적용 문제에서 ROP가 힘든이유** : PIE가 적용안되면 다른 영역의 주소는 계속 변화해도 main함수의 주소는 같아서, 고정된 주소와 코드가젯을 활용하여 익스플로잇 코드에 가젯의 주소를 직접 적어넣고 ROP를 수행할 수 있었지만, ASLR이 적용된 상태에서 만약 PIE가 적용되면 ASLR이 코드영역에도 적용되어서 기존 방법으로는 익스플로잇이 불가능해진다.

<br/>

이럴 때, one gadget을 사용하는 방법이 가능하다면, 이 방법을 쓸 수 있다. one gadget에 대한 내용은 생략하겠다. one_gadget을 이용해 libc 내의 one-shot gadget의 오프셋을 구할 수 있다.

<br/>

그리고, 순간 Buf가 read(0,buf,0x100)함수 내에서 생긴다고 착각해서
read함수에 해당하는 스택프레임의 rbp-0x40의 위치가 buf인줄알고.. BOF를 일으키면 read함수의 main으로의 반환주소가 leak되는거로 착각했다.. 정말 바보같다.. 애초에 buf는 main함수에서 선언되는 놈이다.

<br/>

교훈 : 이번 혼동을 해결하기위해 gdb사용법과 스택 분석을 엄청 진행하면서 상당히 좋은 공부를 했다. gdb를 어영부영 사용하지 말고, 확실히 익혀두자. (x/10wx $rsp같은 것)

## 마치며

Payload를 짜는 과정에서는 큰 난항이 없었고, 위에서 적었듯이 몇가지 개념들에서 혼동이 생겨서 시간이 걸렸던 문제라 Payload는 생략한다. 어쨌든, 이번 포스팅을 기점으로 더이상 컴파일에 의도적인 보호기법 해제를 적용하지 않는, RELRO, NX, Canary, PIE가 모두 적용된 온전한 파일을 분석할 수 있는 기초적인 수준까지 배우게 되었다..!