---
Index: "[[5. CSIndex]]"
tags:
  - Study
  - CS
---
   
## 1. Main 함수의 성질
---
프로그램을 실행시키면 그건 프로세스가 되고 Main 함수는 모든 프로그램의 시작점인 함수이다

즉, Main 함수는 Thread 함수
Main Thread 라고 할 수 있다


## 2. Main 의 Return 값
---
Main 이 Thread 함수기 때문에 Thread의 특성상 종료처리가 제대로 되지 않으면 문제가 생길 수 있다.
Thread는 OS의 자원이기 때문에 종료 인자를 넘겨줘서 반환처리를 해야 한다.

Main에서는 이게 return 0 라고 볼 수 있는데
즉, 

### 1번 케이스
```cpp
void main()
{

}
```

### 2번 케이스
```cpp
int main()
{
	return 0;
}
```

이 중에서 2번 케이스가 Thread를 Call 쪽에 종료처리를 더 제대로 한 케이스이다.
return 0는 정상종료라는 뜻을 가지고 있다.
