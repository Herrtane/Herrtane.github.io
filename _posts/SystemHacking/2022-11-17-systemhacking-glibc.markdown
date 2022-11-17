---
layout: post
title: <System Hacking> 21. Dreamhack 시스템 해킹 기초 강의 완료 및 Glibc 소스코드를 공부할 필요성
date: 2022-11-17 10:30:23 +0900
category: System_Hacking
comments: true
---

## 드림핵 시스템 해킹 기초 강의 완료!

2022년 8월 20일부터 시작하여 마침내 시스템 해킹 기초강의를 모두 이수하였다! 보안 입문자였던 나도 결국 큰 도약을 한번 디딜 수 있어서 정말 뿌듯하다. 이제는 그동안 공부했던 내용들을 복습하면서 구멍이 뚫렸었던 부분들을 공부하고, 아직 포스팅하지 못한 후반부 내용들을 천천히 포스팅할 계획이다. 이제는 본격적으로 Wargame과 CTF도 참여해 볼 계획이다.

## 공부를 하다보니 Glibc 공부의 필요성을 느낌

그동안은 커리큘럼 강좌에 나와있던 부분만 소화하기도 버거웠었는데, 이렇게 공부를 하다보니 결국 **컴퓨터와 커널 자체의 전체적인 흐름을 잘 알아야겠다**는 생각이 들었다. 예를 들어서, malloc이라는 것 자체가 구체적으로 소스코드로 어떻게 구현이 되어있고, 어디에 구현이 되어있는지, 그런 내용들을 공부하지 않으면 결국 공부에 한계가 생길 수 밖에 없는 것 같다.

<br/>

애초에 예전부터 **해커의 정의**는 보안을 하는 사람들보다는, 컴퓨터 자체를 깊게 파고 몰입하는 사람들이었던 만큼, 컴퓨터 자체를 깊게 공부하는 것이 곧 보안을 잘하는 길이겠다는 깨달음을 얻게 되었다. 그래서, 그동안 공부했던 내용들의 구멍을 채우면서, 본격적으로 공부했던 부분들이 어떤 소스 코드나 어떤 시스템에서 구체적으로 왔는지를 보충해서 공부할 계획이다.

<br/>

참고로, glibc와 libc의 차이점이 궁금해서 찾아보니까, **libc는 표준 C라이브러리"를 말하는 대명사고, glibc는 GNU에서 만든 libc**라고 한다. GNU C library 프로젝트는 GNU 시스템과 GNU/linux 시스템, 그리고 Linux를 커널로 사용하는 다른 많은 시스템들을 위한 핵심 library를 제공하는 프로젝트이고, 중요한 API를 제공한다. **C/C++언어로 작성된 프로그램에서 직접 사용하는 많은 저수준 구성 요소를 제공한다.** libc의 종류는 아래와 같다.

1. glibc : /lib/libc.so.6에 있는, 1992 이후로 linux에서 가장 많이 사용되는 libc
2. linux libc : 1990에 glibc 대신 잠시 등장했다가 사라진 libc
3. other C libraries : uClibc, dietlibc, musl libc

## 마치며

[https://codebrowser.dev/glibc/glibc/malloc/malloc.c.html#__libc_malloc](https://codebrowser.dev/glibc/glibc/malloc/malloc.c.html#__libc_malloc)

<br/>

위의 사이트는 glibc와 linux 커널 등의 소스코드가 담겨있는 사이트이다. 앞으로 종종 찾아보려고 한다.