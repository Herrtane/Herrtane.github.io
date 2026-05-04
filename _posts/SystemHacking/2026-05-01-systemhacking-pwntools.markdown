---
layout: post
title: <System Hacking> 23. Pwntools의 payload send시 주의사항
date: 2026-05-01 10:30:23 +0900
category: System_Hacking
comments: true
---

## sendafter vs sendline

포너블 기본 예제들을 천천히 복습하면서 문제들을 다시 풀어오고 있는 와중에, 분명 익스플로잇 설계까지는 완벽하다고 생각했는데 예상치 못한 오류 (stack smashing error 라든가..)가 종종 발생했다. 그래서 LLM과 같이 원인을 파악하다가 결국 알아낸 정말 기본적이고도 중요한 사실을 기록하려고 한다.

### 바이너리 상황 분석

```c
printf("[1] Leak Canary\n");
printf("Buf: ");
read(0, buf, 0x100);
```

* `printf("Buf: ");` → 입력 프롬프트 출력
* `read(0, buf, 0x100);` → **개행 기준이 아니라 그냥 raw byte 읽기**

즉, 이 프로그램은 **line-based (scanf/gets)**가 아니라 **byte-based (read)** 구조이므로, 함부로 개행을 포함한 입력을 주어선 안된다!! 내가 주로 쓰던 코드 패턴은 다음과 같은데, 이것이 문제였다.

```python
p.recvuntil("Buf: ")
p.sendline(payload)
```

* `sendline`은 자동으로 `\n` 추가
* 그런데 `read()`는 **그냥 0x100 bytes를 그대로 읽음**
* 즉 payload 끝에 `\n`이 들어가서:

  * 스택 오염 패턴 깨짐
  * canary overwrite 위치 틀어짐
  * ROP offset 밀림

```
b"A"*40 + canary + rbp + ret + b"\n"
```

이 1바이트가 **return address 뒤 메모리까지 침범**하게 된다.

### 정답은 sendafter

```python
p.sendafter("Buf: ", payload)
```

* `"Buf: "`까지 정확히 sync
* payload를 **그대로 raw write**
* `read()`와 완벽하게 맞음

> **이 바이너리는 `read()` 기반이므로 무조건 `sendafter` 또는 `send()` 계열 (raw) 사용해야 안전하다!!**

## p32/p64 vs str().encode()

- p32/p64 = 메모리에 직접 넣을 값 (주소, 숫자 등)
- str().encode() = 프로그램에 타이핑해서 입력해야 되는 값
