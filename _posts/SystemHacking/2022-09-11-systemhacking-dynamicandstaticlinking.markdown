---
layout: post
title: <System Hacking> 12. Dynamic and Static Link / PLT, GOT
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

동적 링크된 바이너리를 실행하면 동적 라이브러리가 프로세스의 메모리에 매핑된다. 그리고 실행 중에 라이브러리의 함수를 호출하면, 매핑된 라이브러리에서 호출할 함수의 주소를 찾아서 실행한다. 이 과정에서 PLT(Procedure Linkage Table)와 GOT(Global Offset Table)이 사용된다.

<br/>

바이너리가 실행되면 ASLR에 의해 라이브러리가 임의의 주소에 매핑되고, 이 상태로 라이브러리 함수를 호출하면 함수의 이름을 바탕으로 라이브러리에서 심볼들을 탐색하고, 해당 함수의 정의를 발견하면 그 주소로 실행 흐름을 옮기게 된다. 이 과정을 **runtime resolve**라고 한다. 다만, 이 과정이 매번 이루어진다면 비효율적일 것이므로, ELF는 GOT라는 테이블을 두고, 한번 resolve된 함수의 주소는 이 테이블에 등록했다가 추후 다시 호출되면 저장된 주소를 꺼내서 사용한다. 아래의 코드는 Dreamhack의 강의자료에서 가져온 코드이다. 이 코드를 보면서 설명을 이어가겠다.

```c
#include <stdio.h>

int main() {
  puts("Resolving address of 'puts'.");
  puts("Get address from GOT");
}
```

### resolve되기 전

먼저 got.c를 컴파일하고, 실행한 직후에 GOT를 확인해보면 아직 puts의 주소를 찾기 전이므로, 함수의 주소가 아닌 PLT 내부의 주소가 적혀있다.

<br/>

PLT에서는 먼저 puts의 GOT에 쓰인 값으로 실행 흐름을 옮긴다. 이후 dl_runtime_resolve_xsavec라는 함수가 실행되는데, 이 
함수에서 puts의 주소가 구해지고 (**PLT테이블이 필요한 이유**이다. GOT로 JMP하는 기능 외에도 함수의 주소를 구하는 데에 필요한 코드가 구현되어있다.), GOT에 주소가 쓰여진다.

### resolve된 후

두 번째로 puts를 호출할 때는 GOT에 puts의 주소가 쓰여있어서 바로 puts가 실행될 수 있다.

## 마치며

오늘 내용은, 조만간 다룰 GOT overwrite라는 공격기법에서 중요하게 사용되는 개념이다. Return Oriented Programming에서 이 공격기법을 다루기 때문에, 개념을 따로 정리해두었다. 다음 포스팅부터는 내 체감상 이해하는데 시간이 걸린 부분들이 대거 등장할 예정이다.