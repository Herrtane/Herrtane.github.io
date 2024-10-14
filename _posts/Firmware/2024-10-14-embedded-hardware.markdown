---
layout: post
title: <Embedded> 7. PCI, SPI vs I2C, eMMC vs NAND Flash
date: 2024-10-14 3:30:23 +0900
category: Embedded/Kernel
comments: true
---

## 하드웨어 개념 정리

역시 임베디드와 펌웨어 쪽 분야를 연구하려면 하드웨어 지식이 안나올래야 안나올 수가 없다. 바로 직전 포스팅때 다루었던 UART 관련 연구를 진행하면서 배우게 된 지식들을 정리해보려고 한다.

### PCI 버스(Peripheral Component Interconnect Bus)

![pci]({{site.url}}/img/pci.jpg)

PCI 및 PCIe 버스는 흔히 컴퓨터를 조립할 때 많이 보이는 버스이다. 위의 그림과 같은 친구이며, 그래픽 카드 같은 것들을 꽂을 때 PCIe 버스에 장착하게 된다. 과거에 ISA 버스, VESA 버스 등도 존재했으나, PCI가 확실하게 표준으로 자리잡았다고 보면 된다.

### SPI(Serial Peripheral Interface) vs I2C(Inter-Integrated Circuit)

![spi_i2c_uart]({{site.url}}/img/spi_i2c_uart.jpg)

사실 이 통신 방법들 중 UART를 제외하고는 직접 다룰 일이 많지는 않을 듯 하나, 그래도 계속해서 등장하는 용어들이기에 개념정도만 간단하게 짚고 넘어가려고 한다.

1. I2C
    - UART와는 다르게 동기식 직렬 통신 방법이다.
    - 데이터 전송 선 외에 CLK 신호를 주는 선이 존재한다.
    - 데이터 전송 선이 1개이므로 한 번에 송수신을 동시에 진행하지는 못한다. (Half-Duplex)
    - UART와는 다르게 하나의 Master에 여러 Slave 장치들을 연결할 수 있다.
2. SPI
    - I2C의 업그레이드 버전이라고 생각하면 편하다.
    - I2C와 다르게 송신(MOSI), 수신(MISO) 2가지의 통신 선을 가지고 있으므로 Full-Duplex이다.
    - I2C와는 다르게 여러 Master를 가질 수는 없다.

### eMMC(embedded Multi Media Card) vs NAND Flash

우선 이 둘의 차이점을 정리하기에 앞서, Flash 메모리 이전에 RAM과 ROM, 그리고 Flash부터 간단히 정리한 이후, 마지막에 eMMC에 대해 다루도록 하겠다.

1. RAM
    - DRAM(Dynamic RAM) : 일정한 주기로 캐패시터를 재충전(Refresh)하는 방식으로 정보를 유지하므로, SRAM에 비해 가격이 저렴한 대신 속도가 느리다.
    - SRAM(Static RAM) : Flip-Flop 논리회로를 사용하여 DRAM에 비해 빠른 속도를 가지는 대신, 가격이 비싸다. 주로 CPU 레지스터, Cache 메모리로 사용된다.
2. ROM
    - PROM(Programmable ROM) : 처음 한 번만 내용을 Write할 수 있는 ROM 메모리이다.
    - EPROM(Erasable PROM) : **자외선**을 쪼여서 데이터를 지우거나 수정할 수 있으며, 작업을 위해서는 시스템으로부터 분리하여야 한다.
    - EEPROM(Electrically EPROM) : **전기적**으로 데이터를 삭제하거나 수정할 수 있으며, 바이트 단위로 Read, Write할 수 있다. 그러나 RAM보다 Write 속도가 느리기 때문에, RAM으로 사용되기는 어렵고, 데이터를 유지하는 용도의 기억장치로써 사용된다.
    - 후술할 Flash Memory에 밀려 점점 사용 빈도가 낮아지고 있지만, 제어가 매우 쉽고 기록 후 데이터를 유지할 수 있는 수명이 길고 쓰기 가능 횟수가 Flash Memory보다 많아서 아직도 현역으로 많이 쓰인다.
3. Flash Memory
    - EEPROM에서 발전한 것으로, EEPROM보다 가격이 저렴하다.
    - 기계적인 충격, 습기, 고온에 강하다.
    - 소자의 한계로 인한 수명이 존재한다.
    - NAND형은 지우기, 덮어쓰기 모두 블록 단위로 해야 하나, NOR형은 지우기는 블록 단위로, 덮어쓰기는 셀 단위로 가능하다.
4. eMMC
    - NAND Flash Memory에 Flash Memory Controller가 결합된 것이다.
    - 저가, 저사양 휴대용 기기에 정말 많이 쓰인다.
    - 메모리 설계를 단순화하고 NAND 플래시 칩 및 제어 칩을 캡슐화하며 구성 요소가 소비하는 회로 기판의 면적을 절약한다. 
    - NAND 플래시는 단지 저장 장치 일뿐, 데이터 전송을 수행하려면 호스트 측의 컨트롤러에서만 데이터 전송을 조작 할 수 있다.

## 마치며

아직 다루어야 할 개념들이 더 많이 남아있으므로, 천천히 포스팅하도록 하겠다.