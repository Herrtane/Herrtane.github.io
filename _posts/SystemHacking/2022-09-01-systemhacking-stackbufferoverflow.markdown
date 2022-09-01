---
layout: post
title: <System Hacking> 08. Stack Buffer Overflow (BOF)
date: 2022-09-01 22:30:23 +0900
category: System_Hacking
comments: true
---

## 용어 혼동 주의

본론을 시작하기에 앞서, Stack Overflow와 Stack Buffer Overflow는 엄연히 다른 용어임을 주의하고 가자. 전자는 스택 영역이 너무 많이 확장되어서 발생하는 버그를 뜻한다. 스택 영역의 크기가 동적으로 확장될 수는 있으나 한정된 크기의 메모리 안에서 무한히 확장될 수는 없기 때문이다.

<br/>

반면, 오늘 다룰 주제인 후자는 스택에 위치한 버퍼에 버퍼의 크기보다 많은 데이터가 입력되어 발생하는 버그를 뜻한다. 이 둘의 차이점을 기억하자.

## Stack Buffer Overflow 기법

이 기법은 정보보호분야에 있어서 가장 기본적인 기법임과 동시에 가장 중요한 이슈라고 말한다. 이 기법에 대해서 포스팅하겠다.

<br/>

BOF를 통해서

1. 데이터 변조 : 버퍼 뒤의 데이터를 변조하거나,
2. 데이터 유출 : 버퍼 뒤의 중요 데이터를 유출한다거나,
3. 실행 흐름 조작 : Calling Convention에 따르면, 함수의 호출 전에 parameter들을 레지스터 및 스택에 전달한 이후, **함수 호출 시 Return address를 스택에 push하고, rbp (즉, Stack Frame Pointer)를 push하여 스택 프레임의 기초를 만든 뒤에, rsp의 값을 빼서 rbp와 rsp 사이 공간을 새로운 스택 프레임으로 할당**한 뒤에 함수가 모두 끝나는 종결 부분에서 Return value를 rax에 전달하게 된다. 여기서 알 수 있듯이, **Return address**를 내가 원하는 흐름의 주소로 조작할 수 있는데, 이를 **RAO (Return Address Overwrite)**라고 한다.

<br/>

잠시 Dream hack의 Exploit Tech : Return Address Overwrite 강의의 실습 코드를 가져와보겠다.

```c
// Compile: gcc -o rao rao.c -fno-stack-protector -no-pie

#include <stdio.h>
#include <unistd.h>

void init() {
  setvbuf(stdin, 0, 2, 0);
  setvbuf(stdout, 0, 2, 0);
}

void get_shell() {
  char *cmd = "/bin/sh";
  char *args[] = {cmd, NULL};
  execve(cmd, args, NULL);
}

int main() {
  char buf[0x28];
  init();
  printf("Input: ");
  scanf("%s", buf);
  return 0;
}
```

위의 코드의 취약점은 scanf 함수에서 드러난다. 입력의 길이를 제한하지 않고, 공백 문자인 띄어쓰기, 탭, 개행 문자가 들어올 때 까지 계속 입력을 받는 함수이기 때문에, 현재는 쓰는 것을 권장하지 않는 함수이다.

<br/>

여기서 잠시 짚고 넘어가야 할 부분이 있다. 이 scanf 함수에 할당되는 buf는 크기가 0x28인데, 실제로 gdb를 통해 Disassemble해보면 0x30만큼 스택에 할당된다. 이유는 https://stackoverflow.com/questions/49391001/why-does-the-x86-64-amd64-system-v-abi-mandate-a-16-byte-stack-alignment 이 링크의 설명을 참조하면 된다. 요약하자면 rsp가 16바이트로 정렬되어야 하기 때문에, 0x의 끝자리가 0으로 통일되어야 한다는 것이다.

<br/>

즉, 오버플로우를 이용할 scanf의 버퍼는 rbp - 0x30에 위치하고, 그 뒤에 rbp의 SFP가 있으며, 그 뒤에 이어서 rbp + 0x8에 Return address가 저장된다고 할 수 있다. 버퍼의 첫부분과 Return address 사이에 0x38만큼의 공간을 dummy data로 채우고, 실행하고자 하는 코드의 주소를 입력하면 실행 흐름을 조작할 수 있다.

<br/>

이렇게 파악한 정보를 바탕으로, exploit에 사용할 **Payload**를 작성하여 프로그램에 전달하면 된다. 이 프로그램에서는 get_shell()의 주소를 gdb를 통해 알아낸 뒤에, 그 주소를 엔디언을 고려하여 Payload에 실으면 되는 것이다.

<br/>

참고로, Payload를 작성하는 과정은 직접 터미널에서 할 수도 있으나, **pwntools**라는 파이썬 라이브러리를 사용하여 파이썬 코드로 짜서 실행시키는 것이 훨씬 편리하고 좋은 방법이다. 구체적인 과정은 추후에 직접 문제를 포스팅할 기회가 된다면 그 때 다루도록 하겠다.

## 마치며

이렇게 BOF를 이용하여 RAO 공격을 하는 과정의 개념을 포스팅해보았다. 구체적으로 어셈블리어를 분석하고, 이에 대한 Payload를 짜서 공격하는 과정은 포스팅으로 하기에는 길어져서, 나중에 따로 문제를 푸는 시간을 가질 때 다뤄보도록 하겠다.

<br/>

이런 공격 과정을 방어하기 위한 방법이 여러개 고안되었는데, 다음 포스팅에서는 그 방법 중 첫번쨰인 Stack Canary에 대해서 포스팅하도록 하겠다.