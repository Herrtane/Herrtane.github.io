---
layout: post
title: <Reversing> 1. ELF 파일 구조
date: 2023-11-14 12:30:23 +0900
category: Reversing
comments: true
---

## ELF(Executable and Linkable Format) 파일 구조

![elfstructure]({{site.url}}/img/elfstructure.png)

Linux ELF 파일의 구조는 그림과 같으며, Linking View와 Execution View로 나뉜다.

1. Linking View : relocatable file의 형식으로서, Link 하기 전의 object file (*.o)는 이 View를 가진다. 즉, 나중에 Link 과정을 위해 다른 object file과 Link하기 위한 여러가지 정보를 가진다는 의미다. Program Header Table이 optional하고, Segment 대신 Section을 가진다.
2. Execution View : Link가 끝난 후에 완전히 실행 가능한 형태가 된 ELF 형식을 말한다. Section Header Table이 optional하고, Section 대신 Segment를 가진다.

## Readelf 명령어

![elfhelp]({{site.url}}/img/elfhelp.png)

리눅스에는 기본적으로 readelf라는 좋은 ELF 파일 분석 도구가 존재한다. 다양한 옵션을 통해 파일의 상세 내용을 뜯어볼 수 있고, 이를 Reversing 등의 용도로 사용할 수 있다. 나 역시 **학부연구생때 arm 아키텍쳐 기반의 ELF 파일을 Unicorn Emulator에 올리는 과정**에서 ELF 파일 분석과 약간의 Reversing 과정이 반드시 필요해서, 해당 명령어의 도움을 많이 받았었다.

## ELF Header

![elfheader]({{site.url}}/img/elfheader.png)

이 그림이 바로 ELF 파일의 전반적인 정보가 담긴 ELF Header이다. 나중에 설명하겠지만, PE Reversing 때 공부했던 PE 파일 구조와 제법 겹치는 요소가 존재한다. (애초에 결국 파일이라는 기본 원리는 동일하니까 당연한 말이다.) 각 요소에 대한 상세한 내용은 굳이 여기서 서술하지 않도록 하겠다.

## Program Header

![elfprogramheader]({{site.url}}/img/elfprogramheader.png)

![elfprogramheader2]({{site.url}}/img/elfprogramheader2.png)

Segment는 동일한 메모리 속성 (RO, RW...)을 가진 하나 이상의 Section의 집합을 말한다. Program Header는 ELF 내의 Segment들에 대한 정보와, 그 Segment들을 어떻게 Memory에 Loading해야 하는지 정보가 담겨있다. 여기서 몇가지 중요한 요소들에 대해서 설명해보자면,

1. INTERP : INTERP Segment는 프로그램 인터프리터의 이름만을 가지고 있는 Segment이다.
2. LOAD : LOAD Segment는 Load할 수 있는 프로그램 Segment이다. 위의 그림을 보면 다양한 LOAD가 존재하는데, 각 LOAD마다 다른 메모리속성을 가진 Section들의 집합으로 이루어져 있기 때문이다. Flags 속성을 보면 RE, RW 등으로 속성이 다르다.
3. NOTE : 보조적인 정보들의 위치와 크기가 저장되어 있다.
4. DYNAMIC : DYNAMIC Segment는 Dynamic Linking(System Hacking 카테고리 참조)에 사용되는 Segment이다.

## Section Header

![elfsectionheader]({{site.url}}/img/elfsectionheader.png)

Section은 특정 정보를 포함하고 있는 ELF file의 작은 조각이다. Section Header는 프로그램 메모리 레이아웃을 정의하지는 않으므로, 프로그램 실행에 필수 요구 정보는 아니다. Dreamhack 과 System hacking, 그리고 학부연구생 과제를 진행하면서 Section에 대해서는 많이 공부를 진행하였으므로, 자세한 설명은 생략하겠다.

## 마치며

원래 Reversing과 관련된 포스팅은 나중에 하거나 생략하려고 했으나, 최근 Reversing에 대한 정보를 필요할 때 마다 인터넷에서 찾아보는 일이 많아져서 아예 필요한 부분은 포스팅을 진행하려고 한다. 오랜만의 포스팅이라 어색한데, 막학기이고 시간이 많이 없어서 포스팅 진행이 조금 느리다..