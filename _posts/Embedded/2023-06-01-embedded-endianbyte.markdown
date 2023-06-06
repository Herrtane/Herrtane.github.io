---
layout: post
title: <Embedded> 1. 바이트코드, 인코딩, 그리고 학부연구생 이야기
date: 2023-06-01 10:30:23 +0900
category: Embedded/Kernel
comments: true
---

## 진행중인 학부연구생 이야기

한참 반년 전에 드림핵과 관련된 포스팅을 하다가, 올해 1월부터 교내 임베디드 보안 관련 연구실에서 학부연구생을 시작하면서 포스팅을 못했었다. 그동안 정말 바쁘게 지냈고, 많이 힘든 순간들도 있었는데, 그 과정 속에서 많은 성장이 있었다.

<br/>

조만간 기말고사 기간이라 포스팅을 길게 작성할 수는 없지만, 어쨌든 연구라는 것도 배워가고 있고, 혼자서 공부하기 힘든 여러 가지 주제들을 접해보면서 정말 많은 배움을 얻고 있다. 특히, 컴퓨터 구조와 시스템, 그리고 임베디드와 관련된 부분에서 내가 얼마나 부족한 학부생인가를 학부연구생을 하면서 많이 깨닫고 있다. 조만간 종강을 하면, 더 긴 이야기로 포스팅을 적기로 하겠다.

## 바이트 코드, 엔디안, 인코딩에 대한 보다 자세한 복습

정보보호에서 CRC32 Table을 다루거나, Unicorn Emulator에서 Input byte code 등을 넣는 등 이번 학기 전공 과목과 연구 진행 과정에서 자주 나오는 개념들인데 계속 헷갈려서 아예 한번에 다시 정리하고 넘어가기로 하자.

### ASCII, 16진수

0x is used for literal numbers. "\x" is used inside strings to represent a hex ascii character.

```
>>> 0x41 
65
>>> "\x41"  # "\x41" == chr(65)
'A'

>>> "\x01"  # a non printable character
'\x01'
```

<br/>

ASCII는 1byte, 즉 8bit의 데이터를 사용한다. 16진수 숫자 하나는 4bit의 이진수로 나타낼 수 있기 때문에, ASCII는 8bit, 즉 총 2개의 16진수 숫자를 사용하여 나타내는 것이다.

### 인코딩과 Python, 바이트코드

Python2는 ASCII를, Python3는 UTF-8이 디폴트 방법으로 되어 있으므로, Python2와는 다르게 **Python3에서는 모든 문자열이 자동적으로 유니코드로 인코딩 처리** 되므로 한글을 더 편하게 사용할 수 있다. 

<br/>

Python의 encode()함수는 특정 문자열을 원하는 인코딩 방식으로 처리하여 utf-8, ascii 형식의 Byte 코드로 변환해주는 함수인 것이다. 그렇다면 Byte 코드는 무엇일까? 바로 **컴퓨터가 처리하는 데이터 형식인 바이너리 (2진수) 데이터를 코드화한 것**을 말한다. bytes는 **1바이트 단위의 값을 연속적으로 저장하는 시퀀스 자료형**이다(보통 1바이트는 8비트로 정의하며 0~255(0x00~0xFF)까지 정수를 사용).

```
>>> b = b'abcde'
>>> b
b'abcde'

>>> print(b)
b'abcde'

>>> type(b)
<class 'bytes'>

>>> s = b'abc def ghi'
>>> s.split()
[b'abc', b'def', b'ghi']

>>> s = 'Vi er så glad for å høre og lære om Python!'

>>> b = s.encode('utf-8')
>>> b
b'Vi er s\xc3\xa5 glad for \xc3\xa5 h\xc3\xb8re og l\xc3\xa6re om Python!'

>>> b.decode('utf-8')
'Vi er så glad for å høre og lære om Python!'
```

조금 더 자세한 (연산 등) 내용은 아래 링크를 참고하자.

<br/>

[파이썬 바이트코드 추가설명 링크](https://reo91004.tistory.com/238)

### 엔디안

엔디안(endian) : 바이트의 저장 순서

<br/>

메모리의 저장 단위는 8bit (16bit)인데, 실제로 저장해야 되는 바이트는 32bit (64bit) 이다. 그러므로, 어쩔 수 없이 1바이트씩 총 4번 (8번) 나눠서 저장해야 되는데, 이때 **여러 바이트에 나누어서 저장하는 순서를 결정하는 것이 바로 엔디안**이다. 

<br/>

예를 들어, 0x12345678을 저장한다고 하면, 총 4바이트의 저장 공간이 필요하다. 이 숫자를 0x1F4589번지에 저장한다고 하면, 

```
0x1F4589 : 78
0x1F4590 : 56
0x1F4591 : 34
0x1F4592 : 12
```

로 저장하는 방식이 Little Endian 방식이다. 즉, **메모리의 작은 주소에 LSB(Least Significant Bit, 가장 오른쪽의 비트)가 배치되는 방식**이다. 

<br/>

내가 CRC32 Reverse Lookup 과제에서 리틀엔디안 문제로 골머리를 앓았던 이유에 대해서는 형변환 개념이 같이 들어가는데, 이는 아래의 링크에서 확인하도록 하자.

[형변환과 리틀엔디안 링크](https://www.devkuma.com/docs/c/%ED%8F%AC%EC%9D%B8%ED%84%B0%EC%9D%98-%ED%98%95%EB%B3%80%ED%99%98/)

```c
#include <stdio.h>

int main() {
  int iCount , iArray[2] = { 0x02040810 , 0x20408000 };
 unsigned char *cp = (unsigned char *)iArray;

  for(iCount = 0 ; iCount < 8 ; iCount++)
   printf("cp + %d = %d\n" , iCount , *(cp + iCount));

  return 0;
}
```

## 마치며

반년동안 공부한 내용들이 정말 많아서 모든 내용을 전부 포스팅하기는 어려울 것 같고, 그럴 필요성도 높지 않다. 주로 내가 실수하거나 별도로 정리가 필요한 개념들에 한해서 시간이 날때 가끔씩 포스팅하도록 하겠다.