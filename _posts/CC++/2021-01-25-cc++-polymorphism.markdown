---
layout: post
title: <C/C++> 07. virtual과 polymorphism
date: 2021-01-25 14:03:23 +0900
category: C/C++
comments: true
---
## virtual function과 polymorphism

책에서 virtual과 polymorphism에 대해 압축해서 설명하는 문장들을 이곳에 적는다.

1. C++에서 AAA형 포인터 변수는 AAA객체 또는 AAA를 직간접적으로 상속하는 모든 객체를 가리킬 수 있다.
2. C++ 컴파일러는 포인터를 이용한 연산의 가능성 여부를 판단할 때, **포인터의 자료형을 기준으로 판단**하지, 실제 가리키는 객체의 자료형을 기준으로 판단하지 않는다.
3. 2번을 해결하기 위해 virtual function을 사용한다. **virtual function이란, base class내에 선언되어 derived class에 의해 재정의되는 맴버 함수**를 말한다.
4. virtual function이 선언되고 나면, 이 함수를 오버라이딩하는 함수도 virtual function이 된다.
5. **클래스 중에서는 객체생성을 목적으로 정의되지 않는 클래스도 존재**하는데, 이러한 경우 프로그래머의 실수를 막기 위해 pure virtual function으로 선언하여 객체의 생성을 문법적으로 막는 것이 좋다. **pure virtual function은 함수의 몸체가 정의되지 않은 함수**를 말하며, 명시적으로 몸체를 정의하지 않았음을 컴파일러에 알린다. 이 함수를 선언한 클래스를 가리켜 **abstract class**라 한다.
    ```cpp
    virtual void PureExample() = 0;
    ```
6. 상속을 하는 이유는, 상속을 통해 연관된 일련의 클래스들에 대해 공통적인 규약을 정의할 수 있기 때문이다. 따라서, 모든 상속되는 객체들을 공통적인 객체로 바라볼 수 있게 된다.
7. 지금까지 이야기한 virtual function의 호출 관계에서 보인 특성을 가리켜 **polymorphism**이라 한다. 이는 문장은 같은데 결과는 다른 특성을 말한다.
8. virtual destructor 역시 필요하며, 이를 선언하지 않으면 memory leak가 발생할 우려가 있다.

## 마치며

이미 객체지향에 있어서 가장 중요한 개념임은 자명하다.