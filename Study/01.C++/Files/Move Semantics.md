---
Index:
  - "[[1. CppIndex]]"
tags:
  - Study
  - cpp
---
   
> [!tip] Modern Cpp에서 추가된 개념 중 중요한 Rvalue Lvalue 에 관해 서술

## 1. Lvalue? Rvalue?
---
### 1-1. Lvalue
Lvalue 표현식 이후에도 사라지지 않는 값.
지금까지 사용해왔던 변수는 보통 Lvalue라고 생각하면 된다.
   
### 1-2. Rvalue
Rvalue는 표현식 이후에 사라지는 값들.
일반적으로는 상수값, 임시객체, 람다 등 다음줄이면 없어질 값들을 말한다
```cpp
int a = 3; //a는 lvalue, 3은 rvalue
customclass A = customclass(1); // A는 lvalue, 임시객체인 오른쪽은 rvalue
```
좀 더 세세한 값의 체계는 다음을 참고 [[auto와 decltype#2. C++의 값의 종류(value Category)|value category]]
   
   
## 2. 이동연산자 및 이동생성자
---
### 2-1. 이동연산자
Rvalue를 변수에 담을 수 있는데 이 담는 형식을 위해 이동연산자가 사용된다.
이동연산자는 '&&' 형태로 레퍼런스를 2개 붙인 모양이다.
   
### 2-2. 이동생성자
c++11 부터 이동연산자를 이용하여 객체의 Default 로 생성되는 이동생성자가 추가되었다.

|                  복사                   |                   이동                    |
| :-----------------------------------: | :-------------------------------------: |
| ![[KakaoTalk_20230613_183343437.jpg]] | ![[KakaoTalk_20230613_183359997 2.jpg]] |
   
### 2-3. 코드
```cpp
class Person
{
public:
	Person()
	{
		value = new int(1);
		std::cout << "생성자" << std::endl;
	}

	~Person()
	{
		delete value;
		std::cout << "소멸자" << std::endl;
	}

	Person(const Person& _p)
	{
		value = new int;
		*value = *_p.value;	//깊은복사
		std::cout << "복사생성자 호출" << std::endl;
	}

	Person(Person&& _p) noexcept
	{
		value = _p.value;	//얕은복사
		_p.value = nullptr;	//임시객체 삭제될 시 원본값 사라지지 않게 pointer 초기화
		std::cout << "이동생성자 호출" << std::endl;
	}

	Person& operator=(const Person& _p)
	{
		value = new int;
		*value = *_p.value;
		std::cout << "대입연산자 호출" << std::endl;
		return *this;
	}

	Person& operator=(Person&& _p) noexcept
	{
		value = _p.value;
		_p.value = nullptr;
		std::cout << "이동대입연산자 호출" << std::endl;
		return *this;
	}

	void test() { std::cout << "This is Test\n"; }

private:
	int* value;
};

void main()
{
	Person p = Person();	//생성자
	std::cout << "---------------------" << std::endl;
	Person copyPerson = Person(p);	//복사생성자
	std::cout << "---------------------" << std::endl;
	Person movePerson = Person(std::move(p));	//이동생성자
	std::cout << "---------------------" << std::endl;
}
```
   
> [!warning]  주의
> 해당 함수는 rvalue를 인자값으로 받지만 \_p 변수 자체는 lvalue이다
```cpp
Person(Person&& _p) noexcept
```
> [!question] noexcept
> noexcept 키워드는 예외를 발생시키지 않겠다고 명시하는 키워드
> c++ STL에 이동생성자가 정의된 클래스를 담으려고 할 때 해당 키워드가 있지 않으면
> 이동생성자가 호출되지 않고 복사생성자가 호출된다.
>    
> 복사생성의 경우 예외가 발생하면 새로 동적할당된 공간을 그냥 날려버려도 기존 메모리가 남아있지만
> 이동생성의 경우 기존 주소값을 어딘가로 옮기는 과정에서 예외가 일어났기 때문에 일부만 옮겨진 기존공간을 해제하기 애매하기 때문이다
   
```cpp
Person movePerson = Person();
```
> [!tip] 참고
> 억지스러운 상황을 가정하고 위와 같은 코드일 때는 이동생성자가 호출될까?
> -> 정답은 그냥 기본생성자가 호출되고 끝난다
> 이동생성자를 호출하고 싶다면 std::move()를 사용
   
### 2-4. 기본이동생성자 생성조건
기본 이동생성자는 소멸자, 복사생성자, 대입연산자를 사용자 정의할 경우 기본생성되지 않는다.
![[KakaoTalk_20230613_175014293.jpg|500]]
   
   
## 3. 사용처
---
```cpp
Person movePerson = Person(std::move(p));	//이동생성자
```
이 예시로는 사실 이동연산을 왜 해야하는지 잘 감이 오지 않는다.
사실 이동연산이 가장 많이 쓰이는 곳은 함수의 인자값이나 return 값을 사용될 때다.
   
```cpp
class Person
{
public:
	...
	Person(Person&& _p) noexcept
	{
		value = _p.value;	//얕은복사
		_p.value = nullptr;	//임시객체 삭제될 시 원본값 사라지지 않게 pointer 초기화
		std::cout << "이동생성자 호출" << std::endl;
	}
	...
};

Person CreatePerson()
{
	return Person(); //일반생성자 호출
}

void main()
{
	Person MovePerson1 = CreatePerson();  //아무 생성자도 호출하지 않고 그대로 값복사됨 -> 복사생성자도 아님

	Person MovePerson2 = std::move(CreatePerson());	//이동생성자호출

	Person MovePerson3;
	MovePerson3 = CreatePerson();	//이동대입연산자 호출

	std::vector<Person> vec;
	vec.push_back(Person());	//이동생성자 호출
}
```

코드설명
1) MovePerson1의 경우 CreatePerson() 함수에서 return 되는 것은 임시객체이지만 그걸 그대로 받아서 집어넣어서 추가로 생성자 호출이 없음
2) MovePerson2의 경우 std::move를 이용해서 이동연산을 했기 때문에 이동생성자 호출
3) MovePerson2의 경우 생성자에 바로 넣은 것이 아니라 이미 주소가 할당된 상황에서 임시객체를 넣었으므로 이동대입연산자가 호출됨
	(이동대입연산자가 정의가 안되어있으면 대입연산자 호출)
4) stl에서 사용되는 함수에 임시객체를 넣었을 경우 이동생성자가 호출

> [!tip] 
return 할때 또는 매개변수로 전달할 때 복사생성자가 호출되었지만 이제 이동연산자가 생성되는걸로 바뀌어서 생기는 이점은 비용이 큰 동적할당 연산을 줄일 수 있는 것이다.
   
> [!danger] 정리
> 이동생성자가 호출되는 경우
> 1. std::move() 함수를 통해 객체생성
> 2. STL에 인자로 객체를 넘길 때
   
   
## 4. std::move, std::forward
---
이동 연산을 제대로 수행하기 위해서 2가지의 함수가 사용된다.
std::move와 std::forward인데 사용방법은 다음과 같다.
   
### 4-1. std::move
```cpp
wrapper(std::move(MovePerson));
```
std::move가 하는 일은 실제로 이동연산을 해주는 것이 아니라 들어오는 인자값을 무조건 rvalue로 형변환한다
   
### 4-2. std::forward
```cpp
Person wrapper(T&& input)
{
	valueTest(std::forward<T>(input));
	...
```
forward의 경우 현재 들어온 값이 rvalue일 때만 rvalue로의 형변환을 진행한다
무슨 소리인지 이해가 안가니 다시 말하면 인자로 rvalue 타입이 들어와도 인자 자체는 lvalue이기 때문에 전달해줄 때 lvalue가 전달되므로
이럴 때  rvalue형태로 전달할 때 사용
   
   
## 5. Universal Reference(Forwarding Reference)
---
rvalue를 인자값으로 받을 때 rvalue와 lvalue를 동시에 다 받는 자료형이 있다.
이 자료형은 템플릿 타입 추론(Type Deduction)에서 선언된다.
```cpp
template<class T>
Person wrapper(T&& input) //이 부분이 Universal Reference
{
	valueTest(std::forward<T>(input));
	return input;
}

void valueTest(Person& _input) { std::cout << "lValue" << std::endl; }
void valueTest(const Person& _input) { std::cout << "const lValue" << std::endl; }
void valueTest(Person&& _input) { std::cout << "RValue" << std::endl; }

void main()
{
	Person MovePerson;
	wrapper(MovePerson);			//복사생성자 호출
	wrapper(std::move(MovePerson));	//이동생성자 호출
	wrapper(Person());				//이동생성자 호출
}
```
T&& 형태는 Universal Reference형태로 선언된다.
Universal Reference는 lvalue와 rvalue를 동시에 다 받을 수 있어서 함수 오버로딩을 해주지 않아도 됨
> [!danger] const T&& 같이 조금이라도 형식이 달라지면 Universal Reference는 선언되지 않는다.

Universal Reference로 값을 전달할 때는 forward를 사용해서 전달하고
rvalue로 넘겨주길 원할 때는 move를 사용해서 전달
   
   
## 6. push_back, emplace_back의 차이점
---
### 6-1. push_back의 경우
```cpp
template <class _Ty, class _Alloc = allocator<_Ty>>
class vector {
	...
	_CONSTEXPR20 void push_back(_Ty&& _Val)
	...
```
   
### 6-2. emplace_back의 경우
```cpp
template <class _Ty, class _Alloc = allocator<_Ty>>
class vector {
	...
    template <class... _Valty>
    _CONSTEXPR20 decltype(auto) emplace_back(_Valty&&... _Val) {
	...
```
   
### 6-3. 차이점
push_back의 경우 vector를 처음 선언할 때 Template에 자료형을 입력하면 그 자료형이 push_back까지 그대로 적용된다
```cpp
class vector<TestClass, allocator<testClass>> {
	push_back(TestClass&& _Val)
}
```
emplace_back의 경우 vector가 처음 선언되었을 때 자료형과 무관하게 여전히 template 함수이다
```cpp
class vector<TestClass, allocator<testClass>> {
	...
	template <class... _Valty>
	emplace_back(_Valty&&... _Val)
```
push_back의 경우 이미 타입추론이 끝난상황이기 때문에 Rvalue를 인자값으로 받는 것이지 Universal Reference가 아니다
emplace_back의 경우 class의 타입추론과 상관없이 함수를 호출할 때 타입추론이 발생한다.
또한 그 타입은 T&& 형식과 같기 때문에 emplace의 경우 Universal Reference 형식으로 매개변수를 받는다.

> [!note] 또 다른 차이점
> emplace_back의 경우 보면 `class...` 인 구문이 있는데, 이는 가변인자를 받는다는 것이다. [[Template 심화문법#3. 가변길이 Template | 가변길이 Template 참고]]
> 따라서 push_back에 객체를 전달한다면 임시객체나 복사생성자를 이용해서 넣어야 한다
> 하지만 emplace_back은 해당 객체의 생성자의 인자값들을 전달해줄 수 있기 때문에 안쪽에서 생성자를 한번만 호출하게 됨
#### 예제코드
```cpp
#include <bits/stdc++.h>
using namespace std;

class A
{
public:
	A() 
	{
		a = 0;
		b = 0;
		c = 0;
		cout << "생성자 호출" << endl;
	}
	~A()
	{
		cout << "소멸자 호출" << endl;
	}
	A(int _a, int _b, int _c) : a(_a), b(_b), c(_c)
	{ 
		cout << "생성자 호출" << endl;
	}

	A(const A& _input)
	{
		a = _input.a;
		b = _input.b;
		c = _input.c;
		cout << "복사생성자 호출" << endl;
	}

	A(A&& _input) noexcept
	{
		a = _input.a;
		b = _input.b;
		c = _input.c;
		cout << "이동생성자 호출" << endl;
	}

private:
	int a = 0;
	int b = 0;
	int c = 0;
};
void main()
{
	std::vector<A> v1, v2;
	v1.emplace_back(1, 2, 3); //바로 생성자를 넣어줄 수 있음
	cout << "----------------------------" << endl;
	v2.push_back(A(3, 4, 5)); //생성자 바로 넣는 것이 안되고 임시객체를 이용해줘야 함
	cout << "----------------------------" << endl;
}
```
   
#### 출력
```
생성자 호출
----------------------------
생성자 호출
이동생성자 호출
소멸자 호출
----------------------------
소멸자 호출
소멸자 호출
```