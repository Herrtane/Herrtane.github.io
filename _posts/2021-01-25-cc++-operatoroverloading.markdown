---
layout: post
title: <C/C++> 08. operator overloading
date: 2021-01-25 15:33:23 +0900
category: C/C++
comments: true
---
## operator overloading을 하는 두 가지 방법

1. 맴버함수에 의한 operator overloading
    ```cpp
    ...
    class Example {
        public:
            Example operator+(const Example& ref){
                Example pos(xpos+ref.xpos);
                return pos;
            }
    };
    ...
    ```
2. 전역함수에 의한 operator overloading
    ```cpp
    ...
    class Example {
        public:
            friend Example operator+(const Example& pos1, const Example& pos2);
    };

    Example operator+(const Example& pos1, const Example& pos2){
        Example pos(pos1.xpos+pos2.xpos);
        return pos;
    }
    ...
    ```

객체지향에는 전역(global)이라는 개념이 존재하지 않으므로, 가급적 맴버함수로 구현하는 것을 권장한다. 다만, **자료형이 다른 두 피연산자를 대상으로 하는 연산**에서 **교환법칙을 적용해야 할 경우**, 전역함수에 의한 operator overloading이 불가피하다.

```cpp
Example operator*(int times, Example& ref){
    Example pos(ref.xpos * times);
    return pos;
}
```

혹은, 멤버함수에 의한 operator overloading을 구현한 상태에서, 다음과 같이 간단하게 해도 된다.

```cpp
Example operator*(int times, Example& ref){
    return ref * times;
}
```

## 반환형이 참조형일때

```cpp
class Example {
    public:
        Example& operator++(){
            xpos+=1;
            return *this;
        }
    friend Example& operator--();
};

Example& operator--(Example& ref){
    ref.xpos+=1;
    return ref;
}
```

1. this는 객체자신의 포인터 값을 의미하므로, *this는 객체자신을 의미한다. 즉, 위에서는 **객체 자신을 반환**한다.
2. 반환형이 참조형이므로, 위 함수의 호출 결과로 **객체 자신을 참조할 수 있는 참조값이 반환**된다.
3. 만약 반환형이 Example이면, 객체 자신의 복사본을 만들어서 반환을 할 것이다.

## 대입 연산자의 overloading

결론부터 이야기하자면, **복사 생성자와 거의 유사하다.** 한 가지 더 짚고 넘어갈 점은, 자료구조 시간에도 배웠지만, shallow copy시에 dangling pointer와 더불어, **memory leak**가 발생할 우려가 있으므로, 대입 연산자의 첫 부분에는 반드시 동적 할당된 멤버 변수를 **delete**해야한다. 더 나아가서, **initializer를 이용한 객체 생성**이 **생성자 몸체에서의 대입 연산**보다 선호되는 이유는, 전자는 복사 생성자만 호출되지만, 후자는 복사 생성자와 더불어 대입 연산자까지 호출되기 때문에 성능상의 차이가 생기기 때문이다.

## 마치며

이로써, 윤성우 선생님의 열혈 C++ 프로그래밍 책에 대한 중요한 복습 내용을 모두 포스팅하였다. 이후로는, 내가 다른 공부를 진행하면서 C++에서 새로 알게된 중요한 사실이 있다면 그때그때 포스팅하도록 하겠다.