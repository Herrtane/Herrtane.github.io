---
layout: post
title: <System Hacking> 05. Big endian 과 Little endian의 장단점 
date: 2022-08-23 20:20:23 +0900
category: System_Hacking
comments: true
---

## Big endian 과 Little endian이 왜 따로 존재할까

예전에 TCP/IP 소켓 프로그래밍을 공부할 때부터 가졌던 의문이었는데, 그냥 흘려보냈던 의문점을 이번에 공부하면서 구글링을 통해 해결했다. 아래는 구글링에서 붙여넣기한 내용이다. 위키피디아의 설명이 잘 되어있어서 별도의 설명을 추가하지는 않았다.

빅 엔디언은 소프트웨어의 디버그를 편하게 해 주는 경향이 있다. 사람이 숫자를 읽고 쓰는 방법과 같기 때문에 디버깅 과정에서 메모리의 값을 보기 편한데, 예를 들어 0x59654148은 빅 엔디언으로 59 65 41 48로 표현된다.

반대로 리틀 엔디언은 메모리에 저장된 값의 하위 바이트들만 사용할 때 별도의 계산이 필요 없다는 장점이 있다. 예를 들어, 32비트 숫자인 0x2A는 리틀 엔디언으로 표현하면 2A 00 00 00이 되는데, 이 표현에서 앞의 두 바이트 또는 한 바이트만 떼어 내면 하위 16비트 또는 8비트를 바로 얻을 수 있다. 반면 32비트 빅 엔디언 환경에서는 하위 16비트나 8비트 값을 얻기 위해서는 변수 주소에 2바이트 또는 3바이트를 더해야 한다. 보통 변수의 첫 바이트를 그 변수의 주소로 삼기 때문에 이런 성질은 종종 프로그래밍을 편하게 하는 반면, 리틀 엔디언 환경의 프로그래머가 빅 엔디언 환경에서 종종 실수를 일으키는 한 이유이기도 하다.

또한 가산기가 덧셈을 하는 과정은 LSB (Least Significant Bit) 로부터 시작하여 자리 올림을 계산해야 하므로, 첫 번째 바이트가 LSB인 리틀 엔디언에서는 가산기 설계가 조금 더 단순해진다. 빅 엔디언에서는 가산기가 덧셈을 할 때 마지막 바이트로부터 시작하여 첫 번째 바이트까지 역방향으로 진행해야 한다. 그러나 오늘날의 프로세서는 여러개의 바이트를 동시에 읽어들여 동시에 덧셈을 수행하는 구조를 갖고 있어 두 엔디언 사이에 사실상 차이가 없다.

## Endian 확인 실습 코드 (2026.04.29 추가)

Endian과 관련된 복습을 진행하던 중, 아주 좋은 C 예제를 통해 내가 부족했던 내용들을 LLM 답변을 정제하여서 보충하고자 한다.

```c
#include <stdio.h>
int main() {
  unsigned long long n = 0x4006aa;

  printf("Low <-----------------------> High\n");

  for (int i = 0; i < 8; i++) printf("0x%hhx ", *((unsigned char*)(&n) + i));

  return 0;
}
```

### 0x4006aa를 unsigned char로 형변환하면 0xaa만 남게 되므로, 여기서 사용하면 안되지 않는가?

핵심은 “값을 형변환”한 것과 “주소(포인터)를 형변환”한 것은 완전히 다르다는 점이다.

(1) 값 형변환의 경우

```c
unsigned long long n = 0x4006aa; 
unsigned char c = (unsigned char)n;
```

이 경우 c에는 0xaa만 남는다.

이유:

unsigned char는 1바이트(8비트)
따라서 하위 8비트만 남기고 상위 비트는 잘려나간다.
즉, 0x4006aa → 0xaa

(2) 하지만 코드에서 한 것은 값 형변환이 아니다

코드에서는 이렇게 했다:

```c
(unsigned char*)(&n)
```

이건 n의 “값”을 바꾼 게 아니라, “포인터 타입”을 바꾼 것이다.

즉, n은 여전히 8바이트짜리 값이다. 단지 그 메모리를 1바이트 단위로 읽겠다고 선언한 것뿐이다. 값이 잘린 게 아니다. 읽는 단위만 바뀐 것이다.

### h, hh는 무엇인가?

printf에서 %x는 기본적으로 이렇게 해석된다:

%x → unsigned int를 16진수로 출력
즉, 아무 수정자도 없으면 unsigned int (보통 4바이트) 로 처리한다.

printf에서 형식 지정자는 이런 구조다:

% [length modifier] [conversion specifier]

여기서

h → short
hh → char
l → long
ll → long long
즉,

%x → unsigned int
%hx → unsigned short
%hhx → unsigned char
그럼 왜 hh가 char(1바이트)냐?
C 표준에서 이렇게 정의되어 있다:

h → short
hh → signed char / unsigned char
즉 hh는 "half of half"라는 의미로,

int (보통 4바이트) → short (2바이트) → char (1바이트)

이렇게 한 단계 더 줄어든 타입을 의미한다.

### (unsigned char*)&n

&n 은 "n의 주소"다.

```c
unsigned long long *
```

를

```c
(unsigned char *)
```

로 바꾼 것. ("n이 저장된 메모리를 1바이트 단위로 읽겠다")

그래서 이런 게 가능했다

```c
((unsigned char*)&n)[i]
```

→ n의 메모리를 1바이트씩 접근

## 마치며

Dreamhack의 문제들을 풀면서 ASCII - 16진수 변환기를 사용하는데, 내가 의도하는 결과와 반대되는 결과가 나오는 경우가 있어서 찾아보니 endian의 차이였다. 그동안 개념으로만 알고있었는데, 실제로 그 차이를 경험해보니 왜 Network byte order에서 Big endian으로 통일하는지도 알 수 있었다.
