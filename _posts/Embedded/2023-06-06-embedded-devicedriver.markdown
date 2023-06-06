---
layout: post
title: <Embedded> 4. Kernel Programming (3) - Device Driver
date: 2023-06-06 14:31:23 +0900
category: Embedded/Kernel
comments: true
---

## IO Protection

OS 내용을 복습하자면, Hardware Protection에는 크게 3종류가 있다.

1. CPU Protection : 하나의 User Process가 무한 루프 등으로 독점하지 못하도록 Kernel이 Timer interrupt를 특정 주기마다 발생시켜서 자동으로 제어를 Kernel에게 넘기도록 감시하는 방법
2. Memory Protection : 프로세스가 자신에게 할당되지 않은 메모리 영역을 참조하려고 시도할 때 fault를 발생시키는 Protection 방법. Paging에서 valid/invalid bit를 사용하는 것도 하나의 예.
3. IO Protection : Application은 User mode에서 OS에 IO 제어를 요청하도록 하여, Dual Mode Operation을 통해 IO 제어를 Protection하는 방법. 번외로, **WiringPi와 /dev/mem**같은 경우, 원래 IO Protection에 위배되지만, Linux에서는 Super User일 경우 예외적으로 허용.

이 중 IO Protection의 경우, Device Driver의 존재 의의라고 볼 수 있다.

## Kernel Programming - Device Driver

Device Driver는 특정 디바이스를 제어하는 커널 소프트웨어이다. Application Program은 특정 하드웨어에 대한 제어를 커널에 요청하고, 커널이 해당 Device Driver를 호출하여 처리한다. 이때, Application Program은 file IO system call을 통해 Device Driver에 요청하고, special device file을 통해 특정 디바이스를 지정한다.

### Special Device File

Linux는 시스템의 디바이스를 /dev/ 디렉토리 내의 Special File를 통해 접근한다. **Regular file의 목적이 데이터를 저장**하는 데 있다면, **이들은 디바이스에 대한 정보 (디바이스 타입, 주 번호, 부 번호)를 제공**한다.

```
mknod /dev/ttyS4 c 4 68
```

- mknod : 드문 경우, 사용자가 직접 장치 파일을 만들어줘야 하는 경우가 생기는데, 이럴 때 mknod 명령어를 통해 직접 장치 파일을 생성한다.
- p는 FIFO 파일, c는 character device, b는 block device이다.
- Character device : Keyboard, console과 같이, Device를 파일처럼 접근하여 직접 RW를 수행한다. Character 단위로 데이터 송수신.
- Block device : HDD, CD-ROM과 같이, File system을 기반으로 Random access가 가능한 디바이스이다. Block 단위로 데이터 송수신.
- 주 번호, 부 번호

### Module Programming

- 리눅스 커널이 부팅된 후 커널 모듈을 동적으로 추가하거나 제거할 수 있으며, 별도의 재부팅이나 커널 컴파일이 필요하지 않다.
- Device Driver의 경우, 리눅스 커널의 DDI (Device Driver Interface)에 준하여 작성되어야 한다.

```
insmod
rmmod
lsmod
depmod
modprobe
```

```c
static struct file_operations fops = {
    .owner          = THIS_MODULE,
    .open           = led_open,
    .release        = led_release,
    .write          = led_write,
    .unlocked_ioctl = led_ioctl,
};
```

위의 fops struct는 내가 수업시간 과제때 작성했던 Raspberry Pi Led GPIO 제어 Device Driver의 코드 중 일부이다. 해당 struct에 내가 구현할 Interface를 기입해야 한다. 그 외의 자세한 코드는 기회가 될 때 다루도록 하겠다.
