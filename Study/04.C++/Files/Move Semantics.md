# Index : [[CppIndex]]
## Tag : #study #cpp
---
   
> [!tip] Modern Cpp에서 추가된 개념 중 중요한 Rvalue Lvalue 에 관해 서술

## 1. Lvalue? Rvalue?
---
### 1-1. Lvalue
Lvalue 는 주소값을 가질 수 있는 값을 의미한다.
지금까지 사용해왔던 변수는 보통 Lvalue라고 생각하면 된다.
   
### 1-2. Rvalue
Rvalue는 주소값을 가질 수 없는 값을 의미한다.
일반적으로는 상수값, 임시객체, 람다 등 다음줄이면 없어질 값들을 말한다
```cpp
int a = 3; //a는 lvalue, 3은 rvalue
customclass A = customclass(1); // A는 lvalue, 임시객체인 오른쪽은 rvalue
```
   
   
## 2. 이동연산자 및 이동생성자
---
### 2-1. 이동연산자
Rvalue를 변수에 담을 수 있는데 이 담는 형식을 위해 이동연산자가 사용된다.
이동연산자는 '&&' 형태로 레퍼런스를 2개 붙인 모양이다.
   
### 2-2. 이동생성자
c++11 부터 이동연산자를 이용하여 객체의 Default 로 생성되는 이동생성자가 추가되었다.

|                    복사                    |                     이동                     |
|:------------------------------------------:|:--------------------------------------------:|
| ![[KakaoTalk_20230613_183343437.jpg\|390]] | ![[KakaoTalk_20230613_183359997 2.jpg\|450]] |
   
### 2-3. 코드
```cpp
class Person
{
public:
	Person() 
	{ 
		value = new int(1);
		cout << "생성자" << endl; 
	}
	~Person() 
	{ 
		delete value;
		cout << "소멸자" << endl; 
	}
	Person(const Person& _p)
	{
		value = new int;
		*value = *_p.value;	//깊은복사
		cout << "복사생성자 호출" << endl;
	}
	Person(Person&& _p) noexcept
	{
		value = _p.value;	//얕은복사
		_p.value = nullptr;	//임시객체 삭제될 시 원본값 사라지지 않게 pointer 초기화
		cout << "이동생성자 호출" << endl;
	}
	void test() { cout << "This is Test\n"; }
	
private:
	int* value;
};
void main()
{
	Person p = Person();	//생성자
	cout << "---------------------" << endl;
	Person copyPerson = Person(p);	//복사생성자
	cout << "---------------------" << endl;
	Person movePerson = Person(std::move(p));	//이동생성자
	cout << "---------------------" << endl;
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
		cout << "이동생성자 호출" << endl;
	}
	...
};

Person CreatePerson()
{
	Person temp = Person();
	return temp;
}
void main()
{
	Person MovePerson = CreatePerson();	//이동생성자 호출

	vector<Person> vec;
	vec.push_back(Person());	//이동생성자 호출
}
```
이동생성자가 2곳에서 호출
1) CreatePerson() 함수에서 생성(지역변수) 후 return 값이 임시객체이므로 MovePerson에 대입할 때 이동생성자가 호출
2) stl에서 사용되는 함수에 임시객체를 넣었을 경우 이동생성자가 호출

> [!tip] 
return 할때 또는 매개변수로 전달할 때 복사생성자가 호출되었지만 이제 이동연산자가 생성되는걸로 바뀌어서 생기는 이점은 비용이 큰 동적할당 연산을 줄일 수 있는 것이다.
   
> [!danger] 정리
> 이동생성자가 호출되는 경우
> 1. std::move() 함수를 통해 객체생성
> 2. 함수에서 return 값을 받아서 객체생성
> 3. STL에 인자로 객체를 넘길 때
   
   
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

void valueTest(Person& _input) { cout << "lValue" << endl; }
void valueTest(const Person& _input) { cout << "const lValue" << endl; }
void valueTest(Person&& _input) { cout << "RValue" << endl; }

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