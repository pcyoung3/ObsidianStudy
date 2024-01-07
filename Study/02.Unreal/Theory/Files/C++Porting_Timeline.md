# Index : [[UnrealIndex]]
## Tag : #work #unreal #Study 
---
   
## 1. Timeline?
---
특정 값을 변화시킬 때 해당 값의 변화 곡선 즉, 기울기를 Editor에서 손쉽게 조정할 수 있게 하는 Unreal 기능
보통 Blueprint에서 많이 구현하지만 이걸 C++로 구현할 수도 있다.
   
   
## 2. Curve 생성
---
![[Pasted image 20231030192520.png]]
Timeline을 사용하기 위해서는 해당 변화값을 가진 Curve가 필요하다
Curve는 Unreal Editor에서 생성 가능하다

* 우클릭 > miscellaneous >Curve 
   
   
## 3. Header 선언
---
필요한건 UTimelineComponent와 Curve를 담을 변수, 그리고 Timeline Speed가 필요하다
```cpp
private:	//Timeline
	UPROPERTY(VisibleAnywhere, BlueprintReadWrite, Category = "Timeline", meta = (AllowPrivateAccess = "true"))
	TObjectPtr<class UTimelineComponent> RotateToTargetTimeLineComponent;
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Timeline", meta = (AllowPrivateAccess = "true"))
	TObjectPtr<UCurveFloat> CurveFloatTimeline;
	const float RotateToTargetInterpSpeed = 9.f;
```
   
   
## 4. Cpp 사용
---
### 4-1. UTimelineComponent 생성 및 Curve 불러오기
```cpp
AMasterAI::AMasterAI()  //생성자에서 수행
{
	...
	
	//Setting Timeline
	RotateToTargetTimeLineComponent = CreateDefaultSubobject<UTimelineComponent>(TEXT("RotateToTargetTimeLineComponent"));
	static ConstructorHelpers::FObjectFinder<UCurveFloat> UCurveObj(TEXT("/Game/CombatSystem/Blueprints/Timeline/RotateToTargetCurve.RotateToTargetCurve"));
	if(UCurveObj.Succeeded())
		CurveFloatTimeline = UCurveObj.Object;
	...
}
```
   
### 4-2. Timeline 셋팅
```cpp
void AMasterAI::BeginPlay()
{
	Super::BeginPlay();
	...
	
	//Setting Timeline -> 생성자에서 수행할 경우 Curve를 가져오지 못함
	FOnTimelineFloat TimelineFloatCallback;
	TimelineFloatCallback.BindUFunction(this, FName("RotateToTargetUpdate")); //Callback에서 불릴 함수 이름을 삽입
	RotateToTargetTimeLineComponent->SetTimelineLength(0.25f);
	RotateToTargetTimeLineComponent->SetLooping(false);
	if(CurveFloatTimeline)
		RotateToTargetTimeLineComponent->AddInterpFloat(CurveFloatTimeline, TimelineFloatCallback);
}
```
   
### 4-3. Timeline 실행과 중지
#### 실행
```cpp
void ACombatPlayerCharacter::RotateToTarget()
{
	RotateToTargetTimeLineComponent->PlayFromStart();
}
```

#### 중지
```cpp
void ACombatPlayerCharacter::StopRotateToTarget()
{
	RotateToTargetTimeLineComponent->Stop();
}
```
   
### 4-4. Timeline 실행함수
```cpp
UFUNCTION()
	void RotateToTargetUpdate(float Value);
```
해당 함수는 Timeline이 실행되는 매 틱마다 호출