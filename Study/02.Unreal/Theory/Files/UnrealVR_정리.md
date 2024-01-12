---
Index: "[[UnrealIndex]]"
tags:
  - unreal
  - Study
---
   
## 1. MotionContoller, Camera
---
* Pawn에 MotionController를 추가하고 MotionSource를 설정하면 손 위치에 따라 Tracking이 됨
* Camera도 VR Preview 모드로 Play를 할 경우 자동 Tracking 됨
* Camera의 눈높이가 중요하기 때문에 눈높이를 보정하는 작업이 필요
   
## 2. Turn
---
* Pawn에 종속된 카메라가 돌 때 공전이 아닌 자전으로 돌게 하기 위해서 Pawn의 위치값을 보정
   
## 3. GrapComponent
---
* 참고 : [[UnrealVR_Grab Component]]
* 물체의 특성에 맞게 Grab 특성을 3개로 나눔
* Free, Obj to Hand, Hand to Obj
* 물체와 Grab은 충돌시스템을 이용해 처리
   
## 4. 총 시스템
---
* 약실 잡아당기는 모션은 벡터의 투영을 통해서 구현
* 탄창을 넣는 모션은 타임라인과 ABP를 통해서 구현
* 탄창 빼는 모션은 Physics를 이용해서 구현
* 총알 및 데미지 같은 데이터 구현
* 충돌은 레이캐스팅
   
## 5. 몬스터
---
* Behavior tree를 이용해서 생성
* Pawn이므로 Navigation Mesh 사용
   
## 6. UI
---
* VR이기 때문에 직교투영을 한 일반적인 UI가 아니라 Widget을 Scene Component에 Attach 함
* Binding을 통해서 데이터를 넘김
   
## 7. 그 외
---
* 진동 햅틱 구현
* 이펙트 구현





