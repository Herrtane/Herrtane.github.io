---
layout: post
title: <C/C++> 05. const와 static의 유용성
date: 2021-01-22 13:14:23 +0900
category: C/C++
comments: true
---
## const

방어적 프로그래밍을 위해서 꼭 필요한 const 키워드. 자주 사용되는 경우를 이 곳에 적어보면,

```cpp
const int num = 10;
const Example ex(20);
```
객체에 위와 같이 const 선언을 붙이면, 객체의 데이터 변경을 허용하지 않겠다는 의미이며, const 멤버함수만 이 객체를 대상으로 호출할 수 있다.

```cpp
void Func(){...}
void Func() const{...}
```
또한, const의 선언유무로도 함수의 오버로딩이 가능하다. 멤버변수에 저장된 값을 수정하지 않는 함수는 가급적 const로 선언하는 습관을 들이자.

## static

### C에서의 static

1. 전역변수에 선언된 static의 의미 : 선언된 파일 내에서만 참조를 허용하겠다.
2. 함수 내에 선언된 static의 의미 : 한번만 초기화되고, 지역변수와 달리 함수를 빠져나가도 소멸되지 않는다.

### C++에서의 static

가령 Example 객체들이 공유하는 한 전역변수 a가 있다고 하자. 이 변수를 Example 객체들끼리만 공유하고 싶은데, 전역변수의 특성상 이러한 제한을 지켜 줄만한 아무런 장치도 존재하지 않는다. 어디서든 접근이 가능하기 때문이다. 이럴때 사용하는 것이 C++에서의 static이다.

3. static 멤버변수로 선언할 경우, 이를 **클래스 변수** 라고도 한다. 클래스당 하나씩만 생성된다. 예를 들면,

```cpp
class Example{
    private:
        static int a;
    public:
        ...
};
```

위의 클래스 변수 a는 객체가 생성될 때마다 함께 생성되어 객체별로 유지되는 변수가 아니라, 메모리 공간에 딱 하나만 할당이 되어서 공유되는 변수이다. **객체의 외부에 존재하며, 다만 객체에게 멤버변수처럼 접근할 수 있는 권한을 주었다.** 여기서 주의할 점은, 클래스 변수는 **반드시 생성자가 아닌, 클래스 외부에서 초기화해야**한다. 만약 생성자 내에 클래스 변수를 초기화할 경우, **객체가 생성될 때 마다 이 변수는 초기화된다.** 이는 이치에 맞지 않다.

<br/>

더불어, 상당히 유용하게 사용될 수 있는 부분이 있다. 바로, **static 멤버 변수가 public으로 선언되면, 클래스의 이름이나 객체의 이름을 통해서 어디서든 접근이 가능하다는 것**이다. 아래의 코드가 예시인데, 이를 통해서 static 멤버 변수가 객체 내에 존재하지 않는다는 사실도 증명할 수 있다.

```cpp
#include <iostream>
using namespace std;

class Example{
    public:
        static int count;
    public: 
        Example(){
            count++;
        }
};
int Example::count = 0;

int main(){
    cout << Example::count << "번째 Example 객체" << endl;
    Example ex1;
    Example ex2;

    cout << Example::count << "번째 Example 객체" << endl;
    cout << ex1.count << "번째 Example 객체" << endl;
    cout << ex2.count << "번째 Example 객체" << endl;
    return 0;
}
```

참고로, 위의 마지막 두 문장처럼 접근할 경우, 마치 객체의 멤벼변수로 접근하는 착각을 줄 수 있으므로, 가급적이면 첫 번째 문장처럼 클래스의 이름을 이용해서 접근하도록 하자.

4. static 멤버함수 역시 static 멤버변수와 특성이 동일하다. 또한, **static 멤버함수 내에서는 static 멤버변수와 static 멤버함수만 호출이 가능**하다.

## 마치며

오히려 클래스, 상속, 오버로딩 등 자주 사용하는 개념들을 다루다 보면, 지금까지 다루었던 키워드들을 놓치기 쉬운 것 같다. 이 기회에 C++을 정교하게 다질 겸 계속해서 내가 부족했던 부분을 포스팅해나가도록 하겠다.