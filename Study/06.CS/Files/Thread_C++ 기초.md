---
Index:
  - "[[CsIndex]]"
tags:
  - Study
  - CS
  - cpp
---
   
## 1. C++ Thread의 사용
---
> [!info] 기본적인 Thread 사용법에 관한 코드
```cpp
#include <iostream>
#include <thread>
using namespace std;

void HelloThread()
{
	std::cout << "Hello Thread" << endl;
}
void main()
{
	std::thread t(HelloThread);

	cout << "Hello Main" << endl;

	int count = t.hardware_concurrency(); //코어개수 반환
	std::thread::id tId = t.get_id();     //thread id 반

	if (t.joinable())	//쓰레드가 join 가능한 상태인지?
		t.join();		//쓰레드가 끝날때까지 대기하면서 기다림

	//std::thread 객체에서 실제 쓰레드를 분리 -> linux의 Demon을 생각하면 편함
	//분리하면 해당 Thread의 정보를 얻을 수 없기 때문에 활용성은 떨어짐
	t.detach();	//백그라운드 쓰레드로 변경

}
```
   
> [!question] 인자가 있는 Thread나 STL에 넣어서도 가능하다
```cpp
#include <iostream>
#include <thread>
#include <vector>

using namespace std;

void HelloThread(int input)
{
	std::cout << input << endl;
}
void main()
{
	std::vector<std::thread> v;	//vector에 Thread 형
	for (int i = 0; i < 10; i++)
		v.push_back(std::thread(HelloThread, i));
	//인자 있는 thread는 가변인자 형태로 넣어주면 됨

	for (int i = 0; i < 10; i++)
	{
		if (v[i].joinable())
			v[i].join();
	}
}
```
   
   
## 2. Thread의 동기화 - Atomic
---
> [!danger] Thread에서 동기화하지 않을 경우
> Thread에서 서로 공유되는 자원 (대표적으로는 전역변수)을 건드릴 때 동기화를 필수로 해주어야 한다.
> 하지 않으면 Thread에 의해서 값이 중간중간 서로 간섭하면서 원하는 결과가 나오지 않는다

> [!tip] Atmoic은 꽤나 병목현상이나 리소스를 먹기 때문에 남발하면 안됨
### 코드
```cpp
#include <iostream>
#include <thread>
#include <vector>
#include <atomic>

using namespace std;

atomic<int> sum = 0;	//atomic을 사용한 변수
int nonAtomicSum = 0;	//atomic을 사용하지 않은 변수

void Add()
{
	for (int i = 0; i < 1000000; i++)
	{
		sum++;
		nonAtomicSum++;
	}
}

void sub()
{
	for (int i = 0; i < 1000000; i++)
	{
		sum--;
		nonAtomicSum--;
	}
}

void main()
{
	std::thread t1(Add);
	std::thread t2(sub);

	if (t1.joinable())
		t1.join();
	if (t2.joinable())
		t2.join();

	cout << sum << endl;
	cout << nonAtomicSum << endl;
}
```
   
### 결과
```
0
16047
```
아래에 출력되는 값은 매번 랜덤
> [!question] 결과값이 이런 이유
> atmoic을 사용한 경우 all or nothing으로 원자화 되어서 해당값을 어떤 Thread가 수정하고 있다면 다른 Thread는 건드릴 수 없게된다.
> 
> 하지만 사용하지 않을 경우 ++연산이나 --연산이 한줄로 되어있는 것 같겠지만 
> 사실 어셈블리 안쪽을 보면 2~3개 정도의 연산을 하게 된다.
> 2~3번의 연산 중에 thread 끼리 서로의 값을 건드릴 수 있기 때문에 계산 결과가 덮어씌워지거나 해서
> 제대로 된 연산이 되지 않는 것이다.
   
   
## 3. Thread의 동기화 - Lock
---
> [!info] mutex 사용
> 대표적으로 STL 처럼 기본자료형이 아니라면 atomic이 지원이 안된다. (내부 클래스의 함수등을 사용할 수 없음)
> 따라서 Lock을 걸어서 사용해줘야 한다.
> 이럴때는 mutex를 사용해줘서 Lock과 Unlock을 실행해줘야 함

### lock, unlock 코드
> [!warning] 이 코드에서 lock과 unlock을 하지 않으면...
> crush가 나면서 프로그램이 종료된다. 그 이유는 vector가 capacity를 늘릴 때
> 1. 현재 있는 vector의 메모리를 날리고
> 2. 새로운 메모리를 할당받은 다음에
> 3. 기존 vector의 값을 복사
> 
> 의 작업을 하는데 당연하게도 push_back의 기능이 1줄이 아니기 때문에 여러 thread가 실행하면
> 저 위의 단계를 여러 thread가 나누어서 실행하게 된다.
> 그렇게 되면 메모리 장소라던가 댕글링 포인터들이 발생하기 때문에 프로그램이 종료된다
```cpp
#include <iostream>
#include <thread>
#include <vector>
#include <atomic>
#include <mutex>

using namespace std;

vector<int> v;
mutex m;

void Push()
{
	for (int i = 0; i < 10000; i++)
	{
		m.lock();			//lock을 걸어서 다른 thread 접근 못하게 막음
		v.push_back(i);
		m.unlock();			//unlock으로 다른 thread의 접근을 다시 열어줌
	}
}

void main()
{
	std::thread t1(Push);
	std::thread t2(Push);

	t1.join();
	t2.join();

	cout << v.size() << endl;
}
```

### DeadLock
다음 코드에도 약점이 있는데 lock을 걸면 무조건 unlock을 걸어야 한다는 것이다.
여기서 unlock을 까먹게 되면 deadlock 현상이 발생한다.
따라서 프로그램이 종료되지 않고 무한루프로 join을 계속 기다리게 된다.
하지만 lock과 unlock 사이에 기능구현이 많이 된다고 하면 (함수를 계속 부르거나 새로운 Thread를 생성하거나..)
그 모든 단계를 파악하기 힘들어진다.

> [!success] lock_guard를 사용
```cpp
void Push()
{
	for (int i = 0; i < 10000; i++)
	{
		std::lock_guard<std::mutex> lockGuard(m);
		v.push_back(i);
	}
}
```
lock_guard를 사용하면 wrapper class로서 알아서 코드블록이 끝나면 unlock을 시켜주기 때문에 안전하게 사용가능

> [!quote] Lock guard의 구현
```cpp
//lock과 unlock을 위한 wrapper class
//이런걸 RAII 패턴이라고도 함
template <typename T>
class LockGuard
{
public:
	LockGuard(T& m)	//생성자에서는 mutex를 받아서 lock을 바로 실행
	{
		_mutex = &m;
		_mutex->lock();
	}

	~LockGuard()	//소멸자에서 unlock을 걸어준다.
	{
		_mutex->unlock();
	}
	//이렇게 할 경우 코드블록을 빠져나가면 스택메모리가 정리되면서
	//자동으로 소멸자를 호출하게 되므로 unlock의 세트를 맞추지 않아도 됨

private:
	T* _mutex;
};
```
Lock Guard는 대략적으로 다음과 같이 구현되어있다.