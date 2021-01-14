---
layout: post
title: 소켓과 파일 입출력의 유사성
date: 2021-01-14 13:25:23 +0900
category: C/C++
comments: true
---
## 소켓과 파일 입출력

소켓 프로그래밍을 하면 항상 등장하는 socket()과 closesocket(). 그리고, C/C++ 프로그래밍에서 파일 입출력을 진행하다보면 항상 등장하는 fopen()과 fclose(), 혹은 open()과 close(). 문득 보다보니 표준 입출력을 제외한 입출력에서는 항상 어떠한 stream을 형성하거나 link를 형성하게 되는데, 그럴경우 이를 열어서 생성하고 닫아주는 일련의 과정을 공통적으로 거친다는 것을 느꼈다.

<br/>

이런 공통된 원리를 발견할 때 마다 신기하면서도 뿌듯하달까. 아래는 혼동하기 쉬운 C와 C++의 파일 입출력 방법들이라 기록해놓는다.

```c
#include <stdio.h>
...
FILE* filePointer = fopen("...",..);
fgets(message,filePointer);
fclose(filePointer);
...
```

<br/>

```cpp
#include <iostream>
#include <fstream>
#include <string>
using namespace std;
...
ifstream fin("...");    // fin.open("...");과 초기화를 한꺼번에 함.
if(!fin.is_open()){
    cout << "File not found!" << endl;
    return 0;
}
string message;
while(fin)
    getline(fin,message);
fin.close();
...
```