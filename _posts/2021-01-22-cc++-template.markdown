---
layout: post
title: <C/C++> 06. template 심화
date: 2021-01-22 14:03:23 +0900
category: C/C++
comments: true
---
## function template의 특수화 (specialization of function template)

```cpp
template <typename T>
T Max(T a, T b){
    return a > b ? a : b;
}

template <>
char* Max<char*>(char* a, char* b){
    return strlen(a) > strlen(b) ? a : b;
}

template <>
const char* Max<const char*>(const char* a, const char* b){
    return strcmp(a,b) > 0 ? a : b;
}
```

만약 위에서 첫번째 함수 템플릿만 있고, 이 함수를 이용해서 두 문자열을 비교한다고 가정하자. 그럼 무엇을 기준으로 문자열을 비교할 것인가? 그래서 문자열에 한해서는 위의 예외 템플릿을 구성해야 한다.

## class template의 특수화 (specialization of class template)

함수 템플릿과 마찬가지이다.

```cpp
template <typename T1, typename T2>
class Example{
    ...
}

template <>
class Example <int, double> {
    ...
}

template <typename T1>
class Example <T1, double> {
    ...
}
```

참고로, 위의 세번째 같은 경우 T2 하나에 대해서만 부분적으로 특수화를 진행했기 때문에, 이를 가리켜 **클래스 템플릿의 부분 특수화 (class template partial specialization)** 이라 한다. 그리고, **부분 특수화와 전체 특수화의 두 가지 모두에 해당하는 객체 생성 문장에서는 전체 특수화된 클래스가 우선시 되어 생성**된다.

## template 인자

- 템플릿 매개변수 : T, T1, T2와 같이 결정되지 않은 자료형을 의미하는 용도로 사용되는 매개변수
- 템플릿 인자 : 템플릿 매개변수에 전달되는 자료형 정보

템플릿 매개변수에는 변수의 선언이 올 수 있고, 디폴트 값의 지정도 가능하다.

```cpp
template <typename T=int, int len=7>
class Example{
    private:
        T arr[len];
    public:
        T& operator[] (int idx){
            return arr[idx];
        }
};
...
Example<int,5> i5arr;
Example<double,7> d7arr;
Example<> arr;
...
```

이는 얼핏 보면 굳이 해야되나 싶은 생각이 든다. 이 방식이 의미가 있는 상황은 책에 적혀있으므로, 여기선 생략하겠다.

## template과 static

한줄로 요약할 수 있다. 함수 템플릿이든 클래스 템플릿이든, static 변수가 템플릿 별로 각각 존재하게 된다.

```cpp
template <typename T>
void StaticValue(void){
    static T num=0;
}

template <typename T>
class Example{
    private:
        static T num;
};

template <typename T>
T Example<T>::num=0;

template <>
long Example<long>::num=5; // 일부 변수를 특수하게 초기화할 수 있다.
```


## 마치며

이렇게 정리하고 나니, template에 대해서도 어느정도 자신감이 생겼다. 계속 다음 포스팅에서 이어가겠다.