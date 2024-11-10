---
Index:
  - "[[1. CppIndex]]"
tags:
  - study
  - cpp
---
   
## 1. constexpr
---
참고링크 : https://modoocode.com/293
> [!info] 상수 선언으로 해당 객체나 함수의 리턴값을 컴파일 타임에 알 수 있게 만드는 키워드
> 컴파일러가 컴파일 타임에 어떤 식의 값을 결정할 수 있다면 이를 상수식(Constant expression)이라고 부름
> 상수식 중에서 정수인 값들을 정수상수식이라고 부름
> 정수상수식은 배열, 템플릿 타입인자, enum 선언등에서 쓰일 수 있다.
```cpp
#include <iostream>

template <int N>
struct A 
{
	int operator()() { return N; }
};

int main() 
{
	constexpr int size = 3;
	int arr[size];  // Good!

	constexpr int N = 10;
	A<N> a;  // Good!
	std::cout << a() << std::endl;

	constexpr int number = 3;
	enum B { x = number, y, z };  // Good!
	std::cout << B::x << std::endl;
}
```

## 2. const와 constexpr
---
> [!info] 차이점
> const가 더 큰 개념이고 constexpr을 포함한다고 보면 된다.
> 즉, `constexpr` 은 항상 `const` 이지만, `const` 는 `constexpr`이 아니다
```cpp
int main() 
{
	//const는 초기화를 변수로 시킬 수도 있다.
	//즉, 런타임에 그 값이 결정될 수 있다.
	int a = 1;
	const int ca = a;
	//constexpr int cea = a;	//Error
	constexpr int cea = 1;
}
```

   
   
## 3. constexpr 함수
---
```cpp
#include <iostream>

//함수에 constexpr을 넣어서 컴파일 시 상수화
constexpr int Factorial(int n) 
{
	int total = 1;
	for (int i = 1; i <= n; i++)
		total *= i;

	return total;
}

int main() 
{
	std::cout << Factorial(5) << std::endl;
}
```
> [!info] 함수에 constexpr을 넣게 되면..?
> 함수를 상수화 시키게 된다. 즉, 컴파일 시점 연산이 가능
> 인자 또는 내부에서 호출하는 함수들이 전부 constexpr 형태일 경우 무사히 해당 함수도 상수화 된다.

> [!danger] 만약에 인자로 상수가 아닌 일반 변수를 넘겼다면?
> 그러면 함수가 상수화 함수가 아니게 되지만 동작은 한다!!
> 따라서 constexpr을 붙여줄 수 있다면 붙이는 것이 좋다. 컴파일 타임 상수를 생성할 수 있다면 간단하게 이용가능하기 때문

> [!warning] constexpr 제약조건 (C++ 14부터)
- `goto` 문 사용
- 예외 처리 (`try` 문; C++ 20 부터 가능)
- 리터럴 타입이 아닌 변수의 정의
- 초기화 되지 않는 변수의 정의
- 실행 중간에 `constexpr` 이 아닌 함수를 호출
   
> [!question] 리터럴 타입 -> 전부 constexpr이 가능
- `void` 형
- 스칼라 타입 (`char, int, bool, long, float, double`) 등등
- 레퍼런스 타입
- 리터럴 타입의 배열
- 혹은 아래 조건들을 만족하는 타입
	- 디폴트 소멸자를 가지고
	- 다음 중 하나를 만족하는 타입
		- 람다 함수
		- `Arggregate` 타입 (사용자 정의 생성자, 소멸자가 없으며 모든 데이터 멤버들이 `public`)
		    쉽게 말해 `pair` 같은 애들을 이야기함
		- `constexpr` 생성자를 가지며 복사 및 이동 생성자가 없음
   
   
## 4. if constexpr
---
> [!tip] 특징
> if constexpr은 조건이 반드시 bool로 타입 변환될 수 있어야 하는 컴파일 타임 상수식
> if constexpr가 참일 경우 else에 해당하는 문장은 **컴파일되지 않음**
> if constexpr가 거짓일 경우 if에 해당하는 문장은 **컴파일되지 않음**

if constexpr을 사용해서 컴파일을 아예 실행하지 않아서 무사히 컴파일이 되게만든 코드
```cpp
#include <iostream>
#include <type_traits>

template <typename T>
void show_value(T t) 
{
	//if constexpr 사용 -> constexpr이 없으면 컴파일이 안됨
	if constexpr (std::is_pointer<T>::value)
		std::cout << "포인터 이다 : " << *t << std::endl;
	else
		std::cout << "포인터가 아니다 : " << t << std::endl;
}

int main() 
{
	int x = 3;
	show_value(x);

	int* p = &x;
	show_value(p);
}
```
   
> [!quote] constexpr이 없으면 컴파일 에러가 발생
> 템플릿 인자를 넣으면 다음과 같은데 3이라는 상수에 포인터를 붙였기 때문
```cpp
template <typename int>
void show_value(int t)
{
	if constexpr (std::is_pointer<int>::value)
		std::cout << "포인터 이다 : " << *t << std::endl;  //여기서 t가 3이라면 *3 으로 치환되어서 에러가 발생
	else
		std::cout << "포인터가 아니다 : " << t << std::endl;
}
```
