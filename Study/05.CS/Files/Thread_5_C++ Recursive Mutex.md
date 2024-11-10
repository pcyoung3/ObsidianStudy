---
Index:
  - "[[5. CSIndex]]"
tags:
  - Study
  - CS
---
   
## 1. Recursive Mutex?
---
기존의 Mutex는 Lock과 UnLock이 단 한 번씩 이루어진다
따라서 여러개의 Lock을 걸고 싶을 때는 여러개의 Mutex를 선언해야만 한다

보통의 경우라면 그냥 선언하면 되지만 재귀함수라면 어떻게 해야할까
재귀함수나 반복되는 곳에서는 기존의 Mutex를 사용할 수 없으므로
이럴 때 사용하는 것이 Recursive Mutex다
   
   
## 2. 작동방식
---
Lock 과 Unlock을 카운팅함
소유권을 얻기 위해서는 Lock한 횟수만큼 Unlock이 되어야 함

> [!warning] 무한으로 재귀호출이 되면?
> 재귀가 무한루프에 빠지면 함수 stack overflow가 발생하는 것과 같은 방식으로
> Recursive Mutex도 무한으로 카운팅되지 않는다
> 최대 Lock 가능 횟수를 넘어가면 `std::system_error`를 발생시킨다
   
   
## 3. 예시코드
---
```cpp
#include <iostream>
#include <mutex>
#include <thread>
 
class X
{
    std::recursive_mutex m;
    std::string shared;
public:
    void fun1()
    {
        std::lock_guard<std::recursive_mutex> lk(m);
        shared = "fun1";
        std::cout << "in fun1, shared variable is now " << shared << '\n';
    }
    void fun2()
    {
        std::lock_guard<std::recursive_mutex> lk(m);
        shared = "fun2";
        std::cout << "in fun2, shared variable is now " << shared << '\n';
        fun1(); // recursive lock becomes useful here
        std::cout << "back in fun2, shared variable is " << shared << '\n';
    }
};
 
int main() 
{
    X x;
    std::thread t1(&X::fun1, &x);
    std::thread t2(&X::fun2, &x);
    t1.join();
    t2.join();
}
```
   
### 출력
```
in fun1, shared variable is now fun1
in fun2, shared variable is now fun2
in fun1, shared variable is now fun1
back in fun2, shared variable is fun1
```
