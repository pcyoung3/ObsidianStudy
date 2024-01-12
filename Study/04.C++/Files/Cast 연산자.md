---
Index: "[[CppIndex]]"
tags:
  - Study
  - cpp
---
   
## 1. reinterpret_cast
---
* 포인터>포인터, 포인터 > 일반변수, 일반변수 >포인터 자료형을 바꿀 수 있는 Cast
* 형변환이 이뤄지면 해당 자료형의 bit수에 맞게 들어간다
* void* 로 전달할 수 있는 유일한 Casting
* struct에 있는 맴버변수를 캐스팅해서 사용이 가능 [[이득우Framework_10_게임데이터 관리#^a96e76|참고]]
```cpp
int* num = new int(100);
char c = reinterpret_cast<char>(num);   //주소값을 char 형으로 바꿈-> 오류발생 : char는 1byte라 주소값인 4byte를 담을 수 없음

char ch = 3;
int* chTemp = reinterpret_cast<int*>(ch);   // 주소값에 3을 넣는 행위

void* v = (num);
long* l = reinterpret_cast<long*>(v);   //void 형을 원하는 자료형으로 변경 가능
```
   
   
## 2. const_cast
---
* const를 잠시 깰 수 있는 cast
* 포인터 형식으로만 받을 수 있다.
* 본연의 const 값을 바꿀 수는 없다
```cpp
const int i = 3; // i 는 const 로 선언
int* pi = const_cast<int*>(&i);
*pi = 4;      // undefined behavior
cout << *pi << endl;    //4
cout << i << endl;      //3
```
   
* 레퍼런스를 잠시 제거하는데 사용할 수 있다.
```cpp
// 레퍼런스 예시 
int i = 3; // i is not declared const
const int& rci = i; 
const_cast<int&>(rci) = 4; // OK: modifies i = 4
```
   
   
## 3. static_cast
---
* 컴파일 시점에서 일반적인 형변환
* 상속된 객체를 알아보고 cast 한다.
* 업캐스팅을 했을 때 실객체는 신경쓰지 않고 cast를 진행하기 때문에 런타임에서 문제가 생길 수 있다
   
   
## 4. dynamic_cast
---
* 런타임 시점에서 형변환
* virtual 함수가 있어야 형변환을 허용한다
* 런타임 시점에서 vfptr을 참고하여 실객체까지 확인하고 성공하면 포인터를 반환하지만 실패하면 nullptr을 반환한다
* 다만 vfptr을 확인하고 반환해서 부하가 걸림

