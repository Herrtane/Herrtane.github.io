---
layout: post
title: <Firmware> 3. Kernel image Extraction with binwalk, dd, hexdump
date: 2024-06-04 14:30:23 +0900
category: Firmware
comments: true
---

## Introduction

현재 연구실 드론 및 펌웨어와 관련된 과제 (자세한 사항은 보안사항이라 언급할 수는 없다)를 진행하면서 계속해서 펌웨어와 이미지파일에 대한 공부가 필요하여, 이번에도 공부하면서 익힌 정보들을 정리하고자 포스팅한다. 이번에는 **DJI Spark 드론의 chip-off된 internal image file에서 Kernel image file 및 File system data를 직접 추출해서 확인**해보는 과정을 담아보았다.

### The process of extracting a kernel image in drone internal image file

우선 사전에 확보된 DJI Spark 드론 내부 이미지 파일의 구조를 파악하기 위해, 다음과 같은 시도를 했다.

1. FTK imager 상에서 이미지 파일 분석
2. binwalk를 사용하여 이미지 파일 분석

첫번째 방법은 잘 되지 않았는데, FTK imager에서 해당 이미지 파일을 제대로 분석하지 못했기 때문이다. 그렇게 고민하다가 내가 주로 연구를 진행하고 있는 binwalk 도구를 사용해보자는 아이디어가 떠올랐고, binwalk를 사용해서 이미지 파일을 분석해본 결과, 성공적으로 아래 사진과 같이 결과가 나왔다.

![spark_internal_binwalk_1]({{site.url}}/img/spark_internal_binwalk_1.png)

보면 드론 이미지 내부에 ext기반의 file system부터, zimage 형식의 arm linux image 파일도 인식되는 것을 확인할 수 있었다. 바로 -e 옵션을 붙여서 추출을 진행했고, 추출 결과는 아래와 같다.

![spark_internal_ls_no_kernel_file]({{site.url}}/img/spark_internal_ls_no_kernel_file.png)

분명 binwalk 출력창에서는 zimage 커널 이미지가 인식되었으나, 정작 추출 폴더 내에서는 해당 커널 이미지를 찾을 수 없었다. 이를 해결하기 위해, 직접 드론 이미지 중에서 binwalk의 출력창에 표시된 offset 위치의 부분만을 추출해보기로 했다. 우선, binwalk가 정확하게 zimage magic number (signature)를 인식한 것인지, 오탐은 아닌지 확인하기 위해 직접 해당 offset의 hexdump 값을 확인해서 실제 zimage magic number와 비교해보았다. (실제 zimage magic number는 직전 포스팅에도 작성해두었다.)

![spark_internal_image_zimage_magic]({{site.url}}/img/spark_internal_image_zimage_magic.png)

hexdump 결과, binwalk가 오탐을 한 것은 아니라는 것을 확인하였다. 그렇다면, 이제 직접 추출하는 단계만 거치면 zimage 파일을 따로 추출할 수 있을 것이다. 추출 도구는 리눅스 기본 도구인 **dd** 명령어를 사용했다. bs는 binwalk 상에서 zimage라고 인식한 offset의 시작 위치를, skip은 1회 진행하였다.

![spark_internal_dd]({{site.url}}/img/spark_internal_dd.png)

그 다음, 실제로 추출된 이 파일이 실제 zimage 파일이 맞는지 확인하기만 하면 된다. 확인 과정은 리눅스 기본 도구인 **file** 명령어를 사용해서, 해당 명령어가 추출한 파일을 zimage로 인식하면 성공이다.

![spark_internal_kernel_extract_success]({{site.url}}/img/spark_internal_kernel_extract_success.png)

성공적으로 추출하였다! 

### The process of identifying and verifying a file system

다음으로, binwalk를 통해 추출된 ext 파일 시스템 데이터가 정상적으로 추출된 것인지 검증하는 작업을 하기로 했다. **binwalk로 추출된 ext 파일시스템 결과와 실제로 ext 파일 시스템 데이터를 마운트하였을 때 결과물이 동일한지** 검증해보자. 과정은 간단하다. 우선 binwalk를 통해 추출된 파일시스템 결과물은 이미 폴더 상에 존재하므로, 내가 해야할 것은 binwalk의 **중간 생성파일**인 파일시스템 데이터파일 그 자체를 mount하기만 하면 된다. 리눅스의 mount 기능을 사용해서 아래와 같이 진행하였다.

![ext_filesystem_mount]({{site.url}}/img/ext_filesystem_mount.png)

실제로 결과물을 비교한 결과, binwalk가 정확하게 자동 추출했음을 확인하였다.

## Conclusion

이번 포스팅을 통해 펌웨어와 이미지 분석에서 자주 사용되는 다양한 도구 및 분석 방법을 복습하고, 이해하는 시간이 되었다.