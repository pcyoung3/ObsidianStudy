---
Index:
  - "[[5. CSIndex]]"
tags:
  - Study
  - CS
---
   
## 1. Spinlock이란?
---
어떤 기능을 실행하는데 lock이 걸려있다면 lock이 풀릴 때까지 무한루프를 돌면서 대기하다가
lock이 풀리면 바로 작업을 실행하는 방법

> [!tip] 장점
> lock이 풀리는 순간 가장 빠르게 작업을 완료할 수 있다
> 커널모드로 주도권을 넘기지 않고 유저모드에서 처리가 완료될 수 있다

> [!failure] 단점
> lock이 언제 풀릴지 모른다
> 무한루프를 돌면서 대기하면 cpu 점유율을 계속 차지하게 됨

해당 방법은 실전에서도 많이 쓰이기 때문에 안쪽이 어떤식으로 구현되어있는지 알아보는 것도 좋다

> [!question] 실전에서는 그럼 뭘 사용할까?
> 대부분의 상황에서는 C++ 공식 API에 있는 std::mutex를 사용하는 게 더 좋다
> 기본적으로 std::mutex는 짧은시간이라면 spinlock이 돌고 시간이 길어지면 thread를 휴먼상태로 만듦(hybrid mutex)
> 여기 있는 코드들은 내부가 대충 어떻게 구성되어 있는지 공부를 위한 의사코드
   
   
## 2. Spinlock 의사코드 구현
---
```cpp
#include <iostream>
#include <thread>
#include <atomic>
#include <mutex>

using namespace std;

class SpinLock
{
public:
	void lock()
	{
		//lock이 열릴 때까지 반복하면서 기다림
		while(_locked)
		{ }
		_locked = true;	//lock이 풀리면 lock을 true로 바꿔주고 닫아버림
	}

	void unlock()
	{
		_locked = false;
	}
private:
	volatile bool _locked = false;
};

int sum = 0;
SpinLock spinLock;
void Add()
{
	for (int i = 0; i < 10000; i++)
	{
		//std::lock_guard<SpinLock> lockguard(spinLock);	//이렇게 사용해도 무관
		spinLock.lock();	//이해하기 편하게 lock 그냥 사용
		sum++;
		spinLock.unlock();
	}
}

void Sub()
{
	for (int i = 0; i < 10000; i++)
	{
		//std::lock_guard<SpinLock> lockguard(spinLock);
		spinLock.lock();
		sum--;
		spinLock.unlock();
	}
}


void main()
{
	std::thread t1(Add);
	std::thread t2(Sub);

	t1.join();
	t2.join();

	cout << sum << endl;
}

```
다음과 같이 spinlock을 우선 구현할 수 있지만 해당 코드는 제대로 작동하지 않는다.
> [!question] 왜 제대로 작동하지 않을까?
> 자물쇠가 열린 것을 인지하는 것과 자물쇠를 잠구는 행동이 원자적으로 일어나지 않았기 때문
> 일단 보이는 것만 해도 while 문이 1줄
> 그리고 \_locked를 바꾸는 것이 1줄 해서 총 2줄이다

> [!tip] 하드웨어 측면에서 분석
> Thread는 RAM 에서는 Stack 영역을 소유하고 나머지 Heap, Code, Data 부분은 공유
> CPU에서는 연산을 하기 위해 Stack영역의 값을 Register에 저장
> Register의 공간은 Thread마다 별도로 차지하게 됨
> 
> A Thread와 B Thread가 있고 lock이라는 변수에 접근한다고 했을 때
> 코드에서 lock은 하나의 변수이지만 CPU에서 연산을 하기 위해서
> 각 Thread의 Register에 해당 변수가 배정된다
> 
> 따라서 Stack의 측면에서 봤을 때 lock에 서로 다른 Thread가 접근하는게 가능하지만
> 좀 더 자세히 보면 CPU의 Register에 각 Thread의 변수가 셋팅되기 때문에 동기화를 해야되는 것

> [!info] volatile 키워드
> c++에서 volatile 키워드는 컴파일러에게 컴파일 시 코드변환을 하지 말아달라는 키워드이다.
> SpinLock에서 while문이 돌 때 컴파일러 입장에서는 의미 없는 while문이라서 해당 코드를 최적화시킨다.
> 하지만 Thread를 다루고 있는 입장에서는 다른 Thread가 건드릴 수 있기 때문에 해당 키워드를 붙인 것
   
   
## 3. Spinlock 문제사항 수정
---
> [!success] 다음과 같이 수정하면 SpinLock이 제대로 되는 것을 확인가능
```cpp
class SpinLock
{
public:
	void lock()
	{
		//CAS(Compare-And-Swap)
		bool expected = false;
		bool desired = true;

		//한줄에 _locked가 false인지 확인하고 false라면 _locked = true를 해주는 함수
		while (_locked.compare_exchange_strong(expected, desired) == false)
		{
			//bool compare_exchange_strong(_TVal& _Expected, const _TVal _Desired ...)
			//위 함수의 expected 인자가 &이므로 while문을 돌때마다 바뀌기 때문에 다시설정
			expected = false;	
		}
	}

	void unlock()
	{
		_locked.store(false);	//_locked = false; 와 동일
	}
private:
	std::atomic<bool> _locked = false;	//_locked를 atomic으로 바꿔 원자적으로 작동하게 함
};
```
   
> [!question] compare_exchange_strong 함수 설명
```cpp
//compare_exchange_strong의 의사코드
//다음 함수를 한줄로 축약한 것
//expected -> 현재 되어있는 예상값 ->위에서보면 false로 고정
//desired -> 이제부터 만드려는 기대값 ->위에서보면 true 고정
bool Locked;
bool CompareExchangeStrong(bool& expected, bool& desired)
{
	if (Locked == expected)	//Locked가 false였다면 true로 바꿔라
	{
		expected = Locked;
		Locked = desired;	//Locked 변경
		return true;
	}
	else	//Locked가 이미 잠겨있거나 경합에 실패한 경우
	{
		expected = Locked;
		return false;
	}
}
```
   
   
## 4. Sleep이란?
---
SpinLock이 계속 무한루프를 돌면서 Lock이 해제되길 기다렸다면
Sleep은 Lock이 걸려있다면 일단 일정시간 기다렸다가 후에 다시와서 Lock이 해제되어있는지 확인하는 방법이다.

예를 들어 Sleep(100ms) 라면 100ms 동안 해당 Thread의 주도권을 내려놓고 100ms 가 지난 후에 
Process Scheduling Queue 에 다시 해당 Process를 넣고 처리를 시작하기 때문에 정확히 100ms 보다 더 소요될 수 있다.
(100ms + 프로세스 스케쥴링이 다시 돌아오는 시간)

> [!abstract] time slice?
> CPU가 프로세스에 실행소유권을 주는 시간
> CPU는 멀티프로세스 환경이기때문에 시간을 정해주고 그 시간만큼만 실행시킨다.

> [!success] 장점
> 커널모드로 주도권을 넘기기 때문에 time slice를 아낄 수 있다 -> 다른 프로그램을 더 많이 실행시킬 수 있음

> [!danger] 단점
> 커널모드로 주도권을 넘기는 거 자체가 Context Switching을 발생시킨다
   
   
## 5. Sleep 의사코드구현
---
위에 [[Thread_4_C++ Spinlock&Sleep#3. Spinlock 문제사항 수정|SpinLock]]에서 Sleep이나 yield만 추가해주면 됨
```cpp
class SleepLock
{
public:
	void lock()
	{
		bool expected = false;
		bool desired = true;
		while (_locked.compare_exchange_strong(expected, desired) == false)
		{
			expected = false;
			this_thread::sleep_for(std::chrono::milliseconds(1));
			//this_thread::sleep_for(1ms);	//1ms는 이런식으로도 표현가능
			//this_thread::yield();	//주도권을 커널로 넘기는 방식
		}
	}

	void unlock()
	{
		_locked.store(false);
	}
private:
	std::atomic<bool> _locked = false;
};
```