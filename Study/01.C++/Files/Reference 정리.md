---
Index:
  - "[[1. CppIndex]]"
tags:
  - study
  - cpp
---
   
## 1. 기본정의
---
포인터와는 다르게 Reference는 변수에 별명을 하나 더 추가하는 식
따라서 초기화 할 때 대상을 필수로 지정해주어야 하며 바꿀 수 없다

몇 가지의 법칙이 있는데
1. Reference는 메모리 상에 존재하지 않을 수도 있다
2. Reference의 Reference는 만들 수 없다(&&는 불가능)
3. Reference의 배열은 있을 수 없다
4. Reference의 포인터는 있을 수 없다

   
   
## 2. Reference 배열
---
[[Reference 정리#1. 기본정의]] 의 법칙에 따라서 Reference로 이루어진 배열은 있을 수 없다
```cpp
int a, b;
int& arr[2] = { a, b };   //Error
```

하지만 배열을 Reference로 받을 수는 있다.
```cpp
#include <bits/stdc++.h>
using namespace std;

int main()
{
	int arr[3] = { 1, 2, 3 };
	int(&ref)[3] = arr;		//괄호를 추가해서 배열의 Reference를 만듦
	std::cout << ref[0] << ref[1] << ref[2] << std::endl;

	ref[0] = 10;
	ref[1] = 20;
	ref[2] = 30;
	std::cout << ref[0] << ref[1] << ref[2] << std::endl;

	return 0;
}
```
이 때 `int (&ref)[3]`의 reference는 int형의 크기가 3인 배열만 받을 수 있다.
만약 `int (&ref)[5]`라면 int형의 크기가 5인 배열만 받을 수 있다
   
   
## 3. 상수값(literal)을 받는 Referece
---
### 3-1. 상수값을 Reference로 받는 방법
Reference형으로 리터럴을 받게 되면 오류가 발생한다
```cpp
int& ref = 10;
string& refStr = "Hello";
```
해당 코드는 오류가 난다
왜냐하면 reference형으로 받게되면 수정이 가능하다는 뜻이고
위의 코드의 경우 상수 `10`, 또는 문자열 `Hello`가 있는 Data 공간을 수정할 수 있게된다는 뜻이다.
따라서 리터럴은 Reference로 받을 수 없다

하지만 const & 형으로는 상수값도 받을 수 있다
```cpp
#include <bits/stdc++.h>
using namespace std;

int main()
{
	const int& ref = 10;
	const string& refStr = "Hello";

	return 0;
}
```
위의 코드와 같이 const 를 붙여주면 값을 변경할 수 없게되어서 컴파일에러가 발생하지 않게 됨

### 3-2. 함수의 Return값을 받는 방법
const &형의 또다른 특징으로는 함수의 상수 return 값을 받을 수 있다는 점이다.

> [!danger] 일반 상수값을 Return 하는 함수를 단순 Reference로는 받을 수 없다
> 기본적으로는 함수에서 상수값을 return 한다면 복사로 전달됨
> 복사된 값은 임시객체의 형식으로 해당 줄에서만 생존하기 때문에 Reference로 받게 되면
> Dangling Reference가 된다

> [!tip] const & 사용 시의 함수의 일반 상수도 받을 수 있다
> const &형을 사용하게되면 예외로 함수의 Return 값의 생명이 연장된다.
> 해당 생명은 Reference 함수가 사라질 때까지 연장됨
```cpp
#include <bits/stdc++.h>
using namespace std;

int Func()
{
	return 100;
}

int main()
{
	//int& ref = Func();		//에러발생
	const int& ref = Func();	//정상작동

	cout << ref;
	return 0;
}
```
   
   
## 4. Casting Reference
---
Reference를 Casting 할 수 있다

### 4-1. static_cast
```cpp
#include <bits/stdc++.h>
using namespace std;

class Base {};

class Derived : public Base {};

int main()
{
	Derived d;
	Base& br = d;
	Derived& dr = static_cast<Derived&>(br);
	
	return 0;
}
```

### 4-2. dynamic_cast
> [!tip] pointer의 dynamic_cast와 차이점이 있다
> pointer의 dynamic_cast의 경우 실패하면 null을 return 한다
> reference의 dynamic_cast의 경우 실패하면 bad_cast 예외를 발생시킨다
```cpp
#include <bits/stdc++.h>
using namespace std;

class Base 
{
	virtual void func() {}
};

class Derived : public Base {};

int main()
{
	Derived d;
	Base& br = d;
	
	try
	{
		Derived& dr = dynamic_cast<Derived&>(br);
	}
	catch (const std::bad_cast& e)
	{
		//...
	}
	
	
	return 0;
}
```

### 4-3. const_cast
```cpp
#include <bits/stdc++.h>
using namespace std;

int main()
{
	const int& cref = 5;
	int& ref = const_cast<int&>(cref);
	ref = 15;
	
	return 0;
}
```
   
### 4-4. reinterpret_cast
> [!todo] [[Cast 연산자#1. reinterpret_cast]] 참고
```cpp
#include <bits/stdc++.h>
using namespace std;

int main()
{
	int i = 15;
	int& iref = i;

	//int의 bit를 char변수에다 밀어넣는 행위
	char& cref = reinterpret_cast<char&>(iref);

	return 0;
}
```
