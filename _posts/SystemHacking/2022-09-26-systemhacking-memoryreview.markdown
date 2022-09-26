---
layout: post
title: <System Hacking> 15. Memory Layout 복습, 몇가지 Linux 배경지식
date: 2022-09-26 22:30:23 +0900
category: System_Hacking
comments: true
---

## Process Memory Layout 복습

리눅스에서의 **'프로세스'** 메모리 구조에 대해 확실히 짚고 넘어가려고한다. 자꾸 헷갈리는 부분이 있어서..

![memorylayout]({{site.url}}/img/memorylayout.png)

Dreamhack에서 가져온 출처의 그림이다. 위의 주소가 0x0000... 아래의 주소가 0xffff.... 이다. 그림이 짤려서 말로 설명을 추가한다.

1. Code segment : Text segment라고도 부름. 실행 가능한 기계 코드가 담겨있음. rw-
2. Data segment : data segment에는 실행하면서 값이 변할 수 있는 전역변수가 위치. rodata segment에는 값이 변하면 안되는 전역상수가 위치.
3. BSS segment : BSS (Block Started by Symbol segment)에는 컴파일 시점에 값이 정해지지 않은 전역변수가 위치. 프로그램이 시작될 때 모두 값이 0으로 초기화됨. rw-
4. Stack segment : 함수의 인자나 지역변수가 위치. 자세한 내용은 생략.
5. Heap segment : 동적 할당되는 데이터가 위치. 자세한 내용은 생략.

## Linux에서 익숙해져야 할 배경지식

### objdump

```
$ objdump -d ./example          : 해당 바이너리를 disassemble
$ objdump -f ./example          : 해당 바이너리의 파일 헤더 확인
$ objdump -h ./example          : 해당 바이너리의 섹션 헤더 확인
```

### /proc에 대하여

- /proc : 시스템의 프로세스 정보를 담고 있는 디렉토리
- /proc/[pid] : 해당 pid의 정보
- /proc/self : 현재 실행중인 프로세스의 디렉토리 표시
- /proc/[pid]/maps : 해당 프로세스가 매핑된 메모리의 주소 공간

### gcc 컴파일러 사용시 주의사항

gcc 컴파일러 사용시, 

```
$ gcc -o test.c test
```

라고 작성할 경우, 대참사가 일어난다. -o 옵션이 output이라는 옵션이기 때문에, -o test.c 의 의미는 test.c라는 파일에 컴파일 output의 결과가 overwrite된다는 의미이기 때문이다. 결과적으로 기존의 old 파일들을 지우게된다. **순서를 절대적으로 신경쓰자**

```
$ gcc -o test test.c : 바람직한 예시
```

## 마치며

이외에도 Dreamhack을 공부하면서 기본적으로 알아야할 배경지식을 앞으로 계속 추가하려고한다. 진도를 나가다보니 헷갈리는 개념들이 많아서 쉬어갈겸 정리해보았다.