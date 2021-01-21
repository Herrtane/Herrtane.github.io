---
layout: post
title: <C/C++> 04. 복사생성자(Copy Constructor)의 중요성 (매우 중요!)
date: 2021-01-21 17:59:23 +0900
category: C/C++
comments: true
---
## 개요

반년 전 자료구조 수업때 구현했던 프로젝트 (최근 내가 진행하고 있다는 코드 리팩토링의 대상)에서 원인 모를 블로킹 오류가 날 항상 괴롭혔다. 그래서, 당시에는 임시방편으로 일부 자료구조의 소멸자에 존재하는 delete 연산을 주석처리하거나, 아니면 linked list 자료구조형을 사용하지 않는 방법을 사용했다. 하지만, 지금 코드 리팩토링을 하면서 예상치 못하게 비슷한 오류가 또 발생했고, 원인을 도저히 찾지 못하던 나는 좌절한 나머지 리팩토링을 일시중단하기에 이르렀다. 거의 1년째 해결하지 못하는 문제를 보면서 진저리가 났고, 회피하고 싶은 마음 뿐이었다.

<br/>

그러다가, 오늘 C++에 대한 정교한 복습을 진행하면서 나를 괴롭히던 오류의 원인으로 추정되는 부분을 알게되었다. (물론, 100퍼센트 확실한 원인은 아니다. 아직 오늘 공부한 이론을 바탕으로 리팩토링을 재개해보아야 한다.) 우선 내가 짠 코드의 문제가 되는 부분을 작성해보겠다.

```cpp
...
// SortedList에 존재하는 메서드이다.
template <class T>
T SortedList<T>::GetNextItem(){
	current_pointer_++;
	return array_[current_pointer_];
}
...
// Main Application에 존재하는 메서드이다.
storage_list_.ResetCurrentPointer();
	for (int i = 0; i < storage_list_.GetLength(); i++) {
		tempstorage = storage_list_.GetNextItem();
		QString qstring;
		qstring = QString::number(tempstorage.GetStorageId());
		ui.listWidget_storage->addItem(qstring);						
	}
...
```
참고로, 모든 클래스에 복사생성자는 따로 정의되어있지 않아서, 동적 할당이 되어있는 맴버변수들이 shallow copy만 이루어진다. 게다가, 복사생성자가 호출되는 3가지 상황에서 (잠시후 다룰 것이다) 위의 코드에 해당되는 상황이 있는데, 이 상황을 계속 놓치고 있었다. 아마 리팩토링을 복사생성자 위주로 하다보면 어느정도 에러가 해결되지 않을까 기대해본다. 결과는 다음 포스팅때 작성하겠다.

## 복사생성자에 대한 중요한 내용들

1. 복사생성자를 생성하는 방법은 간단하다.
```cpp
template <class T>
ExampleClass(const ExampleClass& copy) : member1(copy.member1), member2(copy.member2) {
    array_member_ = new T[MAX_SIZE];
    for(int i=0;i<copy.length_;i++){
        array_member[i] = copy.array_member[i];
    }
}
```

2. 복사생성자가 호출되는 상황은 3가지이다.
    1. 기존에 생성된 객체를 이용해서 새로운 객체를 초기화하는 경우
    2. Call by value 방식의 함수호출 과정에서 객체를 인자로 전달하는 경우
    3. 객체를 반환하되, 참조형으로 반환하지 않는 경우
    > 이들은 모두 객체를 새로 생성해야 하고, 생성과 동시에 동일한 자료형의 객체로 초기화해야 한다는 공통점이 있다.

이 중 두 번째 경우가 자료구조 시간에 주로 다루었던 상황이고, 세 번째 경우는 내가 가벼이 여기고 넘겼었다. 그런데, 내가 짠 맨 위의 코드를 보면, **GetNextItem()메서드에서 return 값을 참조형으로 반환시키지 않는다. 즉, 내가 미처 구현하지 않은 복사생성자가 이 메서드에서 호출된다는 뜻이고, shallow copy가 이루어져서 어디선가 memory leak이 발생한다는 것이다!**  

(추후 계속 추가)