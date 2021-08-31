---
layout: post
title: <TCP/IP Socket> 05. bind()에 대하여
date: 2021-08-31 20:49:23 +0900
category: TCP/IP_Socket
comments: true
---

## bind()의 인자들에 대한 설명

```c
#include <sys/socket.h>

int bind(int sockfd, struct sockaddr *myaddr, socklen_t addrlen);
```

1. 첫번째 인자 : 주소정보를 할당할 소켓의 파일 디스크립터
2. 두번째 인자 : 할당하고자 하는 주소정보를 지니는 구조체 변수의 주소 값
3. 세번째 인자 : 두 번째 인자로 전달된 구조체 변수의 길이정보

## 구조체들에 대한 설명

```c
struct sockaddr_in{
    sa_family_t sin_family;     // 주소체계(Address Family), 주로 AF_INET을 사용
    uint16_t sin_port;          // PORT 번호. Network Byte Order로 저장.
    struct in_addr sin_addr;    // IP주소. Network Byte Order로 저장.
    char sin_zero[8];           // 사용되지 않음
}

struct sockaddr{
    sa_family_t sin_family;
    char sa_data[14];
}

// sockaddr_in은 IPv4의 주소정보를 담기 위해 정의된 구조체이므로, sockaddr보다 더 편히 구성할 수 있다.
```

## 네트워크 바이트 순서

- Big Endian : 상위 바이트의 값을 작은 번지수에 저장하는 방식. 
- Little Endian : 상위 바이트의 값을 큰 번지수에 저장하는 방식.
- Host Byte Order : CPU에 따라서 차이가 남.
- Network Byte Order : 네트워크를 통해서 데이터를 전송할 때 적용되는 통일된 순서. Big Endian으로 통일하기로 함.

```c
unsigned short htons(unsigned short);
unsigned short ntohs(unsigned short);
unsigned long htonl(unsigned long);
unsigned long ntohl(unsigned long);
```

위의 변환 함수들에서, 일반적으로 뒤에 s가 붙는 함수는 PORT번호의 변환에, l이 붙는 함수는 IP주소의 변환에 사용됨.

## 구체적인 사용 예시

```c
int serv_sock;
struct sockaddr_in serv_addr;
char *serv_port = "9190";

serv_sock = socket(PF_INET, SOCK_STREAM, 0);

memset(&serv_addr, 0, sizeof(serv_addr));
serv_addr.sin_family = AF_INET;
serv_addr.sin_addr.s_addr = htonl(INADDR_ANY);
serv_addr.sin_port = htons(atoi(serv_port));

bind(serv_sock, (struct sockaddr*)&serv_addr, sizeof(serv_addr));
```

위의 코드에서 bind()에서 (struct sockaddr*)로 형변환을 해주어야한다.

## 마치며

이제 서버 소켓에 주소까지 할당하는 과정까지 다루었다.