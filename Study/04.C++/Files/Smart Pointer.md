---
Index: "[[CppIndex]]"
tags:
  - Study
  - cpp
---
   
> [!bug] C 기반의 Raw pointer에서 일어나는 여러가지 문제점을 해소하기 위해 나온 포인터
> 1. 객체가 자동으로 해제되지 않는다(GC의 부재)
> 2. 객체가 삭제가 되었지만 해당 주소를 가리키고있는 포인터는 그대로 남아있는 문제(댕글링 포인터)

## 1. unique_ptr
---
어떤 객체에 소유권을 단 하가지만 가지고 있을 수 있는 포인터
```cpp
std::unique_ptr<Person> ptr = make_unique<Person>();
std::unique_ptr <Person> ptr2 = ptr;	//에러발생
Person* ptr2 = ptr;	//에러발생
std::unique_ptr<Person> ptr2 = std::move(ptr);	//소유권 이전
```
move를 이용해서 소유권을 이전할 수 있다. 그냥 대입하면 에러가 발생한다.
   
> [!tip] unique_ptr의 가장 큰 특징은 객체의 유일한 소유권을 지니고 있는 점
> 따라서 레퍼런스형으로 넘기기는 되지만 추천하지 않는다. 차라리 포인터형으로 주소만 넘기자
   
   
## 2. shared_ptr
---
레퍼런스 카운트를 가지고 있어서 해당 객체를 몇 개의 포인터가 참조하고 있는지 알 수 있다.
```cpp
	shared_ptr<Person> ptr1 = make_shared<Person>();
	shared_ptr<Person> ptr2 = ptr1;
	cout << "shared ptr : " << ptr2.use_count() << endl;	//reference count 출력
	ptr1 = nullptr;		//reference count 감소
	ptr2.reset();		//reference count 감소
```
하지만 서로를 참조할 때 순환참조의 이슈가 있다.
![[Pasted image 20230609210721.png]]
이 경우 서로 참조를 하고 있어서 상대방이 해제를 해도 reference count가 감소되지 않아서 메모리 해제가 안됨
   
   
## 3. weak_ptr
---
위와 같이 순환참조의 문제를 해결하기 위해 도입된 포인터이다.
객체를 참조해주지만 Reference count를 증가시키지 않는다.
그 자체로는 사용할 수 없고 shared_ptr로 변환해서 사용해야 함
```cpp
shared_ptr<Person> p1 = make_shared<Person>();
weak_ptr<Person> p2 = p1;
bool isExpired = p2.expired(); //해당 weakptr이 유효한지?

//사용하기 위해서는 lock 함수를 이용해서 shared_ptr로 변경한 뒤 사용한다.
{
	std::shared_ptr<Person> testPtr = p2.lock();
	
	if (0 != testPtr.use_count())
		testPtr->test();
}
p2.reset();
```
   
   
## 4. Smart Pointer를 new 로 선언하는 것보다 make를 선호해야하는 이유
---
### 4-1. 타자량이 적어진다
```cpp
auto upw1(std::make_unique<Person>());    //make_ 사용
std::unique_ptr<Person> upw2(new Person); //new 사용 -> Person 2번 사용

auto spw1(std::make_shared<Person>());    //make_ 사용
std::shared_ptr<Person> spw2(new Person); //new 사용 -> Person 2번 사용
```
Person 클래스를 new로 선언할 경우 2번씩 작성해줘야 함
   
### 4-2. 메모리 누수 위험
```cpp
processWidget(std::shared_ptr<Widget>(new Widget), DoSomething())
```
이 경우 실행시점 이후 Object Code로 번역할 때 함수들의 인수가 먼저 평가됨
* new Widget
* std::shared_ptr<\Widget>()
* DoSomething()
이 3개의 인수가 차례대로 실행되지 않기 때문에 누수 위험이 있다.
new 가 먼저 실행되고 DoSomething 이후에 shared_ptr이 호출된다고 가정했을 때
new 실행 후 예외가 발생한다면 메모리 누수가 발생된다.
   
```cpp
processWidget(std::make_shared<Widget>(), DoSomething())
```
이 경우에는 어떤 것이 먼저 실행되고 예외가 발생한다고 해도 메모리 누수가 발생하지 않음
   
### 4-3. shared_ptr의 메모리 할당 횟수
std::shared_ptr은 reference count를 비롯한 여러가지 관리 자료를 담는 제어블록을 담는 포인터가 있고 선언할 때 메모리를 할당한다.
new를 사용할 경우 shared_ptr의 제어블록을 위한 할당 1번, 
그리고 만드려는 타겟객체(widget 같은)를 위한 할당 1번이 발생
make_shared를 사용할 경우 할당을 2번하지않고 한번에 다 처리하는 이점이 있다.
   
> [!tip] 언리얼의 스마트 포인터
> [[이득우C++_09_Serialization(직렬화)#2. Unreal의 스마트 포인터 시스템]]
   
   
## 5. Smart Pointer 내부구조
---
### 5-1. Reference Count 구현의 2가지 방법
1. 상속 
	* 상속으로 BaseClass를 만들고 Refcnt를 생성 후 나머지 Class를 전부 상속시킨 뒤에 SharedPtr로 Wrapping해서 구현
	* 이미 만들어진 Class들은 적용 불가라는 단점이 존재
	* Wrapping 을 굳이 하는 이유는 Thread Safety 때문
	```cpp
	A a = new A();
	A b = a;
	//이 부분에서 다른 Thread가 a를 삭제시킬 경우
	a->AddRef(); //Dangling Pointer 참조가 되어버림
	```

	```cpp
	//Base Class
	class RefCountable
	{
	public:
		RefCountable() : _refCount(1) { }
		virtual ~RefCountable() { }
	
		int32 AddRef() { return ++_refCount; }
		int32 ReleaseRef()
		{
			int32 refCount = --_refCount;
			if (refCount == 0)
			{
				delete this;
			}
			return refCount;
		}
	
	protected:
		atomic<int32> _refCount; //Thread Safety를 위한 atomic 선언
	};
	```

	```cpp
	//Wrapping Class
	template<typename T>
	class TSharedPtr
	{
	public:
		TSharedPtr() { }
		TSharedPtr(T* ptr) { Set(ptr); }
	
		// 복사
		TSharedPtr(const TSharedPtr& rhs) { Set(rhs._ptr); }
		// 이동
		TSharedPtr(TSharedPtr&& rhs) { _ptr = rhs._ptr; rhs._ptr = nullptr; }
		// 상속 관계 복사
		template<typename U>
		TSharedPtr(const TSharedPtr<U>& rhs) { Set(static_cast<T*>(rhs._ptr)); }
	
		~TSharedPtr() { Release(); }
	
	public:
		// 복사 연산자
		TSharedPtr& operator=(const TSharedPtr& rhs)
		{
			if (_ptr != rhs._ptr)
			{
				Release();
				Set(rhs._ptr);
			}
			return *this;
		}
	
		// 이동 연산자
		TSharedPtr& operator=(TSharedPtr&& rhs)
		{
			Release();
			_ptr = rhs._ptr;
			rhs._ptr = nullptr;
			return *this;
		}
	
	
	private:
		inline void Set(T* ptr)
		{
			_ptr = ptr;
			if (ptr)
				ptr->AddRef();
		}
	
		inline void Release()
		{
			if (_ptr != nullptr)
			{
				_ptr->ReleaseRef();
				_ptr = nullptr;
			}
		}
	
	private:
		T* _ptr = nullptr;   //Wrapping을 위한 원본 객체
	};
	```

	```cpp
	TSharedPtr<A> a(new A());
	A->ReleaseRef(); //위 코드 구조상 생성자에서 무조건 1이라 감소시켜야되는데 원본코드는 당연히 이렇지 않음
	
	TSharedPtr<A> b = a; //복사생성자 호출
	b = nullptr; //대입연산자 호출 > Release 됨
	```

2. Wrapping
	* 기본 C++ Smart Pointer가 이 구조로 이루어져 있음
	![[KakaoTalk_20231116_211629518.jpg]]
	* shared_ptr 내부에는 객체를 받아줄 수 있는 ptr과 refcnt를 받을 수 있는 refblock 포인터 공간 2개가 있음
	* make_shared의 경우 2공간을 한꺼번에 할당해주기 때문에 new로 할 경우 T에 한번 refcnt block에 한번해서 2번해야 되는 것보다 저비용으로 가능
	* RefcountBlock은 또 2가지의 변수를 가지고 있다. Uses와 Weaks
		* uses는 shared_ptr과 관련된 참조횟수
		* weaks는 weak_ptr과 관련된 참조횟수
	* weakptr은 expired와 lock을 이용해서 댕글링을 검사할 수 있음
	* weakptr는 uses가 0이되면 담고 있던 객체를 날리고 block은 남겨둠. weaks가 0이되면 block도 날라감

