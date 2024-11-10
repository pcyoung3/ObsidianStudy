---
Index:
  - "[[1. CppIndex]]"
tags:
  - study
  - cpp
---
   
## 1. RVO와 NRVO란?
---
> [!note] RVO = Return Value Optimization

> [!note] NRVO = Named Return Value Optimization

객체가 함수에서 반환될 때 이동, 복사 없이 완료되게 하는 최적화
해당 동작은 컴파일러에서 실행된다.
   
   
## 2. RVO
---
```cpp
#include <iostream>
using namespace std;

class Test
{
public:
	Test() { }
	Test(const Test&) { cout << "Call Copy Constructor\n"; }
	Test(Test&&) noexcept { cout << "Call Move Constructor\n"; }
};

Test CreateTest()
{
	return Test();
}

int main() 
{
	Test value = CreateTest();

	return 0;
}
```

> [!tip] 해당 코드의 결과
> 일반 생성자가 불리기 때문에 아무 로그도 발생하지 않음

> [!question] Test(const Test&) 부분에 관한 설명
> 매개변수가 정의되지 않아도 컴파일이 가능하다. 함수내부에서 매개변수를 쓰지 않으면 생략해도 무방

해당 코드를 보면 복사생성자와 이동생성자가 호출되었을 때 로그가 발생함
main에서 CreateTest 함수를 호출하고 CreateTest 함수 내부에서 Test 임시객체를 생성
이 때 Test의 생성자가 호출됨

그리고 CreateTest 의 retrun 값으로 Test를 반환할 때 보통은 복사나 이동생성자가 이곳에서 호출된다
하지만 임시객체를 바로 반환할 경우 컴파일러에서 추가적으로 복사, 이동 연산 없이 바로 value에 대입하게 됨

이런 방식을 RVO 라고 함
   
   
## 3. NRVO
---
```cpp
Test CreateTestNRVO()
{
	Test returnValue;
	return returnValue;
}

int main() 
{
	Test valueNRVO = CreateTestNRVO();

	return 0;
}
```

객체에 이름이 있을 때는 NRVO 가 발생해서 최적화된다
해당 코드도 복사, 이동생성자가 호출되지 않는다
   
   
## 4. RVO, NRVO가 일어나지 않는 경우
---
if문을 이용해서 다중으로 반환되거나 일부 예외적인 상황에서는 RVO, NRVO가 발생하지 않는다

### 다중반환의 예시
```cpp
Test CreateTest()
{
	Test t1, t2;
	if (flag)
		return t1;
	else
		return t2;
}
```
컴파일러가 t1, t2 중 어떤것이 반환될지 알 수 없기 때문에 NRVO가 발생하지 않음

