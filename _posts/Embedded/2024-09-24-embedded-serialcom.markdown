---
layout: post
title: <Embedded> 4. Embedded Serial Communication
date: 2024-09-24 12:31:23 +0900
category: Embedded/Kernel
comments: true
---

## Serial Communication

학부연구생때부터 현재 석사까지 드론과 관련된 연구를 많이 접하다 보니, 자연스럽게 JTAG, UART, Serial 통신과 같은 용어들을 많이 접하게 되었고, 심지어 내부 플래시의 데이터 추출을 실험하기 위해서 Serial 통신을 직접 공부해야 하는 상황에 자주 놓이게 되었다. 그래서 이번 기회에 Serial 통신과 관련된 이론적인 기반 지식들을 정리하고자 한다.

### Introduction

임베디드 시스템에서는 각종 프로세서와 회로들끼리 통신하는 것이 필요한데, 이때 표준 규격이 필요하다. 대표적인 표준 통신 규격이 바로 Serial Communication이다. 마치 웹에서 프로토콜과 같은 개념이다. 

### Serial vs Parallel

결국 한국어로 해석하면 직렬 통신이라고 해석할 수 있는데, 서로 다른 컨트롤러를 직렬로 연결하여 통신하는 방법중의 하나라고 보면 된다. 병렬 통신과의 차이점은 아래와 같다.

- Serial Communication
    - 데이터를 한번에 1bit씩 보내는 방식
    - 비용이 적게 소요됨
    - 하나의 선만을 사용하므로 하드웨어적으로 구현하기가 쉬움
    - 데이터를 모으고 취합하는 과정을 구현해주어야 함
- Parallel Communication
    - 데이터를 한번에 8bits에서 16bits 정도 보내는 방식
    - 비용이 크게 소요됨
    - 선이 많이 필요하므로 하드웨어적으로 구현하기가 불편함

Transmitter가 이진 펄스 형태로 데이터를 보내면 Receiver가 높고 낮음을 구분하여 데이터를 읽는다.

![serial_communication]({{site.url}}/img/serial_com1.png)

### Clock Synchronization

이렇게 통신하기 위해서는 Transmitter와 Receiver의 Clock이 동기화되어야 한다. **Clock이란 어떠한 시점에서 이진 펄스를 논리 정보로 읽을 것인지에 대한 시점**을 말한다. 가령, Transmitter에서 10110011을 Receiver로 보냈는데, Receiver에서 Clock을 Transmitter와 다른 속도로 해석한다면, 10110011이 아닌 다른 해석이 발생할 수 있다. 그러므로 Clock Synchronization이 반드시 필요한데, 인터페이스의 종류에 따라 동기와 비동기로 구분된다.

- 동기식 시리얼 통신
    - 각각의 송수신 주체 사이에 데이터를 보내는 선 말고도 Clock 주기를 맞추는 Clock 선을 하나 더 둠으로써, 미리 보내는 Clock에 서로 타이밍을 맞춘 뒤 데이터를 전송하는 방식이다.
- 비동기식 시리얼 통신
    - 따로 주기를 정하지 않고, 시작과 끝을 알려주고 전송하는 방식이다. 따라서 별도의 Clock 선을 두지 않는다.
    - 동기식 시리얼 통신에 비해 안정성이나 속도가 느리다.

### Bit Rate (BPS) vs Baud Rate



### PIN

임베디드 기기를 다루는 데 자주 등장하는 하드웨어 핀 용어들이 있다. 아래와 같이 정리하기로 한다. 참고로, VCC에서 GND로 전류가 흐르는 건, **전류는 항상 전위가 높은 곳에서 낮은 곳으로 흐른다**는 원리에 따른 것이다.

- VCC (Voltage of Common Collector)
    - 공통 콜렉터용 전압. 보통 5V
    - 건전지의 역할
- GND (Ground)
    - 접지. 보통 0V
    - 철판의 역할
- TX (Transmitter)
- RX (Receiver)

![usb_to_ttl]({{site.url}}/img/usb_to_ttl_cable.jpg)
