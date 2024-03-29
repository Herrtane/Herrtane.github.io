---
layout: post
title: <System Hacking> 04. 보안 공부 재개 및 x86-64 Architecture 
date: 2022-08-22 13:20:23 +0900
category: System_Hacking
comments: true
---

## Long time no see!

2022년 7월 부로 군복무를 완전히 마쳤다! 가장 최근 블로그 포스팅이 정보처리산업기사 실기 합격이었던 것으로 기억하는데, 그 동안 공부 외적으로 나를 돌보는 시간들을 가졌었다. 여행도 다니고, 주변 못만났던 지인들도 많이 만나고, 이런 저런 인생 선배분들을 찾아뵈면서 내 진로에 대한 자문도 구했고, 게임도 정말 원없이 했다! (Apex Legend라는 게임에서 처음으로 골드를 달성해보았다!) 

<br/>

사실 작년 이맘때쯤 보안 공부에 대한 자신감도 떨어지고, 어렸을 때부터 보안을 쭉 공부해왔던 사람들이 많다고 해서 내 스스로가 보안쪽으로는 가면 안되나.. 하는 생각 때문에 한동안 컴퓨터공학 공부에만 집중했었다. 그렇게 보안을 포기하나 싶었는데, 계속 마음 한켠에서 보안 공부를 하고 싶다는 미련이 남더라. 그래서 군대를 전역한 이후에 같은 학교에 보안을 공부하는 동기에게도 진지하게 상담했었고, 다양한 정보를 찾아보면서 다시 한번 보안 공부를 시작해보기로 했다. 

<br/>

목표는 정보보호대학원이다. 다른 사람들보다 많이 뒤쳐진 건 사실이고, 내가 목표를 잘 이룰 수 있을지는 미지수지만, 그래도 계속 고민만 하기 보다는, 해볼 수 있을 만큼 해보고, 그래도 안되면 그 때 포기하기로 했다. 정말 다행인건, 그동안 계속 공부했던 컴퓨터공학 전공들이 보안공부에 있어서 필수적인 요소이기 때문에, 긍정적으로 생각해보면 보안을 공부하기 위한 기반을 그동안 매우 탄탄하게 다졌다고 볼 수 있다!

<br/>

이제 복학하면서 학과 공부도 최선을 다하면서, 보안 공부도 열심히 병행해나가보려고한다. 이렇게 하다가 정말 아닌거같다 싶으면, 이제 그때는 깔끔하게 보안을 내려놓고 게임, 웹 개발 등 내가 기존에 꿈으로 삼았던 분야를 다시 이어나가겠다.

<br/>

최근에 Dreamhack이라는 아주 좋은 보안 학습 사이트를 발견했다. 정말 감사하게도 별도의 복잡한 비용이나 절차 없이 보안을 막 시작하는 사람들부터 보안을 깊게 파는 사람들까지 수준별로 학습할 수 있는 내용을 제공해주더라. 시스템 해킹, 웹 해킹, 리버싱 등 각 분야에 대한 강의부터 다양한 Wargame과 CTF까지 풀어볼 수 있는 사이트라서 감사하다. 당분간 블로그 포스팅은 이 사이트에서 공부했던 내용 중 기억할 만한 내용들이나 배울만한 부분을 추려서 포스팅 할 예정이다.

## x86-64 Architecture에 대해

시스템 해킹을 다루기 앞서, 컴퓨터구조에서 배웠던 내용 중 필요한 내용들을 복습해보자. Von Neumann architecture가 가장 대중적인 컴퓨터 구조인 지금, 이 구조를 작동하게 하는 명령어들을 모아놓은 집합을 ISA (Instruction Set Architecture)라고 한다. ISA에는 여러 종류가 있지만, 내가 다루는 ISA는 인텔의 x86-64 구조이다. 고성능을 위한 구조이기 때문에 발열이 심해서 소형 기기들보다는 PC에 주로 사용되는 대중적인 구조이다. 64비트 아키텍쳐이기 때문에, WORD 당 64비트로 이루어져 있으며, General Register, Segment Register, Instruction Pointer Register, Flag Register가 존재한다.

### General Register

- rax (accumulator register) : 함수의 반환값 저장
- rbx (base register) : x64에선 주 용도가 없음
- rcx (counter register) : 각종 반복 연산의 시행 횟수 저장
- rdx (data register) : x86에선 주 용도가 없음
- rsi (source index), rdi (destination index) : 데이터를 옮길 때 원본, 목적지를 가리키는 포인터
- rsp (stack pointer) : 사용중인 스택의 위치를 가리키는 포인터
- rbp (stack base pointer) : 스택의 바닥을 가리키는 포인터
- r8 ~ r15
- IA-32때는 r이 아닌 e로 시작했으며, IA-16때는 e가 없었다. 각 레지스터의 절반을 기준으로 명칭이 확장된다. 예를 들어, rax의 하위 절반 비트는 eax, eax의 하위 절반 비트는 ax, ax는 다시 ah와 al로 나뉜다.

### Segment Register

### Instruction Pointer Register

- rip : CPU가 현재 어느 부분의 코드를 실행할지 가리키는 포인터

### Flag Register

- CF (Carry Flag) : 부호 없는 수의 연산 결과가 비트의 범위를 넘을 경우
- ZF (Zero Flag) : 연산 결과가 0일 경우
- SF (Sign Flag) : 연산 결과가 음수일 경우
- OF (Overflow Flag) : 부호 있는 수의 연산 결과가 비트의 범위를 넘을 경우
- 이외에도 여러 Flag Register 존재.

## 마치며

컴퓨터공학 전공때 배웠던 내용들이 대거 출현해서 상당히 반가우면서도, 그때 내용들이 실제 보안에서 이렇게 사용된다는 것을 알게되고, 배웠던 내용들이 하나의 유기적인 흐름으로 연결되는 신기한 경험을 해서 뿌듯하다.