---
Index:
  - "[[CsIndex]]"
tags:
  - Study
  - CS
---
   
## 1. DeadLock이란?
---
mutex로 lock을 건뒤에 모종의 이유로 lock이 풀리지 않아서 무한대기 상태에 빠지는 것을 의미
   
   
## 2. DeadLock 이 걸리는 이유 1
---
해당문서 참고 : [[Thread_C++ 기초#DeadLock|DeadLock 이유]]
다음과 같이 Lock을 해주고 Unlock을 수행하지 않았을 경우 DeadLock 상태에 빠지게 된다.
해결법은 lock_guard를 쓰면 된다.
   
   
## 3. DeadLock 이 걸리는 이유 2 - 다른순서로 Lock 걸기
---
다음은 DeadLock이 걸리는 코드이다.
```cpp
#include <iostream>
#include <thread>
#include <vector>
#include <atomic>
#include <mutex>

using namespace std;


std::mutex mutexA;
std::mutex mutexB;

void A()
{
	for (int i = 0; i < 10000; i++)
	{
		//A Lock -> B Lock
		std::lock_guard<std::mutex> LockGuardA(mutexA);
		//Do Something...
		std::lock_guard<std::mutex> LockGuardB(mutexB);
	}
	
}

void B()
{
	for (int i = 0; i < 10000; i++)
	{
		//B Lock -> A Lock
		std::lock_guard<std::mutex> LockGuardB(mutexB);
		//Do Something...
		std::lock_guard<std::mutex> LockGuardA(mutexA);
	}
}

void main()
{
	std::thread t1(A);
	std::thread t2(B);

	t1.join();
	t2.join();
}
```
> [!bug] 해당 코드의 문제점은 Lock 2개를 서로 다른 순서로 호출한다는 것이다.
> Lock을 걸때 mutex가 여러개 즉, 자물쇠가 여러개 있다면 자물쇠를 거는 순서가 같아야 DeadLock을 피할 수 있다.
> 해당 코드는 다른 Thread 2개가 각자의 자물쇠를 잡고 다른 자물쇠가 풀릴 때까지 무한대기하는 상황이다.

> [!success] Lock의 순서를 맞춰주면 제대로 프로그램이 종료됨
```cpp
void A()
{
	for (int i = 0; i < 10000; i++)
	{
		//A Lock > B Lock
		std::lock_guard<std::mutex> LockGuardA(mutexA);
		//Do Something...
		std::lock_guard<std::mutex> LockGuardB(mutexB);
	}
	
}

void B()
{
	for (int i = 0; i < 10000; i++)
	{
		//A Lock > B Lock
		std::lock_guard<std::mutex> LockGuardA(mutexA);
		//Do Something...
		std::lock_guard<std::mutex> LockGuardB(mutexB);
	}
}
```
   
해결방법은 딱히 없다. 조심하게 프로그래밍 해야함
