---
layout: post
title: <Embedded> 4. Embedded Serial Communication, Flash Memory Dump
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

그렇다면 데이터 통신이 이루어질 때, 그 전송 속도를 나타내는 지표는 무엇이고, 그 단위는 무엇일까? 아래에 그 설명이 있다.

- Bit Rate
    - 초당 얼마나 많은 데이터 비트를 전송할 수 있는지를 나타낸 것
    - 비트 전송 속도를 의미
    - 1000 bps = 3000bit/second
- Baud Rate
    - 초당 얼마나 많은 심볼을 전송할 수 있는지를 나타낸 것
    - 심볼 전송 속도를 의미
    - 1000 baud rate = 1초에 1000개의 심볼을 보낸다는 말
    - 심볼 (Symbol) : 물리적인 단위, 하나 이상의 비트를 포함

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

![usb_to_ttl]({{site.url}}/img/usb_to_ttl_cable.jpg){: width="300" height="300"}

### UART (Universal asynchronous receiver/transmitter)

오늘 포스팅을 작성한 목적이 바로 이 UART를 이해하기 위함이다! **UART는 병렬 데이터의 형태를 직렬 방식으로 전환하여 데이터를 전송하는 컴퓨터 하드웨어의 일종**이다. 

- 일반적으로 RS-232, RS-422, RS-485와 같은 통신 표준과 함께 사용한다. 
- 통신 데이터는 메모리 또는 레지스터에 있고, 이를 차례대로 읽어서 직렬화하여 통신하게 된다. 
- 당연히 비동기 (Asynchronous) 통신이기 때문에 별도의 Clock 동기화를 진행하지 않는다.
- USRT (Universal synchronous receiver/transmitter) 개념도 존재하며, 당연히 Clock 동기화가 추가된 개념이다.

아래는 Wikipedia에 나와있는 UART 통신 데이터 포맷이다.

![uart_format]({{site.url}}/img/uart_data_format.png)

그렇다면, 실제 UART 사례는 어떨까? 우선 장비의 회로 기판을 보게 되면, 운이 좋으면 RX, TX라고 적혀있는 핀을 바로 찾을 수 있는데, 이것이 바로 UART Pin이다. 참고로, **권장 보안 정책은 회로 기판을 구성하고 있는 요소들의 식별이 어렵도록 마스킹을 지우고, 돌출되어 있는 Pin들을 없애는 것**이라고 한다. 또, 이 Pin을 통해 통신하기 위해서 필요한 것이 위에 그림에서 첨부했던 'USB to TTL' 케이블이다.

<br/>

만약 RX, TX Pin들이 돌출되어 있지 않다면, USB to TTL 케이블의 피복을 벗겨서 접촉시키는 등의 별도의 처리를 해야 한다. 그리고, **RX, TX는 서로 엇갈리게 연결해줘야 하는데, A와 B 장치가 서로 연결한다고 하면, A의 TX는 B의 RX에, A의 RX는 B의 TX에 연결**해야 한다. 만약 성공적으로 컴퓨터와 임베디드 기기가 연결되었다면, **putty나 Xshell 같은 프로그램으로 Serial 통신을 시도**할 수 있다! 그 때, Baud rate를 맞춰주어야 하며, UART에서 이는 High/Low를 구분할 수 있는 Clock의 역할을 한다. 운이 좋다면 바로 shell을 얻어서 펌웨어 분석이 가능하지만, 그렇지 않은 경우도 많기 때문에, 이 때는 또 별도의 추가적인 처리를 해야 한다.

### JTAG (Joint Test Action Group)

그렇다면 JTAG는 무엇이며 어떻게 동작할까? **JTAG는 하드웨어 디버깅, 프로그래밍 및 테스트를 위해 설계된 직렬 프로토콜**이고, 제품을 만들 때 제품이 잘 동작하는지 테스트를 하기 위해 사용된다.

- CPU의 기계어 코드를 실행하지 않고, MCU 내부의 Flash memory나 임베디드 장치에서 CPU의 외부 Flash memory에 코드를 쓰거나 읽을 수 있다.
- 디버거가 CPU 동작과 연동하여 특정 기계어 코드 위치에서 멈추고 상태를 읽어 내부의 상태를 알 수 있다.
- 주로 TDI(Test Data In) Pin을 통해 데이터를 직렬로 입력하고, TDO(Test Data Out) Pin을 통해 데이터를 직렬로 출력하며, 테스트 장비는 TCK(클럭)과 TMS(모드)를 사용하여 여러 장치에 데이터를 순차적으로 전송한다.

### 그 외의 접근법

만약 UART, JTAG 등과 같은 장치가 없다면, MCU (Micro Control Unit)에 직접 접근해야 하는데, 어떤 블로그의 필자분에 따르자면 검정색 등딱지에 다리를 엄청 많이 가진 녀석이라고.. 어쨌든, 이 MCU의 Data sheet를 참고해서, 각 Pin의 역할을 파악한 뒤, RXD, TXD 등의 부위에 직접 연결을 해야 한다.

<br/>

이 방법도 안된다면, 회로에서 Flash memory를 납땜을 녹여서 Desoldering한 뒤, 아두이노나 라즈베리파이에 연결해서 직접 덤프를 뜨는 **Chip off**방법이 있다. 하지만, 다시 재조립한다고 무사히 동작한다는 보장이 없으므로, 이는 최후의 보루라고 할 수 있다.

### 마치며

하드웨어와 관련된 지식이라 그 동안 겁먹어서 제대로 공부하지 않았었는데, 막상 공부하다보니 생각보다 굉장히 강력하게 펌웨어나 메모리 데이터를 확보할 수 있는 방법이라는 것을 깨닫게 되었다. 특히 지금 포렌식 연구와도 상당히 밀접한 관련이 있으므로, 잘 알아두어야겠다.

아래는 실제 Chip off 방법과 예시가 사진과 함께 나와있는 해외 사이트 주소인데, 참고가 될 것 같아서 이곳에 남겨둔다.

[Hardware Hacking](https://www.tarlogic.com/blog/hardware-hacking-chip-off-for-beginners/)