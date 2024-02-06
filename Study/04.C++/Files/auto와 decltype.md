---
Index:
  - "[[CppIndex]]"
tags:
  - study
  - cpp
---
   
## 1. auto의 타입추론
---
auto 는 기본적으로 C++의 긴 타입들을 짧게 변환할 수 있지만
auto의 타입추론은 몇가지의 상황에서 100%로 정밀하지 않다.
그리고 auto의 타입추론은 template의 타입추론과 똑같다.

### auto가 다른 타입추론을 하는 경우
- `const`, `&`, `volatile`은 읽지 않는다.
- 다만 대입된 것이 포인터일 경우 `const`, `*`까지 다 읽어들인다.
- `auto &`은 `const`를 읽는다.
- `auto &&`은 Universal Reference이다.
- 람다 함수는 파라미터에 `auto`를 사용할 수 있다.
   
> [!example] 예시
```cpp
#include <iostream>

int main()
{
	int a = 3;
	auto a_a = a;	//a_a는 int형

	const int& cri = 3;
	auto a_cri = cri;			//a_ca는 int형-> const를 반영하지 않음
	const auto ca_cri = cri;	//ca_ca는 const int형
	auto& ra_cri = cri;			//ra_cri는 const int&형
	const auto& cra_cri = cri;	//cra_cra는 const int& 형

	const int* pa = new int(3);
	auto a_pa = pa;	//a_pa는 const int* 형

	delete(pa);
}
```
   
   
## 2. C++의 값의 종류(value Category)
---
C++에서는 식을 표현할 때 3가지의 값의 종류를 가지고 있다.
몇 가지의 기준을 가지고 나눠지는데 기준에 대해서 알아보자면 다음과 같다

> [!note] 정체를 알 수 있는가? (identity)
> 식이 2개가 있으면 그 2개를 비교해서 2개가 같은지 다른지 확인할 수 있는 식들을 의미한다
> 대표적으로 lvalue 또는 함수가 정체를 알 수 있는 값들이다.
```cpp
#include <iostream>

int main()
{
	int a = 3;

	int* pa1 = &a;
	int* pa2 = &a;

	if (pa1 == pa2)
		std::cout << "identity 값들이라 비교가능" << std::endl;
}
```
   
> [!note]  이동할 수 있는가? (move)
> 해당 식에 이동(move sementic)이 가능한 식들을 의미한다.
> 일반적으로 알고있는 rvalue의 특성이다.
```cpp
#include <iostream>

int f() { return 10; }
int main()
{
	int a = 3;	//'3' 이 이동가능

	//int b = &&a;	//에러발생 -> a를 이동시킬 수 없다
	
	bool b = true;	//'true'가 이동가능
	f();	//f()는 이동가능
}
```

|                       |  이동시킬 수 있다   | 이동시킬 수 없다 |
|:---------------------:|:-------------------:|:----------------:|
| **정체를 알 수 있다** |  xvalue(eXpiring)   |      lvalue      |
| **정체를 알 수 없다** | pvalue(Pure rvalue) |        X         |
![[16.3.1.png]]
glvalue(Generalized lvalue)
   
> [!note] 이동할 수도 있고 정체도 알 수 있는 xvalue?
> lvalue와 prvalue 2개만 있다면 좌측값으로 분류되는 식을 이동시킬 방법이 없다.
> 따라서 좌측값과 같이 정체가 있지만 이동도 시킬 수 있는 값들이 존재한다.
> 
> 대표적인 예시로 std::move 가 있다. 들어오는 값을 rvalue로 형변환 해주는 것인데 return값을 보면
> 이동가능하지만 변수로 return 하기 때문에 2개의 특성을 다 가지고 있다
```cpp
bool b = true;
bool&& c = std::move(b);
///////////////////////////////////////////////////////
constexpr remove_reference_t<_Ty>&& move(_Ty&& _Arg) noexcept
```
   
   
## 3. decltype
---
declared type의 준말로서 decltype 키워드를 쓰며 2가지 용도로 쓸 수 있다.

> [!tip] 첫번째, 괄호로 둘러쌓이지 않은 식별자 표현식(id-experssion)이라면 해당 식의 타입을 얻을 수 있다.
> 컴파일 타임에 타입이 추론

> [!question] 식별자표현식?
> 괄호로 둘러쌓이지 않은 식
> 어떤 연산을 하지않고 객체 하나만을 가르키는 식
^a23ee8

```cpp
#include <iostream>

int main()
{
	int a = 3;
	const int ca = a;
	const int& cra = ca;

	decltype(a) da = a;			//da는 int
	decltype(ca) dca = ca;		//dca는 const int
	decltype(cra) dcra = cra;	//dcra는 const int&

	std::cout << std::endl;
}
```
auto와의 차이점에 주목하자 auto는 때에 따라서 const와 & 등의 반영이 안되는 수식어들이 많았지만
decltype은 모든 타입들을 그대로 반영해준다.
   
또한, 템플릿과 같이 요긴하게 쓰일 수 있다.
템플릿에서 t + u 는 어떤 인자가 들어오냐에 따라서 다르게 결정된다. 이런것이 처리가 가능하다

> [!todo] decltype(auto) 예시
```cpp
#include <iostream>

//decltype을 포인터형으로 밖으로 빼는 코드
template <typename T, typename U>
void add1(T t, U u, decltype(t + u)* result) 
{
	*result = t + u;
}

//C++ 11부터 가능
template <typename T, typename U>
auto add2(T t, U u) -> decltype(t + u)
{
	return t + u;
}

//C++ 14부터 가능 -> 한층 더 줄일 수 있음
template <typename T, typename U>
decltype(auto) add3(T t, U u)
{
	return t + u;
}

int main()
{
	float ans = 0.f;
	add1(2, 3.f, &ans);

	add2(2, 3.f);
	
	add3(2, 3.f);
}
```
   
> [!tip] 두번째, 식을 전달하면 값의 종류에 따라 달라진다.
> - 만일 식의 값 종류가 xvalue 라면 `decltype` 는 `T&&`
> - 만일 식의 값 종류가 lvalue 라면 `decltype` 는 `T&` 
> - 만일 식의 값 종류가 prvalue 라면 `decltype` 는 `T`
```cpp
#include <iostream>

int main()
{
	int a, b;
	decltype(a + b) da;		//da는 int

	int c;
	decltype((c)) db = c;	//da는 int&로 추론
}
```
`decltype((c))` 부분을 보면 괄호로 둘러쌓여있기 때문에 id-expression([[auto와 decltype#^a23ee8|식별자표현식]])이다.
또한 c는 int이고 이동불가능하기 때문에 lvalue가 된다 따라서 `T&`가 적용되서 `int&`로 추론된다.
   
   
## 참고자료
---
https://modoocode.com/294
https://ansohxxn.github.io/cpp/chapter19-8/
https://blog.seulgi.kim/2017/06/cpp11-value-category.html
