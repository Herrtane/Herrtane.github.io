---
layout: post
title: <System Hacking> 16. 배경지식 - ELF파일, Obj파일, Symbol
date: 2022-09-28 10:30:23 +0900
category: System_Hacking
comments: true
---

## ELF(Executable and Linkable Format)

지난 포스팅에 이어서, 그동안 공부하면서 애매하게만 알고 넘어갔던 몇가지 배경지식들을 공부하면서 알게된 내용들을 정리해보려고 한다.

<br/>

main.c가 preprocessor를 거쳐서 #include <stdio.h> 파일 등의 코드를 불러와서 합친 main.i 파일이 만들어지고, 이후 compile과정을 거쳐서 main.s가, assembler를 거쳐서 main.o가 생성되었다고 하자. 보통 **여기서부터 ELF파일 (Windows에서는 PE파일)이라고 부른다.** 이 .o 파일과 여러개의 다른 .o 파일들을 Link하여 만들어진 최종 실행파일이 a.out파일이다.

<br/>

object 파일은 3가지 종류가 있다.

1. Relocatable object file : executable object file이 되기 위해 다른 Relocatable object file과 결합할 수 있는 파일
2. Executable object file : 메모리에 직접적으로 올라가서 실행될 수 있는 파일
3. Shared object file : Relocatable object file의 일종으로, Relocatable object file이면서 Run-time에 Dynamic Linking이 될 수 있는 파일

![filesectionheader]({{site.url}}/img/filesectionheader.png)

위 그림은 지난 포스팅에서도 다루었던 메모리 및 파일 섹션의 내용과 일맥상통한다.

<br/>

ELF 파일은 ELF 헤더가 제일 앞에 위치하고, program header table과 section header table이 그 뒤에 위치한다.

## Symbol

페이로드를 작성하다보면 자꾸 Symbol에 대한 이야기가 나오는데, 거창한 용어가 아니다. **하나의 object 파일 내에 존재하는 변수나 함수**를 symbol이라고 한다. 이 symbol들은 위의 파일 섹션의 곳곳에 들어있고 (int a = 1 golbal변수에 대한 symbol 내용은 data section에 있는 등) 이런 symbol들에 대한 정보를 하나로 모아놓은 테이블을 symbol table (symtab)이라고 한다.

## 마치며

참고로, Dreamhack에서 종종 쓰는 명령어중에, 

```
$ readelf -h ./test     : ELF 헤더에 대한 정보 출력
$ readelf -I ./test     : program header 정보 출력
$ readelf -S ./test     : section header 정보 출력
$ readelf -s ./test     : symbol table 정보 출력
```

라는 명령어는, 이런 ELF 파일들의 다양한 정보들을 옵션에 따라 나타내주는 명령어이다. 지난 포스팅때 다룬 objdump와 닮은 점이 많은 친구이다.