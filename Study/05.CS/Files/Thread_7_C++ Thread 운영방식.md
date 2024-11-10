---
Index:
  - "[[5. CSIndex]]"
tags:
  - Study
  - CS
---
   
## 1. job-based system
---
> [!INFO] 정의
> Thread 운영방식 중에 Thread Pool을 만들어놓고
> 동기화 하지 않은 상태로 동시에 작업을 모두 수행하는 방식이다
> ![[KakaoTalk_20241026_192852352.jpg]]

같은 작업을 MultiThread 환경에서 동시다발적으로 처리하기 때문에 빠르게 처리할 수 있어서 사용되는 방식이다

대표적인 예시로는 Unreal 에서 Animation을 Task 형식으로 처리하는데 여기서 사용되는 기법이다
   
   
## 2. Producer - Consumer Model (생산 소비 모델)
---
> [!INFO] 정의
> Thread를 지정해서 한쪽 Thread에서는 어떤 데이터를 생산시키고
> 나머지 Thread에서는 데이터를 소비시키는 모델이다
> ![[KakaoTalk_20241026_192929785.jpg]]
> 
> 이 때 데이터를 생산해서 Queue(다른 자료구조도 가능)에 집어넣고 
> 소비하는 쪽에서는 Queue에 있던 데이터들을 소비하는 식으로 구현함

해당 방식으로 Multi Thread의 데이터 전달이 유용해지기 시작

