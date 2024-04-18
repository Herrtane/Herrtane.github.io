---
layout: post
title: <Firmware> 1. CFE (Common Firmware Environment)
date: 2024-04-18 14:30:23 +0900
category: Firmware
comments: true
---

## CFE

### Definition and Characteristic

- Common Firmware Environment is the firmware and bootloader developed by Broadcom for SoC systems
- It is intended to be a flexible toolkit of CPU initialization and bootstrap code for use on embedded processors
- Its main responsibility is to initialize CPUs, caches, memory controllers, and peripherals required early on in the power on stage.
- It is the first code that runs when the router boots and performs functions similar to Apple's Open Firmware

1. Initializes the system
2. Sets up a basic environment in which code can run
3. Optionally provides a CLI non-standard usage
4. Loads and executes a kernel image (expecting to be jettisoned shortly thereafter)

- Just like in other boot loaders environment, variables are commonly configured in persistent storage to create auto boot options. 
- It also has support for network bootstrap
- So, in normal operation, a user will not see CFE working at all; it will load the Linksys kernel and send it on its merry way without hesitation. 
- For us, however, CFE is crucial, because it provides us with the ability to load an image over the network using TFTP
- ASUS Routers, Netgear, Linksys WRT54G series, LG, Samsung smart TV

### Secure Boot

1. The SoC has as factory settings, most probably in the OTP fuses, the private key unique per each model and also 2 keys AES CBC (ek & iv). This is the Root of Trust which is known by OEM.
2. During boot, the PBL (Primary Boot Loader coded in the SoC) will search for storage peripherals e.g. NAND or NOR SPI. If found then loads a small portion from start of storage into memory. Exact amount may depend on model and storage but most typically 64kb. In the sources this chunk is called CFEROM.
3. Once loaded the CFEROM, the PBL will analyse the structure, which is a compound of different chunks: valid header, magic numbers, signed credentials, CRC32, actual compiled code, etc. In the end, the PBL will decide if CFEROM meets the structure required and it is properly signed. If this is so, then the PBL will execute the compiled code encapsulated. Note that this code is usually not encrypted and therefore can be detected with naked eyes.
4. Typically, CFEROM will start PLL's and full memory span. Most probably doesn't need to run a storage driver since it is already working. Then it will jump to CFERAM location as coded
5. CFERAM binary is encoded in JFFS2 filesystem. It must meet a certain structure as CFEROM. The compiled code is usually LZMA compressed and AES CBC encrypted, rendering the resulting binary absolutely meaningless.

## 마치며

펌웨어 분석 중 CFE 개념이 등장해서 이 기회에 확실하게 정리했다!

## Reference

[cfe_openwrt](https://openwrt.org/docs/techref/bootloader/cfe)