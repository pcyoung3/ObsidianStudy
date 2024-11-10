---
Index: "[[4. UnrealIndex]]"
tags:
  - work
  - unreal
  - Study
---
   
## 1. 전체 구조
---
![[KakaoTalk_20230601_154242504.jpg|800]]
1. C++에서 UABAnimInstance를 상속하고 필요한 데이터를 선언 및 정의
2. Animation Blueprint에서 만든 AnimInstance 기반의 C++ 을 상속하고 본격적인 애니메이션을 정의
   
   
## 2. ABP 구조
---
![[KakaoTalk_20230601_155348798.jpg|800]]
1. ABP 내부에서 State machine을 만듦
2. State Machine 내에서 State를 생성하고 애니메이션을 지정
3. 필요에 따라 State Alias를 사용해서 State를 지정(if문이라고 생각하면 편함)
   
   
## 3. AnimInstance C++ 생성
---
![[Pasted image 20230601160617.png|800]]
```cpp
//Header
UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = Character)
TObjectPtr<class ACharacter> Owner;

UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = Character)
TObjectPtr<class UCharacterMovementComponent> Movement;

//Cpp
Owner = Cast<ACharacter>(GetOwningActor());
if (Owner)
{
	Movement = Owner->GetCharacterMovement();
}
```
다음과 같이 Character를 가져오고 반대로 Character에서 Animation을 가져올 수 있다
   
> [!tip] 생성 시 다음 함수를 기억
```cpp
// Animation init 시 호출
virtual void NativeInitializeAnimation() override;

// Animation 매 프레임 호출
virtual void NativeUpdateAnimation(float DeltaSeconds) override;
```
   
   
## 4. ABP 상세설정
---
### 4-1. State Machine
![[Pasted image 20230601161258.png]]
1. 가장 바깥쪽인 AnimGraph에서 생성가능
2. 결과물을 Cache로 만들어 낼 수 있다.
	1. New Save Cache Pose 로 저장
	2. Use cached pose 로 사용 가능
   
### 4-2. Bland Space(1D)
1. Animation > Legacy > Bland Space 1D로 생성
   
![[Pasted image 20230601161921.png|500]]
2. 간선에 몇개의 애니메이션을 추가해서 특정 값에 따라 Blend 가능
   
![[Pasted image 20230601161929.png]]
3. 특정 값은 Horizontal Axis에서 설정 가능하다
   
### 4-3. State
1. State Machine을 만들면 그 안에서 State를 생성할 수 있다.
![[Pasted image 20230601162346.png|600]]
2. State 내부에서는 Blue Print로 작업 가능
![[Pasted image 20230601162427.png|600]]

> [!example] Apply Additive를 사용하면 두 애니메이션을 합칠 수 있음
![[Pasted image 20230601162634.png|600]]
   
### 4-4. State Alias
State Alias를 사용하여 상태를 지정하여 해당 상태일 때 조건을 만족하면 다른 상태로 넘어가게 할 수 있다.
   
1. 노드 생성
![[Pasted image 20230601162905.png|300]]
2. 진입점(Locomotion 상태에서만 들어올 수 있음)
![[Pasted image 20230601162913.png|300]]
3. 이동 조건 처리(bIsJumping 이 True 면 상태 이동)
![[Pasted image 20230601163139.png|300]]
조건을 만족하지 않으면 State가 변경되지 않음


