---
Index:
  - "[[CppIndex]]"
tags:
  - study
  - cpp
---
   
## 1. Type이 아닌 Template 인자값
---
Template에 Type이 아닌 값을 전달해줄 수 있다.
```cpp
#include <iostream>

template <typename T, int num>
T add_num(T t) 
{
	return t + num;
}

int main() 
{
	int x = 3;
	std::cout << "x : " << add_num<int, 5>(x) << std::endl;
}
```
template으로 int num을 전달해줄 수 있다.

전달해줄 수 있는 타입은 다음과 같다
- 정수 타입들 (`bool`, `char`, `int`, `long` 등등). 당연히 `float` 과 `double` 은 제외
- 포인터 타입
- `enum` 타입
- `std::nullptr_t` (널 포인터)

> [!info] 이런걸 어디다 쓸까?
> c++은 일반 배열을 함수의 인자값으로 전달하려면 포인터형식과 사이즈를 전달해줘야해서 불편한데
> 이 부분을 이용하면 다음과 같이 쓸 수 있다.
```cpp
#include <iostream>
#include <array>

template <typename T>
void print_array(const T& arr) 
{
	for (int i = 0; i < arr.size(); i++) 
		std::cout << arr[i] << " ";

	std::cout << std::endl;
}

int main() 
{
	//std::array를 이용하고 배열의 타입과 size를 전달
	std::array<int, 5> arr = { 1, 2, 3, 4, 5 };
	std::array<int, 7> arr2 = { 1, 2, 3, 4, 5, 6, 7 };
	std::array<int, 3> arr3 = { 1, 2, 3 };

	print_array(arr);
	print_array(arr2);
	print_array(arr3);
}
```
   
### template의 의존타입(dependent type)
template에 값과 타입이 전부 들어갈 수 있기 때문에 컴파일러 입장에서 종종 들어오는 값이
값인지 타입인지 불분명해진다. 이러한 것을 template의 의존타입이라고 함
```cpp
#include <iostream>

/*
* <오류발생코드>
* 이렇게만 정의가 되어있다면 p도 불분명하고
* 또한 T가 타입인지 값인지 모름
* template에 들어가는 인자가 값인지 타입인지 알려주어야 오류가 발생하지 않음
*/
//template <typename T>
//void func()
//{
//	T::t* p;
//}

int p = 3;
template <typename T>
void func1()	//template은 기본적으로 값으로 우선 생각함
{
	T::t* p;	//p가 정의되어있으므로 해당 문장은 들어온 t*3;의 뜻으로 해석
	std::cout << "this is Func1" << std::endl;
}

template <typename T>
void func2()
{
	typename T::t* p;	//p가 typename이라고 명명되어있으므로 int p;로 해석
	std::cout << "this is Func2" << std::endl;
}

class A
{
public:
	const static int t;
};

class B
{
public:
	using t = int;
};

int main()
{
	func1<A>();
	func2<B>();
}
```
> [!tip] 값과 타입의 우선관계
> template은 값과 타입이 뭔지 모르는 상황이 오면 값을 우선시 해서 컴파일한다.
   
   
## 2. Default Template 인자
---
Default 매개변수처럼 Template에도 기본값을 넣어줄 수가 있다.
```cpp
#include <iostream>

template <typename T, int num = 5>
T add_num(T t)
{
	return t + num;
}

int main()
{
	int x = 3;

	//add_num에 3과 default로 5가 들어감
	std::cout << "x : " << add_num(x) << std::endl;
}
```
> [!todo] add_num(x)에서 template기호가 빠진 이유
> 컴파일러가 템플릿 타입추론을 자동으로 해줘서 가능하다
> 다만 함수형 Template에만 가능하고 class template에서는 다른 처리를 해주어야 함
   
   
## 3. 가변길이 Template
---
template을 정의할 때 인자값을 가변형태로 정의할 수 있다.
이렇게 되면 여러개의 인자값을 한번에 처리가능하다.

### 3-1. 예시 1
```cpp
#include <iostream>

//기본템플릿이 먼저 정의되어 있어야 함
template <typename T>
void print(T arg) 
{
	std::cout << arg << std::endl;
}

//오버로딩 형태로 다음이 정의가 됨
//typename... Types : Template Parameter Pack
template <typename T, typename... Types>
void print(T arg, Types... args)	//함수 파라미터 팩
{
	std::cout << arg << ", ";
	print(args...);	//재귀적으로 호출
}

int main() 
{
	print(1, 3.1, "abc");
	print(1, 2, 3, 4, 5, 6, 7);
}
```
> [!abstract] 정의할 때 알아두어야 하는 포인트
> 1. 기본 template이 정의가 되어있어야 한다 (= 재귀의 BaseCondition)
> 2. 가변인자 template은 `typename... Types`로 작성
> 3. 가변인자 template을 받는 함수타입은 `Types... args`로 작성
> 4. 재귀호출을 시켜서 인자를 하나씩 처리한다

> [!question] 처리과정 정리 - 재귀식으로 호출되는 것을 참고
```cpp
//1. 함수호출
print(1, 3.1, "abc");

//2. 첫번째 재귀
void print(int arg, double arg2, const char* arg3) 
{
	std::cout << arg << ", ";
	print(arg2, arg3);
}

//3. 두번째 재귀
void print(double arg, const char* arg2) 
{
	std::cout << arg << ", ";
	print(arg2);
}

/*
* template <typename T, typename... Types>
* void print(T arg, Types... args)
* 세번째 재귀에서 이 형태도 호출이 가능
* 하지만 C++ 규칙상 파라미터 팩이 없는 함수의 우선순위가 높아서
* 인자 1개의 함수가 호출됨
*/
//4. 세번째 재귀
template <typename T>
void print(T arg)
{
	std::cout << arg << std::endl;
}
```
   
### 3-2. 예시2
> [!success] 참고사항
> 첫번째) sizeof... 는 들어오는 인자의 전체 개수를 반환
> 두번째) 함수이름이 달라도 가변인자는 정상작동 할 수 있다
```cpp
#include <iostream>

// 재귀 호출 종료를 위한 베이스 케이스
int sum_all() 
{ 
	return 0; 
}

template <typename... Ints>
int sum_all(int num, Ints... nums) 
{
	return num + sum_all(nums...);
}

//재귀에 들어가는 함수 이름이 달라도 가변인자는 정상작동
template <typename... Ints>
double average(Ints... nums) 
{	//sizeof...는 인자의 전체 개수를 반환
	return static_cast<double>(sum_all(nums...)) / sizeof...(nums);
}

int main() 
{
	// (1 + 4 + 2 + 3 + 10) / 5
	std::cout << average(1, 4, 2, 3, 10) << std::endl;
}
```
   
> [!tip] C++ 17부터 다음과 같이 사용 가능 - Fold 형식 이용
> 여기서 사용된 Fold 형식은 단항좌측이며 단항우측, 이항우측, 이항좌측의 형식이 있다.
```cpp
#include <iostream>

int sum_all()
{
	return 0;
}

template <typename... Ints>
int sum_all(Ints... nums)
{
	return (... + nums); //C++ 17부터 가능한 Fold 형식
}

template <typename... Ints>
double average(Ints... nums)
{
	return static_cast<double>(sum_all(nums...)) / sizeof...(nums);
}

int main()
{
	// (1 + 4 + 2 + 3 + 10) / 5
	std::cout << average(1, 4, 2, 3, 10) << std::endl;
}
```
   
   
## 4. 템플릿 메타 프로그래밍 Template Meta Programming(TMP)
---
template을 잘 활용하면 런타임이 아닌 컴파일 타임에 값을 계산할 수 있다.
타입은 컴파일타임에 확정이 되어야 하므로 template을 이용하면 컴파일 타임에 연산을 할 수 있다.

> [!success] 장점
> 컴파일때 모든 연산이 끝나기 때문에 프로그램 실행속도를 향상시킬 수 있다.

> [!failure] 단점
> 프로그래밍이 굉장히 복잡해진다
> 컴파일타임에 연산하기 때문에 디버깅을 하지 못함
> 템플릿 오류시에 에러코드가 굉장히 길다. 즉, 버그를 찾는 것이 힘들다.
   
> [!info] Factorial을 TMP를 이용해서 짠 코드
> [[Template 심화문법#1. Type이 아닌 Template 인자값]] 참고
```cpp
#include <iostream>

//Type이 아닌 Template 인자값
template <int N>
struct Factorial 
{
	static const int result = N * Factorial<N - 1>::result;
};

//템플릿 특수화 + 재귀의 Base Condition
template <>
struct Factorial<1>
{
	static const int result = 1;
};

int main() {
	std::cout << "6! = 1*2*3*4*5*6 = " << Factorial<6>::result << std::endl;
}
```