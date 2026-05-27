---
layout: post
title: <Embedded> 10. Yocto, SDK Toolchain, Kernel Module Build
date: 2026-05-27 10:30:23 +0900
category: Embedded/Kernel
comments: true
---

## Introduction

회사에서 본격적으로 커널과 관련된 업무를 진행하다보니, 자주 접해야되는 개념들이 있어서, 커널 빌드와 관련된 배경지식을 이곳에 정리하려고 한다. 회사 내 업무 내용은 비공개 사항이므로 해당 내용은 제한다.

## Yocto

Yocto란 하드웨어 아키텍처와 무관하게 커스텀 임베디드 리눅스를 만들기 위한 빌드 프레임워크이다. 가상의 환경 예시를 통해서 개념과 실제 빌드 예시를 다루어보겠다.

- 빌드 디렉토리: `~/yocto/build`
- 소스(워킹) 디렉토리: `~/yocto/poky`
- 타겟 머신: `myboard64`
- 이미지 레시피: `core-image-demo`
- 커널 레시피: `linux-myvendor`
- 앱 레시피: `myapp`
- 커널 모듈 레시피: `mymodule`

```
# 1) Yocto 환경 진입  (필수 1회/터미널마다)
cd ~/yocto/poky
source oe-init-build-env ~/yocto/build

# 2) 레이어 확인/추가(선택)
bitbake-layers show-layers
bitbake-layers add-layer ../meta-mycompany

# 3) 레시피 단품 빌드
bitbake myapp
bitbake mymodule

# 4) 커널만 빌드(선택)/ 커널 메뉴 설정
bitbake linux-myvendor
bitbake linux-myvendor -c menuconfig

# 5) 이미지 빌드(최종 산출물)
bitbake core-image-demo

# 6) 필요하면 SDK 생성(선택)
bitbake core-image-demo -c populate_sdk
```

## SDK Toolchain

업무를 하다가 문득 궁금해졌다. '왜 벤더사마다 Toolchain이 다 다를까? 번거롭게 하지 말고 그냥 aarch-linux-gnu... 등으로 통일하면 안되려나?'

그래서 이에 대한 의문을 정리해보았다.

### 벤더사가 굳이 고유의 SDK Toolchain을 배포하는 이유?

1. 툴체인이란? Compiler, Binutils, C 라이브러리, 각종 헤더파일들, sysroot 들의 패키지
2. sysroot란? 개발 PC에 있는 `/usr/lib` 등은 x86_64이므로, aarch64용 라이브러리/헤더 세트 등을 개발 PC에 복제해놓고 컴파일러/링커가 그 복제본을 /(root)처럼 보게 하는 것
3. 이 전체 조합이 특정 제품의 OS, RootFS와 1:1로 맞게 구성됨
4. 동일 aarch64여도 OS/배포판/라이브러리 버전이 다르면 링크 결과가 달라짐 -> sysroot로 이것까지 맞춰야함
5. 그래서 벤더는 **“우리 타겟에 맞는 sysroot + 그 sysroot에 맞춘 gcc/binutils 세트”**를 툴체인으로 제공
ex) `source ~/bdk-.../environment-setup-aarch64-linux...` : 이 스크립트를 통해 CC, --sysroot=..., CFLAGS, PATH 우선순위 등을 한꺼번에 세팅해서 Target 환경을 정확하게 재현한다

## Kernel Module Build

커널 모듈 빌드 자체는 사내 리눅스 프로그래밍 교육 때 배웠지만, 몇 가지 기억하면 좋을 사항들을 따로 정리해둔다.

### 외부 모듈 빌드 시의 정석 플로우

```
# 1) 커널 설정(.config)을 최신 Kconfig 기준으로 보정
make -C $KSRC O=$KBLD olddefconfig

# 2) 외부 모듈 빌드에 필요한 생성물 준비
make -C $KSRC O=$KBLD modules_prepare

# 3) 그 다음부터는 모듈만 반복 빌드
make -C $KSRC O=$KBLD M=$PWD modules
```

### config 옵션

```
oldconfig : 새로 생긴 옵션을 하나하나 물어봄(대화형)
olddefconfig : 새 옵션은 기본값(default)으로 자동 채움(비대화형, CI/자동화에 유리)
defconfig : 아예 특정 보드/아키에 맞는 기본 설정으로 새로 생성
menuconfig : 메뉴 UI로 수정

Yocto/빌드 서버/자동 스크립트에서는 대부분 olddefconfig를 선호 (사람 입력이 필요 없으니까)
```

### module_prepare

외부 모듈은 커널 소스만 있으면 되는 게 아니라, 커널의 설정과 빌드 과정에서 생성되는 파일들이 필요 (autoconf.h, utsrelease.h, compile.h 등) -> 외부 모듈(.ko)을 빌드하는 데 필요한 커널 빌드 환경을 준비
<-> 참고로, 그냥 prepare는 커널/내장 빌드까지 포함해 더 넓은 준비를 하는 느낌

### 오류 메시지별 상황 매칭

1. `include/generated/autoconf.h: No such file or directory`
→ `modules_prepare` 안 됨 / KBLD 깨짐

2. 새 옵션 질문이 뜨거나, `config` 불일치 경고가 많음
→ `olddefconfig` 필요

3. 모듈 로드 시 `vermagic mismatch`
→ 커널 빌드에 사용된 컴파일러/설정과 모듈이 다름
(CC/툴체인, KBLD/.config, kernel release 불일치)
