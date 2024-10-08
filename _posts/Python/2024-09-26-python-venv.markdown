---
layout: post
title: <Python> 1. venv와 환경변수, requirements.txt
date: 2024-09-26 12:31:23 +0900
category: Python
comments: true
---

## 고질적인 윈도우 venv 문제

일반적으로 파이썬 venv를 사용하게 되면, global로 설치된 파이썬과는 격리된 공간에 각종 모듈들이 설치된다.

```
# 윈도우 상에서의 venv 설치 및 사용법
python -m venv venv_test
.\venv_test\Scripts\activate
python
>> import sys
>> sys.executable
```

그러나, 꼭 내 연구용 컴퓨터의 Windows 상에서는 파이썬이 2개 이상 설치되어 있고, 환경변수로 두 개의 파이썬이 모두 등록되어있을 경우, 아무리 venv를 활성화하여도 global의 파이썬 인터프리터가 실행된다. 정말 이 문제로 수 없이 많은 실험들을 했으나, 결과는 마찬가지였다.

1. 파이썬 환경변수 우선순위를 바꾸어보았으나, 실패
2. 파이썬을 모두 삭제하고, 흔적 파일들까지 삭제한 뒤 다시 설치하고 재부팅해보았으나, 실패
3. venv의 이름을 다른 이름으로도 바꾸어보았으나, 실패

결국 내 연구용 컴퓨터에서는, **직접 venv 내의 인터프리터를 지정해서 명령어를 실행**해야 venv 내에서 정상적으로 동작했다. 결국 문제의 원인은 아직도 밝히지 못했으나, 유력한 예상 원인은 **Windows 환경 변수가 무엇인가 꼬인 것 같다**는 것이다. 계속 global 인터프리터를 터미널에서 먼저 인식하다보니..

<br/>

그리고, 가급적이면 특정 버전의 파이썬으로 venv를 구성하고 싶으면, 아래의 명령어를 쓰는 습관을 들이자.

```
py -3.7 -m venv venv_test
```

## requirements.txt 문제 해결

다음으로, requirements.txt를 설치할 때, 이놈이 당최 설치가 잘 되다가도 정작 Libs/site-packages 안에 들어가봐도 아무것도 설치되어있지 않는 문제가 있었다. 그래서 **Process Monitor를 사용해서 설치 과정에서 로컬의 어느 경로에 이 패키지들이 저장되는지 추적**해보았는데, **로컬의 Temp폴더 내에 저장**되는 것을 확인하였다. 

<br/>

뭔가 이상해서 다시 한번 requirements.txt를 설치해보니, 중간에 Error가 한번 뜨면서 설치가 중단되는데, 나는 이 Error가 발생해도 그 전까지 설치된 애들은 정상적으로 설치가 되는줄 알았으나, 알고보니 그것이 아니었다. **우선 Temp폴더 내에 설치를 한 후, 문제가 없으면 그 내용들을 Libs/site-packages로 이동**하는 원리로 설치하는 것이었다. 그래서 중간에 Error가 발생해서 설치가 중단되면 단 하나도 설치가 되지 않는 것이었다.

## 마치며

예전부터 venv 사용에 있어서 자꾸 문제가 생겨서 venv 사용이 꺼려지고, 결국 파이참에서 설정해주는 venv로만 사용했었는데, 이번 기회에 venv의 구조나 원리 등도 철저히 공부할 수 있었고, venv의 두려움이 사라졌다. 고통스러운 문제 해결 과정이었으나, 앞으로 venv 사용은 매우 자주 하게 될 것이므로, 오히려 잘 되었다고 할 수 있다.