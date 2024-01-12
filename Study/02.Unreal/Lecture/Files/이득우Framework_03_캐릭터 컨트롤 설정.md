---
Index: "[[UnrealIndex]]"
tags:
  - work
  - unreal
  - Study
---
   
## 1. 에디터 Play 에서 수치확인
---
```
DisplayAll PlayerController ControlRotation
```
![[Pasted image 20230531144650.png]]
![[Pasted image 20230531144619.png]]
   
   
## 2. Move, Look 함수 분석
---
```cpp
//Contorller의 회전값을 Update
AddControllerYawInput(LookAxisVector.X);
AddControllerPitchInput(LookAxisVector.Y);
```

```cpp
//특정 방향으로 이동시키는 함수
AddMovementInput(ForwardDirection, MovementVector.X);
AddMovementInput(RightDirection, MovementVector.Y);
```
   
   
## 3. Controller 옵션
---
> [!question] 사전준비
> 만들어놓은 CharacterPlayer를 상속시킨 Blueprint에서 옵션을 볼 수 있다

### 3-1. controller의 회전값과 Pawn의 회전값을 동기화
![[Pasted image 20230531145548.png]]
   
### 3-2. controller의 회전값과 spring Arm의 회전값을 동기화
![[Pasted image 20230531145854.png]]
Use Pawn Control Rotation : controller와 회전값 동기화
Inherit ~ : 부모로부터 받은 회전값을 적용
   
![[Pasted image 20230531145903.png]]
카메라의 충돌이 발생했을 때 카메라를 당겨주는 옵션
   
![[Pasted image 20230531163741.png]]
Use Pawn Control Rotation -> controller의 회전값을 사용할 것인지?
   
   
## 4. Data Asset
---
> [!info] Controller나 Movement에 쓰일 여러 데이터 수치들을 모아놓을 수 있다

### 4-1. PrimaryDataAsset 상속 받아 Class 생성
![[Pasted image 20230531174521.png]]
   
### 4-2. 원하는 데이터를 삽입
```cpp
//ABCharacterControlData.h
UCLASS()
class ARENABATTLE_API UABCharacterControlData : public UPrimaryDataAsset
{
	...
	UPROPERTY(EditAnywhere, Category = CharacterMovement)
	uint32 bUseControllerDesiredRotation : 1;
	
	UPROPERTY(EditAnywhere, Category = CharacterMovement)
	FRotator RotationRate;
	
	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = Input)
	TObjectPtr<class UInputMappingContext> InputMappingContext;
	...
};
```
에디터에서 설정할 데이터들을 Header에 정의
   
### 4-3. 만든 Class 상속시켜 Data Asset 생성
Miscellaneous > Data Asset
![[Pasted image 20230531180643.png]]
   
### 4-4. 원하는 데이터 셋팅
![[Pasted image 20230531180929.png]]
   
### 4-5. C++ 셋팅
```cpp
//헤더파일
//Enum셋팅 뒤 Map 으로 저장
UPROPERTY(EditAnywhere, Category = CharacterControl, Meta = (AllowPrivateAccess = "true"))
TMap<ECharacterControlType, class UABCharacterControlData*> CharacterControlManager;

//CDO
static ConstructorHelpers::FObjectFinder<UABCharacterControlData> ShoulderDataRef(TEXT("/Script/ArenaBattle.ABCharacterControlData'/Game/ArenaBattle/CharacterControl/ABC_Shoulder.ABC_Shoulder'"));
if (ShoulderDataRef.Object)
{
	CharacterControlManager.Add(ECharacterControlType::Shoulder, ShoulderDataRef.Object);
}

static ConstructorHelpers::FObjectFinder<UABCharacterControlData> QuaterDataRef(TEXT("/Script/ArenaBattle.ABCharacterControlData'/Game/ArenaBattle/CharacterControl/ABC_Quater.ABC_Quater'"));
if (QuaterDataRef.Object)
{
	CharacterControlManager.Add(ECharacterControlType::Quater, QuaterDataRef.Object);
}
```
   
### 4-6. 사용
```cpp
SetCharacterControl(ECharacterControlType::Shoulder);
```
```cpp
void AABCharacterPlayer::SetCharacterControl(ECharacterControlType NewCharacterControlType)
{
	UABCharacterControlData* NewCharacterControl = CharacterControlManager[NewCharacterControlType];
	check(NewCharacterControl);

	SetCharacterControlData(NewCharacterControl);

	APlayerController* PlayerController = CastChecked<APlayerController>(GetController());

	//Input Mapping Context 변경
	if (UEnhancedInputLocalPlayerSubsystem* Subsystem = 
		ULocalPlayer::GetSubsystem<UEnhancedInputLocalPlayerSubsystem>(PlayerController->GetLocalPlayer()))
	{
		//기존에 존재하던 Mapping Clear
		Subsystem->ClearAllMappings();
		UInputMappingContext* NewMappingContext = NewCharacterControl->InputMappingContext;
		if (NewMappingContext)
		{
			Subsystem->AddMappingContext(NewMappingContext, 0);
		}
	}

	CurrentCharacterControlType = NewCharacterControlType;
}
```

```cpp
void AABCharacterPlayer::SetCharacterControlData(const UABCharacterControlData* CharacterControlData)
{
	// Pawn
	bUseControllerRotationYaw = CharacterControlData->bUseControllerRotationYaw;
	// CharacterMovement
	GetCharacterMovement()->bOrientRotationToMovement = CharacterControlData->bOrientRotationToMovement;
	...
	//Camera
	CameraBoom->TargetArmLength = CharacterControlData->TargetArmLength;
	CameraBoom->SetRelativeRotation(CharacterControlData->RelativeRotation);
	...
}
```
   
### 4-7. 정리
![[KakaoTalk_20230531_202641605.jpg|700]]