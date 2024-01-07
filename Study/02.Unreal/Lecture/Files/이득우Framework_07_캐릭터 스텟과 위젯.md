# Index : [[UnrealIndex]]
## Tag : #work #unreal
---
   
## 1. 위젯 구조
---
![[Pasted image 20230614200729.png|800]]
![[Pasted image 20230614200754.png|800]]
   
   
## 2. WidgetComponent 생성
---
1) ActorComponent를 상속받아서 클래스 생성
2) Widget Blueprint로 Hp바를 생성
3) UserWidget을 상속받아 또 다른 클래스 생성
4) 방금 만든  3)의 클래스를 Widget Blueprint의 부모로 지정 
	이렇게 하면 c++에서 코드 수정을 할 때 자동으로 Widget Blueprint에서 연동이 된다

UPROPERTY(Trasient ...) : 디스크에서 저장하지 않는 프로퍼티
   
   
## 3. Delegate 사용
---
```cpp
//Header 에서 선언
DECLARE_MULTICAST_DELEGATE(FOnHpZeroDelegate);
DECLARE_MULTICAST_DELEGATE_OneParam(FOnHpChangedDelegate, float /*CurrentHp*/);
...
FOnHpZeroDelegate OnHpZero;
FOnHpChangedDelegate OnHpChanged;
...

//cpp에서 사용
OnHpZero.Broadcast();
OnHpChanged.Broadcast(CurrentHp);
```
   
   
## 4. UserWidget 코드
---
### 4-1. 생성자 구조를 맞춰서 구현
```cpp
UABHpBarWidget(const FObjectInitializer& ObjectInitializer);

UABHpBarWidget::UABHpBarWidget(const FObjectInitializer& ObjectInitializer) : Super(ObjectInitializer)
{
	...
}
```
   
### 4-2. 빌드에 UMG 추가
```C#
PublicDependencyModuleNames.AddRange(new string[] { "Core", "CoreUObject", "Engine", "InputCore", "EnhancedInput", "UMG" });
```
   
### 4-3. UserWidget에 Widget Blueprint 연동
```cpp
virtual void NativeConstruct() override;

UPROPERTY()
	TObjectPtr<class UProgressBar> HpProgressBar;
//Header
//===============================================================
//Cpp
void UABHpBarWidget::NativeConstruct()
{
	Super::NativeConstruct();

	HpProgressBar = Cast<UProgressBar>(GetWidgetFromName(TEXT("PbHpBar")));
	ensure(HpProgressBar);
```
   
   
## 5. Widget Component 를 Character에 추가
---
### 5-1. Widget Component를 Character에서 생성
```cpp
// Stat Section
protected:
	UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = Stat, Meta = (AllowPrivateAccess = "true"))
	TObjectPtr<class UABCharacterStatComponent> Stat;

// UI Widget Section
protected:
	UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = Widget, Meta = (AllowPrivateAccess = "true"))
	TObjectPtr<class UWidgetComponent> HpBar;
```

```cpp
// Stat Component 
Stat = CreateDefaultSubobject<UABCharacterStatComponent>(TEXT("Stat"));

// Widget Component 
HpBar = CreateDefaultSubobject<UABWidgetComponent>(TEXT("Widget"));
HpBar->SetupAttachment(GetMesh());
HpBar->SetRelativeLocation(FVector(0.0f, 0.0f, 180.0f));
static ConstructorHelpers::FClassFinder<UUserWidget> HpBarWidgetRef(TEXT("/Game/ArenaBattle/UI/WBP_HpBar.WBP_HpBar_C"));
if (HpBarWidgetRef.Class)
{
	HpBar->SetWidgetClass(HpBarWidgetRef.Class);
	HpBar->SetWidgetSpace(EWidgetSpace::Screen);
	HpBar->SetDrawSize(FVector2D(150.0f, 15.0f));
	HpBar->SetCollisionEnabled(ECollisionEnabled::NoCollision);
}
```

## 6. 위젯 컴포넌트의 초기화 과정
---
![[Pasted image 20230614204103.png|800]]
![[Pasted image 20230614204512.png]]
캐릭터 정보를 전달하는 과정 : UserWidget(Hp바가 표시되는 실제 위젯) <- WidgetComponent <- Character
   
과정은 이렇게 옮겨야하지만 타이밍에 문제가 있다.
그래서 WidgetComponent::InitWidget() 타이밍 때 Actor 정보를 넘기는 
Set함수를 UserWidget을 상속한 클래스에 구현하고 거기서 Owner 셋팅을 해준다.
   
### 6-1. UUserWidget
```cpp
class ARENABATTLE_API UABUserWidget : public UUserWidget
{
	...
	FORCEINLINE void SetOwningActor(AActor* NewOwner) { OwningActor = NewOwner; }
	...
}
```
   
### 6-2. WidgetComponent
```cpp
class ARENABATTLE_API UABWidgetComponent : public UWidgetComponent
{
	...
protected:
	virtual void InitWidget() override;
}
//=================================================================
void UABWidgetComponent::InitWidget()
{
	Super::InitWidget();

	UABUserWidget* ABUserWidget = Cast<UABUserWidget>(GetWidget());
	if (ABUserWidget)
	{
		ABUserWidget->SetOwningActor(GetOwner());
	}
}
```
   
### 6-3. UUserWidget - NativeConstruct
```cpp
virtual void NativeConstruct() override;
//=================================================================
void UABHpBarWidget::NativeConstruct()
{
	Super::NativeConstruct();

	HpProgressBar = Cast<UProgressBar>(GetWidgetFromName(TEXT("PbHpBar")));
	ensure(HpProgressBar);
	...
}
```
   
### 6-4. ACharacter - PostInitializeComponents
> [!info] PostInitializeComponents 함수
> 해당 함수는 언리얼의 객체가 생성될 때 마지막으로 불리는 함수이다.
> 이 다음은 BeginPlay()가 실행되고 이 때부터 tick()함수가 호출됨
```cpp
virtual void PostInitializeComponents() override;
//=================================================================
void AABCharacterBase::PostInitializeComponents()
{
	Super::PostInitializeComponents();

	Stat->OnHpZero.AddUObject(this, &AABCharacterBase::SetDead);
}
```



![[Pasted image 20230615152224.png]]
![[Pasted image 20230615152057.png]]