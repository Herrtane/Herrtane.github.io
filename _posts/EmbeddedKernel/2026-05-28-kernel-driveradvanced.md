---
layout: post
title: <Kernel> 3. Device Driver 개념 정리
date: 2026-05-28 10:30:23 +0900
category: Embedded/Kernel
comments: true
---

## Driver의 기본 구조

Linux Driver의 가장 기본적인 뼈대는 다음과 같다.

```c
#include <linux/module.h>
#include <linux/init.h>

MODULE_LICENSE("GPL");

static int __init my_init(void) { return 0; }
static void __exit my_exit(void) {}

module_init(my_init);
module_exit(my_exit);
```

### char device 등록 3단계

만약 Character Device Driver의 경우, 다음과 같은 3단계의 등록 절차를 거친다. 이 3단계가 없으면 `/dev/` 파일이 안 만들어져서 유저스페이스에서 접근 불가하다.

```c
// 1단계: major번호 할당 + file_ops 등록
register_chrdev(0, "mydev", &file_ops);  // 0 = 자동 할당

// 2단계: 클래스 생성 (/sys/class/myclsname/)
class_register(&my_cls);

// 3단계: 디바이스 파일 생성 (/dev/mydev)
device_create(&my_cls, NULL, MKDEV(major, 0), NULL, "mydev");
```

### file_operations

char 디바이스 드라이버라면 file_operations 구조체를 구현하는 것은 사실상 필수이다. 유저스페이스가 `/dev/` 파일을 통해 드라이버와 통신하는 유일한 수단이기 때문이다. 단, 드라이버 종류에 따라 세부적인 내용은 차이가 있다.

```c
static struct file_operations file_ops = {
    .owner          = THIS_MODULE,  // 필수: 모듈 참조 카운트 관리
    .open           = device_open,  // open() 시스템콜
    .release        = device_release, // close() 시스템콜
    .read           = device_read,  // read() 시스템콜
    .write          = device_write, // write() 시스템콜
    .unlocked_ioctl = device_ioctl, // ioctl() 시스템콜 ← 핵심
    .llseek         = device_llseek,
    .mmap           = device_mmap,  // 메모리 매핑 (고급)
    .poll           = device_poll,  // select/poll (고급)
};
```

### User-Kernel 데이터 교환

유저와 커널 사이는 서로 모드가 다르므로, 데이터를 교환하기 위해서는 별도의 함수를 사용해야 한다.

```c
// 유저 → 커널
copy_from_user(dst_kernel, src_user, size);

// 커널 → 유저
copy_to_user(dst_user, src_kernel, size);

// 접근 가능한 메모리인지 먼저 확인
access_ok(user_ptr, size);
```

### IOCTL 정의

아래의 매크로는 명령 코드 하나만으로 "이 드라이버 것이 맞는지", "얼마나 읽어야 하는지"를 알 수 있게 하기 위한 약속된 인코딩 방식이다. 헤더 파일을 유저스페이스와 커널 드라이버가 공유하면, 양쪽이 동일한 상수를 사용하게 되어 mismatch가 없어진다.

```c
#define MYDEV_TYPE  'M'   // 고유 magic number (Documentation/userspace-api/ioctl/ioctl-number.rst 참고)

#define MY_IOCTL_NOARG    _IO(MYDEV_TYPE, 0)          // 인자 없음
#define MY_IOCTL_WRITE    _IOW(MYDEV_TYPE, 1, int)    // 유저→커널
#define MY_IOCTL_READ     _IOR(MYDEV_TYPE, 2, int)    // 커널→유저
#define MY_IOCTL_RDWR     _IOWR(MYDEV_TYPE, 3, int)   // 양방향

 31      29 28    16 15     8 7       0
┌──────────┬─────────┬────────┬────────┐
│ direction│  size   │  type  │ number │
│  (2bit)  │ (14bit) │ (8bit) │ (8bit) │
└──────────┴─────────┴────────┴────────┘

#define MY_IOCTL_WRITE  _IOW('M', 1, int)
//                            │   │   └── sizeof(int) = 4  → size 필드에 4
//                            │   └────── 1             → number 필드에 1
//                            └────────── 'M' = 0x4D    → type 필드에 0x4D
//                       _IOW  → direction = 01 (write, 유저→커널)
//
// 결과: 0x40044D01  (컴파일 타임 상수)
```

## Driver의 실행 흐름

### init() 함수 이후의 흐름?

`xxx_init()`은 초기화만 하고 리턴한다. 이후 드라이버는 이벤트 기반으로 수동 대기 상태가 되며, 유저스페이스의 요청에 따라 함수가 호출된다. 핵심은 `xxx_init()` 자체는 아무 반복/대기도 하지 않고 즉시 리턴하며, 이후의 모든 함수 호출은 커널 VFS 레이어가 유저의 파일 오퍼레이션을 받아 `file_operations` 구조체의 함수 포인터를 통해 간접 호출하는 방식으로 동작한다는 것이다.

```
유저스페이스          커널 VFS                드라이버
─────────────────────────────────────────────────────
open("/dev/testlkm") 
    │
    └──────────────► chrdev lookup
                     major번호로 file_ops 찾기
                     file_ops.open 함수포인터 호출 ──► device_open()
                                                        return 0
                     ◄───────────────────────────────
    ◄──────────────  fd 반환
```

```
[유저스페이스 애플리케이션 or 퍼저]
         │
         │  open("/dev/testlkm")
         ▼
    device_open()
         │  → open_count 증가, module 참조 획득
         │
         │  ioctl(fd, TESTLKM_IOCTL_*, arg)
         ▼
    device_ioctl()
         │  → magic 검증 (_IOC_TYPE)
         │  → cmd 분기
         │
         ├─ TESTLKM_IOCTL_NO_ARGS    → ioctl_test_0()  [이슈 시뮬레이션]
         ├─ TESTLKM_IOCTL_WRITE_*    → ioctl_test_1~5() [유저→커널 데이터 수신]
         ├─ TESTLKM_IOCTL_READ_*     → ioctl_test_6~10() [커널→유저 데이터 반환]
         └─ TESTLKM_IOCTL_RDWR_*    → ioctl_test_11~15() [양방향]
         │
         │  read(fd, buf, size)
         ▼
    device_read()   [로그만 출력, 실제 데이터 없음]
         │
         │  write(fd, buf, size)
         ▼
    device_write()  [로그만 출력]
         │
         │  close(fd)
         ▼
    device_release()
         │  → open_count 감소, module 참조 해제
         │
         │  cat/echo /sys/class/testcls/testlkm/attr*
         ▼
    attr*_show() / attr*_store()  [sysfs read/write]
         │
[모듈 언로드: rmmod testlkm]
         ▼
    testlkm_exit()
         │  → sysfs 제거 → device_destroy → class_unregister → unregister_chrdev
```

## 계산기 예시

```c
// calc_drv.h

#ifndef CALC_DRV_H
#define CALC_DRV_H

#include <linux/ioctl.h>

#define CALC_MAGIC  'C'

/* 유저→커널: 두 수를 보냄 */
struct calc_input {
    int a;
    int b;
};

/* 커널→유저: 결과를 받음 */
struct calc_output {
    int result;
};

#define CALC_IOCTL_ADD  _IOWR(CALC_MAGIC, 1, struct calc_input)
#define CALC_IOCTL_MUL  _IOWR(CALC_MAGIC, 2, struct calc_input)

#endif
```

```c
// calc_drv.c

#include <linux/module.h>
#include <linux/fs.h>
#include <linux/uaccess.h>
#include "calc_drv.h"

MODULE_LICENSE("GPL");

static int major;
static struct class calc_cls = { .name = "calc_cls" };
static struct device *calc_dev;

static long calc_ioctl(struct file *filp, unsigned int cmd, unsigned long arg)
{
    struct calc_input  in  = {0};
    struct calc_output out = {0};

    if (_IOC_TYPE(cmd) != CALC_MAGIC) return -ENOTTY;

    /* 유저 공간에서 입력 읽기 */
    if (copy_from_user(&in, (void __user *)arg, sizeof(in)))
        return -EFAULT;

    switch (cmd) {
    case CALC_IOCTL_ADD:
        out.result = in.a + in.b;
        break;
    case CALC_IOCTL_MUL:
        out.result = in.a * in.b;
        break;
    default:
        return -ENOTTY;
    }

    /* 결과를 유저 공간에 돌려줌 */
    if (copy_to_user((void __user *)arg, &out, sizeof(out)))
        return -EFAULT;

    return 0;
}

static int calc_open(struct inode *inode, struct file *filp)    { return 0; }
static int calc_release(struct inode *inode, struct file *filp) { return 0; }

static struct file_operations calc_fops = {
    .owner          = THIS_MODULE,
    .open           = calc_open,
    .release        = calc_release,
    .unlocked_ioctl = calc_ioctl,
};

static int __init calc_init(void)
{
    major = register_chrdev(0, "calc", &calc_fops);
    class_register(&calc_cls);
    calc_dev = device_create(&calc_cls, NULL, MKDEV(major, 0), NULL, "calc");
    return 0;
}

static void __exit calc_exit(void)
{
    device_destroy(&calc_cls, MKDEV(major, 0));
    class_unregister(&calc_cls);
    unregister_chrdev(major, "calc");
}

module_init(calc_init);
module_exit(calc_exit);
```

```c
// user_app.c

#include <stdio.h>
#include <fcntl.h>
#include <sys/ioctl.h>
#include <unistd.h>
#include "calc_drv.h"   // 드라이버와 동일한 헤더 공유

int main(void)
{
    int fd = open("/dev/calc", O_RDWR);
    if (fd < 0) { perror("open"); return -1; }

    struct calc_input in = { .a = 7, .b = 3 };

    /* 덧셈 요청 */
    ioctl(fd, CALC_IOCTL_ADD, &in);
    struct calc_output *out = (struct calc_output *)&in;
    printf("7 + 3 = %d\n", out->result);  // 10

    /* 곱셈 요청 */
    in.a = 7; in.b = 3;
    ioctl(fd, CALC_IOCTL_MUL, &in);
    printf("7 * 3 = %d\n", out->result);  // 21

    close(fd);
    return 0;
}
```
