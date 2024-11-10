---
Index:
  - "[[1. CppIndex]]"
tags:
  - study
  - cpp
---
   
## 1. Placement New란?
---
미리 할당된 공간에 프로그래머가 객체를 생성 및 초기화를 해줄 수 있는 문법
메모리 풀을 구현할 때 해당 방식을 사용한다
   
   
## 2. Placement New 설명
---
\<new>헤더를 포함시켜줘야 사용 가능

### 2-1. 예제
```cpp
#include <iostream>
#include <new>	//placement new에서 사용

class Widget
{
public:
	Widget(int i) : id(i) { std::cout << "Widget " << id << " Constructed\n"; }
	~Widget() { std::cout << "Widget " << id << " Destroyed\n"; }

	int id;
};

int main() 
{
	char buffer[sizeof(Widget)];	//메모리 풀 만들기
	
	Widget* w = new (buffer) Widget(1);	//Placement New -> 생성자 따로 호출
	std::cout << "Use Widget : " << w->id << "\n";	//메모리풀에 생성한 객체 사용

	w->~Widget();	//소멸자 따로 호출 (delete 사용불가)

	return 0;
}
```

```
Widget 1 Constructed
Use Widget : 1
Widget 1 Destroyed
```

> [!tip] 코드설명

일정한 크기의 버퍼를 만들고 그 뒤에 Placement New를 이용해서 해당 버퍼 내부에 객체를 선언한다
사용한 뒤 객체의 소멸자를 호출
일반적인 new를 이용해서 객체를 생성한 것이 아니기 때문에 delete는 사용 불가능하다
그래서 소멸자를 따로 호출해주어야 함


> [!question] Placement New의 사용이유?

1. 메모리 할당과 new를 분리해서 수행할 수 있다
2. 메모리 풀을 사용함으로서 메모리 할당을 반복적으로 수행해야 할 때 재사용함으로 부하를 줄일 수 있다
3. 원하는 메모리에 객체를 할당할 수 있다
