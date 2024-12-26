---
layout: post
title: <Embedded> 2. Kernel Programming (1) - Introduction
date: 2023-06-06 12:30:23 +0900
category: Embedded/Kernel
comments: true
---

## Kernel Programming의 개요

Kernel Programming은 Kernel Mode에서 동작하는 프로그램을 작성하는 것을 말하며, 커널 프로그래밍을 해야 되는 이유는 다음과 같다.

1. Kernel Core 기능 추가 : System call 등을 추가하기 위함
2. Kernel Algorithm 개선
3. Kernel Module Programming : Device driver 등을 추가하기 위함. (**별도의 커널 컴파일 불필요**)

### Kernel vs Application Program

우선, 수행 시기에서 차이가 있다. Application Program은 순차적으로 수행하지만, Kernel Program은 응용 프로그램이 호출한 System call 이나 interrupt handler를 수행하기 위해 비동기적으로 수행한다. 참고로, OS 시간에 배웠지만, Kernel이 호출되는 경우는 아래의 3가지이다.

1. Boot
2. Interrupt
3. System Call

<br/>

또한, 당연하게도 Application Program은 User Mode에서 동작하므로 Device/Memory에 대한 접근이 제한되지만, Kernel Program은 Kernel Mode에서 동작하므로 제약이 없으며, 그 만큼 위험 부담도 있다.

<br/>

마지막으로, Application Program과 Kernel Program은 각각 User address space와 Kernel address space를 별도로 쓰기 때문에, (이후 포스팅에서 다루겠지만) 두 프로그램 사이에서 포인터 등의 주소를 다루는 데이터를 주고 받을 때는 

```c
#include <linux/uaccess.h> 
int copy_to_user(void __user* to, const void* from, unsigned long n);
int copy_from_user(void* to, const void __user* from, unsigned long n);
```

와 같은 함수를 통해 주고 받아야 한다.

### Kernel Programming 유의사항

Kernel Programming 시에 주의할 사항이 있는데, 우선 리눅스 커널이 권고하고 있는 규칙대로 이름을 지어야 한다. Application Program에서는 현재 개발하고 있는 프로그램에서만 각 함수와 변수의 name을 구별해주면 되지만, **Kernel Program에서는 현재 개발하는 커널 module + 커널 core 전체에서 함수와 변수의 name이 충돌하면 안된다**. 

1. 외부 파일과 link하지 않을 모든 심볼은 static으로 처리하고, 
2. 외부 파일과 link할 심볼은 symbol table에 EXPORT_SYMBOL(name); 형태로 등록한다.
3. 전역 변수는 sys_open() 등의 잘 정의된 prefix를 사용해야 한다. 

<br/>

또한, Application Program은 모든 라이브러리를 Link하고 사용할 수 있으나, Kernel Program은 kernel에서 export하는 것만 사용 가능하다.

```c
#include <usr/include/linux>
#include <usr/include/asm>
```

위의 두 header file만을 사용한다. 참고로, 

1. static 변수와 global 변수의 공통점 : 프로그램이 실행되는 동안 고정된 메모리 주소를 차지
2. 차이점 : 해당 함수 내에서만 참조 가능한지, 코드 어디서나 참조 가능한지
3. static global 변수 : 다른 파일에서 extern 참조 불가 (소프트웨어 공학 차원에서 중요한 습관)
4. function 앞에도 static 붙이는 것이 사실 좋다

이외에도 Kernel Panic이나 Fault handling 등을 매우 유의하며 작성해야 한다.