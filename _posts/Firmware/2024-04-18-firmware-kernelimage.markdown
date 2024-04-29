---
layout: post
title: <Firmware> 2. Kernel image
date: 2024-04-29 14:30:23 +0900
category: Firmware
comments: true
---

## Kernel image

Firmware를 분석하다보면, 다양한 Kernel image를 마주하게 된다. 현재 Kernel image에 대한 공부와 정리가 필요하여, 이에 대한 정보를 조사하고 정리하기로 하자.

<br/>

The Linux Kernel (or any other OS Kernel) is just a binary image containing machine code for the target architecture. It is kinda like a statically linked executable, because there's no operating system to link any dependecy before it's running,so that once loaded in the main memory, it is able to execute without any other helper. This doesn't mean that it cannot load any other module dynamically. In Linux, this behavior is easily seen when you load a module from userspace (it's different process from loading a .so file tough).

<br/>

Kernel image formats differ based on compression, architecture, and specific use cases. Let’s look at the various Linux kernel images we have.


### vmlinux

The original, uncompressed Linux kernel image is called vmlinux. vmlinux is the kernel in a uncompressed and non-bootable form. It’s the intermediate step to producing vmlinuz. It has debugging symbols included along with the complete and unmodified kernel code.
In most cases, vmlinux is used to develop kernels, debug, and analyze them. The vmlinux image should be bootable before being loaded to an operating system kernel. To make it bootable, we add a multi-boot header, boot sector, and set up routines.
The “vm” preceding the Linux stands for virtual memory. In Linux, we can use a portion of the hard disk space as virtual memory, hence the name “vm”.

### vmlinuz

vmlinuz is a compressed Linux kernel image file for booting the Linux operating system. When we compress the vmlinux file we create vmlinuz. The compression uses the gzip algorithm, resulting in a smaller file size than the uncompressed vmlinux.
The resulting file contains the kernel’s essential components. Compression reduces file size, optimizing boot efficiency and memory usage during boot.
When we’re booting the system, the boot loader reads the vmlinuz file from the boot device and decompresses it into memory. The decompressed kernel image is run from the memory.

### vmlinux.bin

The vmlinux.bin file is an uncompressed binary image generated during Linux kernel source code compilation. It includes the entire compiled kernel code, with debug symbols and additional information for debugging and analysis.
The vmlinux.bin image file is not directly executable and is too big for practical use. As a result, developers use it to understand and analyze the Linux kernel behavior.

### zimage

zimage refers to a distinct compressed kernel image file format. It’s designed for X86-based systems. It addresses the limitations of older bootloaders that couldn’t handle sizeable compressed kernel images.
We compress zimage using a compression algorithm called LZ77. The LZ77 compression algorithm optimizes speed and perfectly balances compression ratio and decompression performance. LZ77 compression creates a smaller image file than bzimage.
We must note that the zimage format is typical of the x86 architecture and other architectures may not support it.

### bzimage

bzimage refers to a compressed kernel image file used by the Linux bootloader to load and initialize the kernel during the system boot. The bootloader reads the bzimage file from the boot device and decompresses it into memory. It then transfers control to the decompressed kernel image, which continues the boot process.
The bzimage file is the by-product of compiling the Linux kernel source code, which includes the kernel’s core functionality, device drivers, and other vital elements. When we compile the kernel, it generates vmlinux. However, vmlinux is too big to fit into the finite memory available during boot (the first 640KB of RAM). As a result, we compress the vmlinux file into a smaller size (often compressed below 512KB) using the gzip utility, creating the bzimage image file.
The compression process significantly reduces the kernel image size, making the boot process easier. The “bz” in bzimage stands for “big zipped” because we compress the kernel image using the gzip compression algorithm.
Let’s look at some of the components present in the bzimage file. First, we have the boot loader header, which contains information the boot loader requires to load and execute the kernel. This header provides details like the kernel version, the size of the compressed kernel, the offset of the compressed kernel, and the location of the initial RAM disk (initrd) if present.
The initrd is a temporary root file system loaded into memory during the boot process before the actual root file system is mounted.
Then, we have the bzimage file containing the compressed kernel image. This image includes all the necessary code and data to initialize the kernel and start the operating system. It consists of the kernel’s entry point, initialization routines, device drivers, file systems, etc.

<br/>

zimage is an older compressed format, and bzImage is an improved version.
Lastly, choosing the right kernel image format depends on a variety of factors, such as the use case, the hardware architecture, boot loader compatibility, and compression/optimization requirements.

### uimage

uImage : zImage에 다른 가공을 한 바이너리이다. u-boot source에 있는 mkimage라는 툴을 이용하여 u-boot용 커널이미지를 생성한 것이다. 얘는 zImage앞에 64byte의 헤더가 붙어서, 커널이 몇번지에 올라가야하는지, 커널이 무슨 종류인지, 등의 정보를 싣게 된다.(target architecture, operating system, image type, compression method, entry points, time stamp, CRC32 checksums 등)zImage와 달리 bootloader에서 uImage를 실행시키면 uImage가 알아서 자기위치에 압출풀고 실행된다는 것이다.

## Reference

[kernel-images](https://www.baeldung.com/linux/kernel-images)