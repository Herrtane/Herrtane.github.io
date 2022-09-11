---
layout: post
title: <System Hacking> 10. BOF 보호기법 (1) - Stack Canary
date: 2022-09-11 19:30:23 +0900
category: System_Hacking
comments: true
---

## 배경지식

CPU에는 다양한 세그먼트 레지스터가 존재한다고 했었다. 초기에는 세그먼트 레지스터로 code segment(cs), data segment(ds), extra segment(es)만 있었는데, CPU 개발자들은 여기에 두 개의 세그먼트 레지스터를 추가하기로 하면서 이름을 고민하다가, c,d,e 다음에 있는 f와 g를 사용하기로 했다.

<br/>

cs, ds, es는 CPU가 사용 목적을 명시한 레지스터인 반면, fs와 gs는 목적이 정해지지 않아 운영체제가 임의로 사용할 수 있는 레지스터이다. 리눅스는 fs를 Thread Local Storage(TLS)를 가리키는 포인터로 사용하는데, TLS에 카나리를 비롯하여 프로세스 실행에 필요한 여러 데이터가 저장된다.

## 스택 카나리

스택 카나리에 대해서는 지난 며칠동안 시행착오를 겪으면서 따로 그때그때 메모해둔 기록들이 있어서, 그 기록들을 이 포스팅에서 정리해서 합치도록 하겠다.

```s
mov	    rax, QWORD PTR fs:0x28
mov	    QWORD PTR [rbp-0x8], rax
xor	    eax, eax
;...
;...
;...
mov	    rcx, QWORD PTR [rbp-0x8]
xor 	rcx, QWORD PTR fs:0x28
je	    0x6f0 <main+70>
call	0x570 <__stack_chk_fail@plt>
```

스택 카나리는 함수의 프롤로그에서 스택 버퍼와 반환 주소 사이에 임의의 값을 삽입하고, 함수의 에필로그에서 해당 값의 변조를 확인하는 보호 기법이다. 카나리 값은 프로세스가 시작될 때 main 함수가 호출되기 전에 랜덤으로 생성되어 TLS (Thread Local Storage)에 전역 변수로 저장되고, 각 함수마다 이 값을 참조한다.

<br/>

컴파일러에서 SSP 보호기법을 적용하는 경우, **스택 배열을 사용하는 함수가 있으면** 함수의 시작 부분과 끝 부분에 아래 코드와 같이 stack_guard 체크 코드가 삽입된다.

```c
#include <stdio.h>
#include <string.h>

void func(char *s){
    char buf[16] = {};
    /* 
    long canary = stack_guard; 
    */
    strcpy(buf, s);
    /* 
    if (canary != stack_guard)
        stack_chk_fail();
    */
}

int main(int argc, char *argv[]){
	func(argv[1]);
}
```

fs 세그먼트 레지스터의 값을 rax에 저장하여 이를 다시 rbp의 8바이트 앞에 저장하고, 이 값을 canary 값이라고 한다.
(32bit는 gs:0x14, 64bit는 QWORD PTR fs:0x28을 통해 canary를 얻어온다.)
(**32bit는 canary가 4바이트**이다.)

<br/>

정확히는, 64bit기준으로 canary값의 랜덤 생성 부분은 7바이트이고, 리눅스에서는 첫 바이트가 널 바이트인, **"\x00 + 7바이트의 임의의 값"**으로 구성된다. 그 이유가 궁금해서 찾아보니, gets(), strcpy()등의 string 관련 함수들이 널바이트를 만나면 종료되는 특성을 이용해서, 공격자가 canary를 오버라이트 하는 것을 방지하기 위함이라고 하더라.

<br/>

만약 오버플로우가 발생하지 않았다면 이 값은 그대로 유지될 것이고, xor 연산에서 정상적으로 0이 나올 것이다. 만약 오버플로우가 발생했다면, 이 값도 당연히 바뀔 것이고 xor 연산에서 기존의 fs 세그먼트 레지스터의 값과 다르므로 다른 값이 도출될 것이다.

<br/>

Dreamhack 실습 문제들에서는 Canary leak 기법을 자주 사용하는데, printf 함수 등의 %s 포맷스트링은 NULL 바이트를 만날 때까지 출력해주는 특성을 이용한다. buf 배열의 끝이 NULL 바이트가 아니라면 buf 배열 밖의 메모리까지 출력할 수 있음을 이용하는 것이다.

<br/>

Canary leak으로 Canary 값을 알아낸 뒤에, payload에 Canary를 붙여서 탐지를 우회하고, Shellcode를 버퍼에 삽입한 뒤에 리턴 주소를 해당 Shellcode로 설정하는 기법을 사용해버리면 그만이다. 이를, **Return to Shellcode**라고 한다.

## 마치며

아직 갈길이 멀다. 이 우회기법을 막기위해 NX와 ASLR 보호기법이 고안되었고, 또 이 기법을 뚫는 방법이 등장했다. 이는 다음 포스팅에서 다루겠다.