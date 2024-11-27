---
layout: post
title: <Kernel> 1. Boot Process (with Arch Linux)
date: 2024-11-27 11:30:23 +0900
category: Embedded/Kernel
comments: true
---

## Boot Process

그동안 Ubuntu와 Kali Linux 위주로 리눅스를 사용하다가, 정말 가벼운 나만의 리눅스를 직접 구축해보고 싶어서 Arch Linux를 직접 구축해보았다. 구축 과정에서 직접 Partition도 나누고, Boot Loader도 설치하다보니 학부때 OS 전공과목에서 배웠던 Boot Process 개념이 다시 등장한다는 것을 깨닫게 되었다. 이번에는 Arch Linux를 기준으로 Boot Process에 대해 포스팅해보려고 한다.

### Firmware types

Firmware는 시스템의 전원이 인가되고 나서 가장 처음에 실행되는 프로그램이다. 일반 PC에서 주로 사용되는 펌웨어는 다음과 같다.

1. BIOS(Basic Input-Output System)
2. UEFI(Unified Extensible Firmware Interface)

최근에는 점점 BIOS보다 UEFI로 많이 사용되고 있다.

### System Initialization

System Initialization 과정은 BIOS와 UEFI에서 다소 차이가 있다. Arch Linux는 UEFI 기반이지만, 우선 두 가지 모두 다루겠다.

1. BIOS의 경우
    - 시스템 전원이 인가되면, **POST(Power-On Self Test)**과정을 거쳐서 하드웨어 점검을 진행한다.
    - POST 과정 이후, BIOS는 부팅에 필요한 하드웨어(Disk, Keyboard controllers, etc.)들을 초기화한다.
    - BIOS disk order에 적힌 첫번째 disk의 MBR 부분을 실행한다.
    - MBR 부트 코드에서 두 번째 단계 코드를 실행한다. 여기서 두 번째 단계 코드는 Post-MBR Gap, VBR 등 경우에 따라 다른 위치에 존재한다.
    - 두 번째 단계 코드에 진입하면, 실제 부트 로더가 실행된다.
    - 부트 로더는 OS를 로드한다.
2. UEFI의 경우
    - 시스템 전원이 인가되면, POST(Power-On Self Test)과정을 거쳐서 하드웨어 점검을 진행한다.
    - POST 과정 이후, BIOS는 부팅에 필요한 하드웨어(Disk, Keyboard controllers, etc.)들을 초기화한다.
    - UEFI Firmware는 NVRAM 안에 존재하는 boot entry를 읽어서, 어떤 disk와 partition으로부터 EFI application을 실행할지 결정한다.
    - UEFI Firmware는 EFI application을 실행한다. 여기서 EFI application은 부트 로더가 될 수도 있고, EFI boot stub을 사용하는 커널 그 자체가 될 수도 있다.

### Boot Loader

Boot Loader는 **kernel과 initial ramdisk를 실행하는 중요한 역할**을 한다. EFI boot stub, GRUB, LILO 등 굉장히 다양한 종류가 있으며, 주로 Arch Linux를 포함한 UEFI 펌웨어는 GRUB를 자주 Boot loader로 사용한다.

### Kernel

Boot Loader는 kernel이 포함되어있는 vmlinux image를 로드한다. 여기서 vmlinux는 statically linked executable file (ELF)이고, vmlinuz는 compressed된 vmlinux임을 의미한다. 관습적으로 vmlinux image를 만들때는 gzip, LZMA, bzip2 등을 사용하여 vmlinuz로 만들며, 최근에는 kernel의 크기가 커져서 bzImage(big zImage)로 제작하기도 한다. 하드 디스크의 주소 접근 한계 문제를 방지하기 위해서, Linux의 경우 사용자에게 **Boot Loader와 Kernel 관련 파일이 따로 저장된 partition을 생성**할 것을 권고하고 있다. 그리고 관습적으로 이 partition은 /boot 경로 혹은 / 경로에 마운트되는 경우가 많다.

### Initramfs

Boot Loader는 RAM에 initramfs를 임시로 적재한다. initramfs에는 **rootfs를 적재하기 위한 필수적인 모듈과 스크립트**만 담겨있다. Early userspace는 temporary rootfs가 마운트되는 동안 발생하는데, 여기서 initramfs에서 제공된 파일들이 사용된다. Early userspace에서 결국 rootfs를 마운트하기 위한 모듈과 복호화 로드 과정이 이루어진다.

<br/>

Early userspace의 마지막 단계에서 real rootfs가 /sysroot/ (systemd-based initramfs의 경우) 혹은 /new_root/ (busybox-based one의 경우)에 마운트되고 이 경로로 전환되면서 Late userspace가 시작된다. 여기서 Late userspace가 시작될 때 **real rootfs에서 init 프로그램이 실행된다.**

## 마치며

공부하다보니 펌웨어 관련 보안 과제에서 궁금했던 내용들도 많이 등장하고 해소되어서 뿌듯하다!