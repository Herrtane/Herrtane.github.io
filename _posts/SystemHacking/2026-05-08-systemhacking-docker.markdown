---
layout: post
title: <System Hacking> 24. libc, ld 버전 호환을 위한 Docker 사용법
date: 2026-05-08 10:30:23 +0900
category: System_Hacking
comments: true
---

## libc, ld 버전에 대하여

ELF 바이너리는 실행 시 loader에 의해 로드되며, 이 loader가 특정 버전의 libc를 기준으로 내부 구조와 심볼을 해석하기 때문에, libc만 단독으로 교체할 경우 내부 함수 동작이나 메모리 레이아웃, hook 위치 등이 어긋나게 된다. 

<br/>

특히 CTF 환경에서는 제공된 libc를 기준으로 주소 계산을 수행하기 때문에, 로컬에서 system libc와 섞여 실행될 경우 오프셋 불일치나 EOFError 같은 문제가 발생할 수 있다. 따라서 안정적인 익스플로잇을 위해서는 libc뿐만 아니라 해당 libc와 짝을 이루는 loader까지 함께 맞춰 실행 환경을 동일하게 구성하는 것이 필수적이다.

<br/>

다행히 Dreamhack wargame에서는 libc 파일을 제공하고, Dockerfile도 같이 제공하기 때문에, 필요한 ld 파일을 Docker Container 내에서 추출해서 사용할 수 있다.

## 문제 해결 방법

### Docker 설치 및 설정

```
$ sudo apt update
$ sudo apt install -y docker.io
$ sudo service docker start
$ sudo usermod -aG docker $USER
$ newgrp docker
```
 
### 문제 제공 Dockerfile 통한 Container 설치 및 Bash 실행

```
$ docker build -t pwn_env .
$ docker run -dit --name pwn_container pwn_env
$ docker exec -it pwn_container /bin/bash
```

### Container 내 문제 파일에 연결된 libc, loader 확인 및 복사

이 부분은 Docker 내부에서 입력해야되는 명령어들이다. fho는 Hook Overwrite 문제에서 썼던 문제 파일 이름이므로, 다른 문제 파일에서는 해당 파일명을 적으면 된다.

```
$ ldd ./fho
        linux-vdso.so.1 (0x00007ffc1a9d6000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x000075472e06b000)
        /lib64/ld-linux-x86-64.so.2 (0x000075472e65e000)
```

여기는 다시 Docker 외부 Host에서 입력하면 된다.

```
$ docker cp pwn_container:/lib/x86_64-linux-gnu/libc.so.6 .
$ docker cp pwn_container:/lib/x86_64-linux-gnu/ld-2.27.so .
$ mv ld-2.27.so ld-linux-x86-64.so.2
```

이제 성공적으로 복사되었다면 아래와 같이 ld-linux-... 파일이 Symbolic Link가 아닌 ld 파일 그 자체여야 한다. 또한 정상 실행이 되어야한다.

```
$ ls -l ld-linux-x86-64.so.2
-rwxr-xr-x ...
$ LD_LIBRARY_PATH=. ./ld-linux-x86-64.so.2 ./fho
(정상 실행)
```

### pwntools 스크립트 작성법

```python
from pwn import *

p = process(
    ['./ld-linux-x86-64.so.2', './fho'],
    env={"LD_LIBRARY_PATH": "."}
)

libc = ELF('./libc-2.27.so')
e = ELF('./fho')
```

### 새로운 문제 해결 시 위의 과정 반복

새로운 문제에서는 또 Dockerfile이 다르기 때문에, 번거롭더라도 기존 Dockerfile은 지우고 다시 새로 구축해야한다.

```
$ docker rm -f pwn_container
(이후 동일한 과정 반복)
```
