---
layout: post
title: <Embedded> 8. 회로를 이해하기 위한 Hardware 지식 (with. Embedded Recipe)
date: 2024-12-05 15:30:23 +0900
category: Embedded/Kernel
comments: true
---

## AC(Alternating Current)와 DC(Direct Current)

Digital 신호는 Analog 신호의 일종이며, 대부분 DC 성분으로 이루어진 Boolean logic 값이다. Digital 신호는 한계값 또는 Threshold 이상이면 High, 그 이하면 Low로 판단한다. 또한, **DC는 Frequency가 0 혹은 low이고, 나머지 Frequency 성분을 가진 것들은 모두 AC라고 봐도 무방**하다는 것이다. 따라서, 우리가 다루는 Digital 신호 입장에서는 AC는 보통 1,0을 판별하는 데에 방해가 되는 noise로 생각할 수 있다. 따라서, 후술하겠지만 Low frequency와 High frequency를 분리해내는 것이 Filter이다.

## R(Resistor), L(Coil and Transformer), C(Capacitor)

R은 저항이며, 옴의 법칙에 의해 V=IR 공식이 성립한다. 즉, **정해진 전압에 대해 R의 크기를 적당히 조절하면 우리가 원하는 전류의 크기를 R에 흘러가게 만들 수 있다.** 일반적으로 전류는 R이 낮은 경로를 찾아가는 성질이 있다. 

<br/>

C는 캐패시터이고, dV/dt=I/C, 즉 전압의 시간에 따른 변화율이 클수록 전류를 더 잘 통과시키며, 저항이 작다. C값이 클수록 저항이 적게 느껴진다. C의 중요한 성질은 **일정한 전류를 흘리면서도 전압에 대해서는 낮은 주파수 성분의 전압은 통과를 못하게 한다. 즉 DC성분과 AC성분을 분리해낸다.**

<br/>

L은 인턱터이고, V=L*dI/dt를 만족한다. 중요한 성질은 **저주파의 전류만이 통과할 수 있다. 즉 급격한 신호의 흐름을 막는다.**

## Filter

LPF(Low Pass Filter)는 저주파 성분만 통과시키는 역할을 하는데, 보통 임베디드 시스템에서 고주파 성분은 noise 역할을 하기 때문에 고주파 성분을 걸러내는 역할이 중요하기 때문에 자주 사용된다. 구체적인 회로도 예시는 책에 적혀있으므로, 여기서는 생략한다.

## Transistor

Transistor는 Trans-Resistor의 약자로, Resistor값을 변화시킬 수 있다는 의미이다. Resistor의 용도는 **전류의 양을 조절**하는 것이므로, Transistor는 전류의 양을 원하는 대로 조절할 수 있다는 뜻이 된다. Transistor는 다음과 같이 구성된다.

1. Collector : 전자, 정공 등을 모은다.
2. Base : Transistor가 동작하게 하는 Switch 역할을 한다.
3. Emitter : 전자, 정공 등을 내보낸다.

Transistor는 크게 증폭과 스위칭 기능을 한다. 증폭 기능은 Base의 작은 변화에도 전류가 급격하게 변하는 활성 영역에서의 성질을 이용해서 Collector와 Emitter 간의 전류가 큰 폭으로 변하게 하는 기능이며, 스위칭 기능은 Base에 전류를 주거나 안주는 행위를 통해 말 그대로 스위치의 역할을 하는 기능이다.

## Logic Gate

컴퓨터구조 시간에는 바로 논리회로부터 배웠지만, 결국 논리회로의 내부는 위와 같은 Transistor와 같은 Hardware Component로 이루어져있다. 논리회로의 자세한 내용은 너무 길어지기도 하고, 이미 컴퓨터구조와 논리회로 수업때 자세히 배웠으므로 생략한다.

## Register

Register는 결국 Flip Flop의 집합이고, Flip Flop은 각가 1bit 정보를 저장할 수 있는 것들을 의미하므로, Register는 n bit 정보를 저장할 수 있다. 참고로 Flip Flop은 다음 input이 들어올 때 까지 Data Out을 유지하고 있으므로, 결국 Memory 기능을 가지고 있게 된다. 이를 RS Flip Flop이라고 한다.

## 마치며

역시 종이에 직접 요약하는 것보다 키보드로 타이핑을 하는 것이 쉬운 것 같으면서도 그림 예시를 들기가 참 어려워서 까다롭기도 하다. 책을 읽어가면서 또 필요한 내용은 간단하게 포스팅하도록 하겠다.