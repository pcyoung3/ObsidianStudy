# Index : [[UnrealIndex]]
## Tag : #work #unreal #Study 
---
   
## 1. Actor, Pawn, Character의 차이
---
* Actor : Level에 배치할 수 있는 Unreal Object
* Pawn : Player나 AI가 빙의할 수 있는 Actor, Navigation을 사용가능
* Character : Pawn을 상속하고 Animation을 염두한 SkeletonMesh가 들어가 있음
   
## 2. 모드, 스테이트
---
* Game mode : 플레이어의 수, 플레이어의 스폰위치, 일시정지 처리 등
* Game State : 게임실행 시간, 각 플레이어의 참가 기간, 플레이어들의 현재 상태
   
## 3. Event
---
* 이벤트 함수들 : BeingPlay, Tick, Actor Begin Overlap 
* 커스텀 이벤트 함수를 만들 수 있음
	* 블루프린트에서 이벤트 : Return 값 없는 함수
	* C++에서 이벤트 : Delegate의 한 모드. 외부 클래스에서 추가 호출할 수 없다
   
## 4. Replication
---
* 서버와 클라이언트 사이에 데이터를 전송, 동기화 시켜주는 것 
