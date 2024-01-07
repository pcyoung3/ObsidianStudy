# Index : [[UnrealIndex]]
## Tag : #work #unreal
---
   
## 1. 애니메이션 몽타주란?
---
![[Pasted image 20230601173048.png|800]]
   
   
## 2. 애니메이션 몽타주 생성
---
1. Asset browser 에 있는 Animation Asset을 몽타주 그래프에 드래그
![[Pasted image 20230601174839.png|700]]
2. 그래프 우클릭 후 Section 생성
![[Pasted image 20230601175025.png]]
3. Section 의 이름, 링크 등을 셋팅
![[Pasted image 20230601175111.png]]
> [!tip] 링크
> Section의 Link가 처음에는 다 이어져있는데 이러면 애니메이션이 자동으로 전부 재생된다
> 끊으면 해당 구간만 반복하게 됨
   
4. ABP에 최종 결과물에 몽타주가 나올 수 있게 추가
![[Pasted image 20230601175613.png|500]]
   
   
## 3. C++ Animation Montage
---
### 3-1. Header 선언
```cpp
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = Animation)
TObjectPtr<class UAnimMontage> ComboActionMontage;
```
   
### 3-2. Play
```cpp
// Animation Setting
const float AttackSpeedRate = 1.0f;
UAnimInstance* AnimInstance = GetMesh()->GetAnimInstance();

//Montage_Play(UAnimMontage* 몽타주, 속도)
AnimInstance->Montage_Play(ComboActionMontage, AttackSpeedRate);
```
   
### 3-3. Section 이동
```cpp
UAnimInstance* AnimInstance = GetMesh()->GetAnimInstance();

CurrentCombo = FMath::Clamp(CurrentCombo + 1, 1, ComboActionData->MaxComboCount);
FName NextSection = *FString::Printf(TEXT("%s%d"), *ComboActionData->MontageSectionNamePrefix, CurrentCombo);

//montage section 이동 함수
//Montage_JumpToSection(FName 섹션이름, UAnimMontage* 몽타주)
AnimInstance->Montage_JumpToSection(NextSection, ComboActionMontage);
```
   
### 3-4. 종료 시 Delegate
```cpp
FOnMontageEnded EndDelegate;

//AABCharacterBase::ComboActionEnd 함수 binding
EndDelegate.BindUObject(this, &AABCharacterBase::ComboActionEnd);

//Montage_SetEndDelegate(Delegate 변수, Montage 변수)
AnimInstance->Montage_SetEndDelegate(EndDelegate, ComboActionMontage);
```
   
   
## 4. C++ Timer 
---
> [!info] 특정 시간마다 함수를 실행

### 4-1. 선언
```cpp
FTimerHandle ComboTimerHandle;
```
   
### 4-2. 타이머 셋팅
```cpp
//SetTimer
//1.TimerHandle, 2.Timer 함수 호출하는 오브젝트, 3.타이머가 발동될 때의 함수
//4.타이머가 호출될 시간, 5.반복여부
GetWorld()->GetTimerManager().SetTimer(ComboTimerHandle, this, &AABCharacterBase::ComboCheck, ComboEffectiveTime, false);
```
   
### 4-3. 타이머 비활성화
```cpp
ComboTimerHandle.Invalidate()
```