---
Index:
  - "[[5. CSIndex]]"
tags:
  - Study
  - CS
---
   
## 1. Event
---
> [!INFO] 동기화 할 때 최적화를 위한 Flag
> signal , non-signal 의 상태가 있어서 해당 상태가 아니면 자동 또는 수동으로 
> 들어온 Thread를 Sleep 해주는 기능을 가지고 있어서 과도한 Spinlock이 되지않게 해줌

[[Thread_7_C++ Thread 운영방식#2. Producer - Consumer Model (생산 소비 모델)|Producer - Consumer Model]]  같은 패턴을 구현했을 때 Event를 사용할 수 있다

> [!warning] Event는 유저모드가 아닌 커널모드로 동작
> 즉, Context Switching 이 발생
> 따라서 Spinlock 처럼 자주 확인하게 되는 Thread의 경우 [[가상메모리_Swapping#3. Thrashing|Threshing]] 이 발생할 수도 있다
   
### 예시코드
```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <queue>
#include <windows.h>

using namespace std;

mutex m;
queue<int> q;
HANDLE handle;

void producer()
{
	while (true)
	{
		{
			unique_lock<mutex> lock(m);
			q.push(100);
		}

		//Event의 상태를 Signal로 변경하는 함수
		::SetEvent(handle);

		this_thread::sleep_for(100ms);
	}
}

void consumer()
{
	while (true)
	{
		//이 곳에 Thread가 들어오고 Event의 Signal에 따라서
		//계속 진행할 수도 Sleep으로 변경될 수도 있다.
		//대기시간은 무한으로 설정
		::WaitForSingleObject(handle, INFINITE);
		//::ResetEvent(handle);	//Manual일 때는 해당 함수를 통해서 non-signal로 교체

		std::cout << "consumer 함수" << std::endl;
		{
			unique_lock<mutex> lock(m);
			if (q.empty() == false)
			{
				int data = q.front();
				q.pop();
				cout << data << std::endl;
			}
		}

		this_thread::sleep_for(100ms);
	}
}

int main()
{
	//Event 생성
	//2번째 인자 : Reset을 Auto 로 할 것인가 Manual로 할 것인가
	//3번째 인자 : 시작 상태를 Signal로 할 것인가 Non-Signal로 할 것인가
	handle = ::CreateEvent(nullptr, false, false, nullptr);

	thread t1(producer);
	thread t2(consumer);

	t1.join();
	t2.join();

	//Event 반납
	::CloseHandle(handle);

	return 0;
}
```
   
   
### 코드설명
> [!info] Event 생성
```cpp
#include <windows.h>

HANDLE handle;
...

handle = ::CreateEvent(nullptr, false, false, nullptr);

```
커널모드의 기능을 사용하는 것이기 때문에 Event를 만들고 Handle을 받아온다

CreateEvent의 각 인자를 살펴보면
1번째인자는 보안속성인데 신경 X
2번째인자는 대기함수에서 Thread가 Signal인 상태로 들어오면 자동으로 non-signal로 바꿔줄 것인가 아니면 수동으로 바꿔줄 것인가를 정함
3번째인자는 시작할 때 signal로 시작(true) 할 것인지 non-signal로 시작(False)할 것인지를 정함

> [!info] ::WaitForSingleObject(handle, INFINITE);
```cpp
	::SetEvent(handle);
	
	...
	
	::WaitForSingleObject(handle, INFINITE);
	::ResetEvent(handle);
```
#### SetEvent
Event를 Signal 상태로 변경

#### ResetEvent
Event를 Non-signal 상태로 변경 
위에서 Event를 Auto 상태로 변경해줬다면 자동으로 Non-signal 상태가 되므로 사용 안해도 됨

#### WaitForSingleObject
Thread가 들어왔을 때 Event가 non-signal 상태라면 Thread를 Sleep 상태로 만듦
Sleep 상태의 Thread가 있을 때 Event가 다른 곳에서 Signal 상태가 되었다면 Sleep 상태의 Thread를 깨워서 다시 함수를 시작
Sleep 으로 만든 Thread의 대기시간은 속성으로 줄 수 있다. 무한일 경우 `INFINITE`

> [!question] 이렇게 해서 얻는 이점?
```cpp
void consumer()
{
	while (true)
	{
		std::cout << "consumer 함수" << std::endl;
		{
			//이 부분에서 Spinlock 처럼 Lock을 얻을 때까지 반복
			unique_lock<mutex> lock(m);
			if (q.empty() == false)
			{
				int data = q.front();
				q.pop();
				cout << data << std::endl;
			}
		}
		this_thread::sleep_for(100ms);
	}
}
```
만약 Event를 사용하지 않는다면 Spinlock 과 같은 효과가 발생한다
Lock 안에 작업이 빨리 끝나고 대기시간을 최소화 해야 한다면 Spinlock이 좋지만
Thread의 들어오는 주기가 빠르지 않다면 Spinlock은 CPU의 점유율을 상승시킴

따라서 이럴 때 Event를 사용하게 되면 CPU 점유율을 줄일 수 있다.
   
   
## 2. condition_variable
---
> [!INFO] Event 기법에 조건을 추가한 C++ 공용 방식
> 위의 Event에 더해서 조건식을 추가로 검사할 수 있는 방식이다
> condition_variable은 user-level object 이다

Event와 비슷한 방식으로 흘러간다
C++ 11에서 추가된 기능

#### Producer
1. Lock을 잡는다
2. 공유자원을 수정
3. Lock을 푼다
4. 조건변수를 통해 다른 Thread에 통지

#### Consumer
1. Lock을 잡는다
2. 조건을 확인
	* True : 아래로 빠져나와서 코드를 진행
	* false : Lock을 풀어주고 대기상태

### 예시코드
```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <queue>
#include <windows.h>

using namespace std;

mutex m;
queue<int> q;
//조건변수 condition_variable
condition_variable cv;

void producer()
{
	while (true)
	{
		{
			unique_lock<mutex> lock(m);
			q.push(100);
		}

		cv.notify_one();	//wait 중인 Thread가 있으면 1개를 깨움
		//cv.notify_all();	//wait 중인 Thread가 있으면 전부 깨움
		this_thread::sleep_for(100ms);
	}
}

void consumer()
{
	while (true)
	{
		std::cout << "consumer 함수" << std::endl;
		{
			unique_lock<mutex> lock(m);
			//1번 인자 : 현재 걸린 lock
			//2번 인자 : 조건 (함수 -> 조건자) 람다나 함수를 넣어주면 됨
			cv.wait(lock, []() { return q.empty() == false; });
			{
				int data = q.front();
				q.pop();
				cout << data << std::endl;
			}
		}

		this_thread::sleep_for(100ms);
	}
}

int main()
{
	thread t1(producer);
	thread t2(consumer);

	t1.join();
	t2.join();

	return 0;
}
```

조건을 추가로 걸 수 있어서 Event보다 범용성이 좋다
게다가 Event는 Kernel Object이고 조건변수는 user object라서 성능상 이점도 있다
따라서 C++ 11에서 추가된 조건변수를 사용하는 것이 더 이득이라고 볼 수 있다.

### 코드설명
```cpp
cv.notify_one();
```
Thread 중 1개를 sleep 상태에서 꺠우는 함수
즉, 위와 같이 Event를 Signal 상태로 변경

```cpp
cv.wait(lock, []() { return q.empty() == false; });
```
1. wait받은 lock을 잡으려고 시도함. (위에서 lock이 잡혀있으면 그냥 넘어감)
2. 조건을 체크, 조건을 만족할 경우 코드를 진행시킨다
3. 조건을 체크, 조건을 만족하지 않을 경우 lock을 풀고 Thread를 대기상태로 만듦
   
   
## 3. Spurious wakeup
---
> [!INFO] Thread가 Wakeup 했지만 Lock이 걸려있어서 아무것도 못하는 상황
> Event를 거는 행위와 Lock을 거는 행위는 서로 독립적으로 돌아가기 때문에 발생하는 상황

우선 이것을 이해하기 위해서는 Thread의 실행순서는 정말로 Random이라는 걸 다시 한번 숙지해야 한다
> [!danger] 예제코드에 주의할 점
```cpp
void producer()
{
	while (true)
	{
		{
			unique_lock<mutex> lock(m);
			q.push(100);
		}

		::SetEvent(handle);
	}
}

void consumer()
{
	while (true)
	{
		::WaitForSingleObject(handle, INFINITE);

		{
			unique_lock<mutex> lock(m);
			if (q.empty() == false)
			{
				q.pop();
				cout << q.size() << std::endl;
			}
		}
	}
}
```
해당 코드를 실행시키면 `q.size()` 가 무한으로 증가하는 것을 볼 수 있다
**즉, producer를 돌리는 Thread와 consumer를 돌리는 Thread는 같은 속도로 돌지 않는다**

얼핏보면 producer 함수와 consumer 함수가 1번씩 돌아서 동기화가 될 것 같지만
서로 속도가 다르기 때문에 Queue에 Data가 계속 쌓인다

그래서 위에 [[Thread_8_C++ Event, condition_variable#예시코드|Event 예시코드]]에서 Sleep 시간으로 어느정도 프로그램의 진행상황을 동기화 한 것이다

> [!question] Spurious wakeup?
> 바로위에서 알아봤듯이 producer와 consumer 함수의 실행속도가 다르다
> 따라서 다음 예시와 같은 상황이 일어날 수 있다
> ![[KakaoTalk_20241102_205006370.jpg]]
> Lock을 거는 행위와 Signal을 받는 행위는 독립적이라는 것을 명심

> [!success] Spurious wakeup 방지방법
> lock을 얻었다고 해서 바로 공유자원에 접근하지 않고 condition_variable를 통해 wait를 이용해서
> 한번 더 조건을 체크하고 그 뒤에 공유자원을 처리할 수 있도록 해야 함
   

> [!question] lock 안에서 notify_one을 하면 되는거 아님?
```cpp
{
	unique_lock<mutex> lock(m);
	q.push(100);
	cv.notify_one();
}
```
이 경우 어차피 아래에서 lock을 먼저 잡고 그 다음에 wait를 하면서 조건변수로 조건식을 검사하기 때문에
```cpp
	unique_lock<mutex> lock(m);
	cv.wait(lock, []() { return q.empty() == false; });
```
lock을 위에서 잡은 상태에서 notify_one을 해서 thread를 깨워도 꺠운 시점에서는 lock이 없기 때문에
진행을 못하는 똑같은 상황이 나오기 때문에 별 의미가 없다
