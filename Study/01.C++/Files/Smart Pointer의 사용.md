---
Index:
  - "[[1. CppIndex]]"
tags:
  - study
  - cpp
---
   
## 0. 참고문서
---
[[Smart Pointer]] 해당 문서 먼저 숙련 후 참고
   
   
## 1. 사용상의 주의사항
---
스마트포인터를 사용하면 temlpate란에 \*를 붙이지 않지만 포인터로 선언되는 것을 항상 염두해야 한다.

> [!example] int\*를 float\*로 받으려는 예제
```cpp
shared_ptr<int> iTest = make_shared<int>(3);
shared_ptr<float> fTest1 = make_shared<float>(static_cast<float>(*iTest));	//컴파일 성공 -> 값을 가져와서 float으로 형변환 뒤 float*에 넣음
//shared_ptr<float> fTest2 = static_pointer_cast<float*>(iTest);	//컴파일 에러 -> iTest는 int*인데 float**로 변환하려는 것
//shared_ptr<float> fTest3 = static_pointer_cast<float>(iTest);	//컴파일 에러 -> void* 형도 아닌데 float*로 int*를 가르키려 함
//shared_ptr<float> fTest4 = iTest;	//컴파일 에러 -> 형변환도 안하고 총체적 난국
```

> [!example] void\*를 스마트포인터로 선언
```cpp
shared_ptr<void> pData = make_shared<T>(Data);
```
   
   
## 2. 스마트 포인터 형변환
---
스마트 포인터를 형변환 할 때 기존에 존재하던 [[Cast 연산자|C++의 형변환]]을 사용할 수가 없다
스마트 포인터 전용의 형변환을 사용해야 한다.

기존에 존재하던 것과 같이
> [!abstract] static_pointer_cast, dynamic_pointer_cast, const_pointer_cast 

와 같은 casting 이 추가되었다.

### 예시코드
```cpp
#include <bits/stdc++.h>
using namespace std;

class Base
{
public:
	int a;
	virtual void f() const { std::cout << "Base 클래스\n"; }
	virtual ~Base() {}
};

class Derived : public Base
{
public:
	void f() const override { std::cout << "Derived 클래스!\n"; }
	~Derived() {}
};

int main()
{
	auto basePtr = std::make_shared<Base>();
	std::cout << "Base 포인터: ";
	basePtr->f();

	auto derivedPtr = std::make_shared<Derived>();
	std::cout << "Derived 포인터: ";
	derivedPtr->f();

	//basePtr에 derivedPtr을 대입
	basePtr = std::static_pointer_cast<Base>(derivedPtr);
	std::cout << "derived를 Base로 업캐스팅 static 형변환: ";
	basePtr->f();

	//downcastedPtr <- basePtr <- derivedPtr
	auto downcastedPtr = std::dynamic_pointer_cast<Derived>(basePtr);
	if (downcastedPtr)
	{
		std::cout << "Base를 Derived 로 Dynamic 다운캐스팅: ";
		downcastedPtr->f();
	}

	//derivedPtr의 포인터를 derivedPtr, basePtr, downcastedPtr 3개의 포인터가 사용
	std::cout << "Pointers to underlying derived: "
		<< derivedPtr.use_count()
		<< '\n';
}
```

### 결과
```
Base 포인터: Base 클래스
Derived 포인터: Derived 클래스!
derived를 Base로 업캐스팅 static 형변환: Derived 클래스!
Base를 Derived 로 Dynamic 다운캐스팅: Derived 클래스!
Pointers to underlying derived: 3
```
   
   
## 3. 스마트포인터의 this
---
this는 포인터이고 스마트포인터 중 shared_ptr은 참조되는 수를 Reference Count를 이용해서 세어준다.
따라서 this를 같이 사용한다면 shared_ptr 참조를 하나 더 늘리는 형식이 된다.

그래서 shared_ptr을 사용하면 this를 사용할 수 없는데
그럼에도 불구하고 this를 사용할 일이 많기 때문에 스마트포인터를 사용한다면 this를 다른 방식으로 사용해야 한다.

> [!tip] enable_shared_from_this 를 상속시킨 뒤 shared_from_this 함수로 this를 불러옴
```cpp
#include <bits/stdc++.h>
using namespace std;

//enable_shared_from_this 상속
class Object : public std::enable_shared_from_this<Object>
{
public:
	bool IsSame(std::shared_ptr<Object> obj);
};

bool Object::IsSame(std::shared_ptr<Object> obj)
{
	if (obj == shared_from_this())	//shared_from_this는 shared_ptr의 this
		return true;

	return false;
}

int main()
{
	shared_ptr<Object> a = make_shared<Object>();
	shared_ptr<Object> b = make_shared<Object>();

	if (a->IsSame(b))
		cout << "같다!" << endl;
	else
		cout << "다르다!" << endl;
}
```
   
   
## 4. weak_ptr의 사용
---
[[Smart Pointer#3. weak_ptr|weak_ptr의 사용]] 해당 문서도 참고
weak_ptr은 약한 참조를 거는 스마트 포인터인데 상호참조 상황일 때를 대비할 수 있다.
다만 막상 실제로 사용하려고 하면 헷갈릴 수가 있기 때문에 정리

```cpp
#include <bits/stdc++.h>
using namespace std;

class Object
{
public:
	Object() {}
	Object(int input) { test = input; }
	~Object() {}
public:
	int test;
};

int main()
{
	std::shared_ptr<Object> objShared = std::make_shared<Object>(3);
	cout << "변경 전 :" << objShared->test << endl;

	std::weak_ptr<Object> objWeak = objShared;	//weak_ptr에 shared_ptr 넣기

	if (!objWeak.expired())	//weak_ptr이 가지고 있는 shared_ptr이 유효한지?
	{
		//expired와 같이 lock을 사용해서 받을 경우 없으면 nullptr이라 if문에서 활용 가능
		if (auto objPtr = objWeak.lock())
			objPtr->test = 5;

		objWeak.reset();	//weak_ptr이 가지고 있는 shared_ptr 주소를 nullptr로 만듦
	}

	cout << "변경 후 :" << objShared->test << endl;
}
```