---
layout: post
title: <Embedded> 3. Kernel Programming (2) - System Call
date: 2023-06-06 12:31:23 +0900
category: Embedded/Kernel
comments: true
---

## Kernel Programming - System Call

**System call** : OS의 커널이 제공하는 서비스에 대해 응용 프로그램의 요청에 따라 커널에 접근하기 위한 인터페이스. 보통 C/C++ 등 고급 언어로 작성된 프로그램은 직접 System call을 사용할 수 없기 때문에, 고급 API를 통해 System call에 접근한다.

![interruptvector]({{site.url}}/img/ivt.png)

- Interrupt Vector Table : 인터럽트 번호마다 인터럽트 핸들러가 매핑이 되어있음 (몇백가지가 있음)
- Trap : 소프트웨어 인터럽트로, 위의 인터럽트 넘버를 전부 기억할 수 없기 때문에, 하나의 함수로 되어있고 대신 그 인자로 IVT 번호를 받음 

![syscall]({{site.url}}/img/syscall.png)

## System Call 구현 방법

전공과목 AIoT 소프트웨어에서 커널 프로그래밍을 진행하면서 System Call을 구현했던 과정은 아래와 같다.

1. 커널 수정을 먼저 진행한다. syscall.tbl에 system call 번호와 system call 처리함수를 등록한다.
2. syscall.h에 system call 처리함수를 prototyping 한다.
3. System call 처리함수를 구현한다.
4. Kernel Re-compile을 한다.
5. Target Device에 컴파일된 Kernel file을 적용시킨다.
6. 다음으로, Application 작성을 진행한다. Library를 작성하고, 해당 Library를 Link하여 system call을 호출하는 Application을 작성한다.
7. 마지막으로, Application을 Target Device에 다운로드하여 실행시킨다.

위의 6번과정을 부연설명하는 예시 코드는 아래와 같다.

```c
// Library
#include ".../.../unistd.h" // Kernel파일에서 unistd.h가 위치한 경로 
#define __NR_mysyscall  451

int mysyscall(int n, int m){
    return syscall(__NR_mysyscall, n, m);
}

// Application.c
#include <stdio.h>
#include <unistd.h>

int main(){
    mysyscall(2,4);
    return 0;
}
```
