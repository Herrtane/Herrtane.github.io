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

## 마치며

이미 객체지향에 있어서 가장 중요한 개념임은 자명하다.