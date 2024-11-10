---
Index: "[[1. CppIndex]]"
tags:
  - Study
  - cpp
---
   
## 1. SFINAE
---
> [!question] Substitution Failure Is Not an Error

치환실패는 에러가 아니다
라는 뜻으로 Template을 사용할 때 타입변경에 실패해도 컴파일 에러가 발생하지 않는다는 뜻이다
   
### 1-1. 예제코드
```cpp
struct Case	//비교용 클래스
{
	using inner_t = int;	//int 타입을 typedef 처럼 재정의
};

template <typename T>
void println(typename T::inner_t t)	//클래스 용 Template
{
	cout << "inner_t is defined: " << t << endl;
}

template <typename T>
void println(T t) //일반 자료형 용 Template
{
	cout << "inner_t is not defined: " << t << endl;
}

void main()
{
	println<Case>(10); //클래스용 Template 호출
	println<int>(20); //일반 자료형 용 Template 호출
}
```
   
### 1-2. 결과
```
inner_t is defined: 10
inner_t is not defined: 20
```
1. println\<Case>(10)
	Case 클래스를 넣어서 클래스 Template이 정상적으로 호출
2. println\<int>(20);
	1. 처음에 클래스 Template을 호출하려고 시도
	2. 하지만 int형에는 inner_t 라는 맴버가 없으므로 타입변경에 실패
	3. 컴파일에러는 발생하지 않음
	4. 일반 자료형 Template을 호출
   
   
## 2. enable_if
---
> [!tip] enable_if 키워드를 통해서 SFINAE를 좀 더 유용하게 적용할 수 있다

enable_if로 template 오버로딩을 하는 구조는 그대로 유지하면서 들어오는 type에 따라 다른 것을 호출
   
**\[enable_if 원형]**
```cpp
template<bool B, class T = void>
struct enable_if {};
 
template<class T>
struct enable_if<true, T> 
{ typedef T type; };
```
enable_if를 호출하면 
1. template<bool B, class T = void> 의 enalbe_if 호출
2. bool B가 true일 경우에 template<class T\> 의 enable_if 호출
3. bool B가 false인 경우는 정의되지 않아서 치환 실패
   
> [!question] template에 bool이 들어가는 상황 참고

1. template 에 자료형을 정하는 상황
```cpp
//template에 자료형을 미리 정해줄 수 있다.
template <bool TestBool, typename T>
struct TestStruct
{
	void Test()
	{
		if (TestBool)
			cout << "TestBool is True" << endl;
	}
};

void main()
{
	TestStruct<true, int> FTest;
	FTest.Test();   // "TestBool is True" 출력

	TestStruct<false, int> FTestFalse;
	FTestFalse.Test();  // 아무것도 출력안함
}
```
   
2. template에 자료형을 정하고 오버로딩을 추가한 상황
```cpp
//class T = void 는 Default 매개변수
template <bool TestBool, class T = void>
struct TestStruct
{ };

//앞에 조건이 참일 때만 들어옴
template <class T>
struct TestStruct<true, T> //오버로딩
{
	void Test()
	{
		cout << "TestBool is True" << endl;
	}
};

void main()
{
	TestStruct<true, int> FTest;
	FTest.Test();   // "TestBool is True" 출력

	TestStruct<false, int> FTestFalse;
	FTestFalse.Test();  // 오류발생
}
```
   
### 2-1. 예제코드
```cpp
using namespace std;

template <typename T>
struct Point
{
	std::string name;
	T x;
	T y;
};

//int형일 때 호출
template <typename T> // void println_point(const Point<T>& point)
typename std::enable_if<std::is_integral_v<T>>::type println_point(const Point<T>& point)
{
	//typeid(T).name() : type의 이름 호출
	cout << point.name << '<' << typeid(T).name() << '>' << " = integral point(" << point.x << ", " << point.y << ")" << endl;
}

//int형이 아닐때 호출
template <typename T>
typename enable_if<!std::is_integral<T>::value>::type println_point(const Point<T>& point)
{
	cout << point.name << '<' << typeid(T).name() << '>' << " = non-integral point(" << point.x << ", " << point.y << ")" << endl;
}

//int형일 때
template <typename T, typename std::enable_if<std::is_integral<T>::value>::type* = nullptr>
bool Check_Integral(const Point<T>& point)
{
	return true;
}

//int형이 아닐때
template <typename T, typename std::enable_if<!std::is_integral<T>::value>::type* = nullptr>
bool Check_Integral(const Point<T>& point)
{
	return false;
}

void main()
{
	ios::sync_with_stdio(0);
	cin.tie(0);

	Point<int> p0{ "p0", 1, 2 };
	Point<long long> p1{ "p1", 3ll, 4ll };
	Point<float> p2{ "p2", 0.1f, 0.2f };
	Point<double> p3{ "p3", 0.3, 0.4 };
	println_point(p0); //println_point<int>(p0); 이지만 <int> 생략 가능
	println_point(p1);
	println_point(p2);
	println_point(p3);

	cout << "\n";

	if (Check_Integral(p0))
		cout << p0.name << " is Integral" << "\n";
	if(!Check_Integral(p2))
		cout << p2.name << " is Not Integral" << "\n";
}
```
> [!tip] 코드상세설명
> ![[KakaoTalk_20240118_132508524.jpg]]
   
### 2-2. 결과
```
p0<int> = integral point(1, 2)
p1<__int64> = integral point(3, 4)
p2<float> = non-integral point(0.1, 0.2)
p3<double> = non-integral point(0.3, 0.4)

p0 is Integral
p2 is Not Integral
```
   
### 2-3. enable_if 축약
```cpp
//축약하지 않고 전부 적을 때
template <typename T>
typename std::enable_if<std::is_integral_v<T>>::type println_point(const Point<T>& point)
{ ... }

//축약해서 적을 때
template<typename T>
typename std::enable_if_t<std::is_integral_v<T>> println_point(const Point<T>& point)
{ ... }
```
   
   
## 참고문헌
---
https://learn.microsoft.com/en-us/cpp/standard-library/enable-if-class?view=msvc-170