---
Index: "[[UnrealIndex]]"
tags:
  - work
  - unreal
  - Study
---
   
## 1. UI 생성과정
---
![[Pasted image 20230629162430.png]]
![[스크린샷 2023-06-29 162414.png]]
Player화면에 관련된 정보를 Player Controller가 가지고 있다.
그래서 Player의 정보를 띄워주는 UI는 Player controller에서 생성하게 됨
   
   
## 2. HUD 생성
---
### 2-1. WBP 생성 및 설정
![[스크린샷 2023-06-29 163337.png]]
![[스크린샷 2023-06-29 163349.png]]
Widget Blueprint 생성
Content Browser 우클릭 > User Interface > Widget Blueprint > User Widget
   
### 2-2. WBP 설정
![[Pasted image 20230629165252.png]]
Canvas Panel과 미리 만들어놓았던 WBP 추가
   
### 2-3. C++ Class 생성
![[Pasted image 20230629165406.png]]
UserWidget을 상속하는 Class 생성
![[Pasted image 20230629165451.png]]
2-2에서 만든 WBP에 Parent Class로 지정
   
### 2-4. Player Controller에 Widget 추가
헤더에 widget 변수 정의
```cpp
//ABPlayerController
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = HUD)
TSubclassOf<class UABHUDWidget> ABHUDWidgetClass;

UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = HUD)
TObjectPtr<class UABHUDWidget> ABHUDWidget;
```
   
생성자에 widget 정보 로딩
```cpp
static ConstructorHelpers::FClassFinder<UABHUDWidget> ABHUDWidgetRef(TEXT("/Game/ArenaBattle/UI/WBP_ABHUD.WBP_ABHUD_C"));
if (ABHUDWidgetRef.Class)
{
	ABHUDWidgetClass = ABHUDWidgetRef.Class;
}
```
   
BeginPlay에서 Widget 생성
```cpp
void AABPlayerController::BeginPlay()
{
	Super::BeginPlay();

	FInputModeGameOnly GameOnlyInputMode;
	SetInputMode(GameOnlyInputMode);

	ABHUDWidget = CreateWidget<UABHUDWidget>(this, ABHUDWidgetClass);
	if (ABHUDWidget)
	{
		ABHUDWidget->AddToViewport();
	}
}
```
   
   
## 3. HUD에 데이터 연동
---
### 3-1. Component, Actor, Widget 초기화 시점
![[Pasted image 20230629171123.png]]
![[KakaoTalk_20230629_175034853.jpg]]
   
### 3-2. Initialize Component에서 Stat 초기화
1) bWantsInitializeComponent 을 true로 셋팅해야 InitializeComponent 가 활성화 됨 
```cpp
UABCharacterStatComponent::UABCharacterStatComponent()
{
	...
	bWantsInitializeComponent = true;
}
```
2) InitializeComponent에서 스탯 초기화
```cpp
void UABCharacterStatComponent::InitializeComponent()
{
	Super::InitializeComponent();

	SetLevelStat(CurrentLevel);
	SetHp(BaseStat.MaxHp);
}
```
   
### 3-3. Widget NativeConstruct 설정
1) NativeConstruct 및 맴버변수로 Widget 설정
```cpp
protected:
	virtual void NativeConstruct() override;

protected:
	UPROPERTY()
	TObjectPtr<class UABHpBarWidget> HpBar;

	UPROPERTY()
	TObjectPtr<class UABCharacterStatWidget> CharacterStat;
```
2) NativeConstruct에서 Widget 셋팅
![[Pasted image 20230629173802.png]]
```cpp
void UABHUDWidget::NativeConstruct()
{
	Super::NativeConstruct();

	// GetWidgetFromName : Hierachy에 정의한 Widget Blueprint를 가져옴
	HpBar = Cast<UABHpBarWidget>(GetWidgetFromName(TEXT("WidgetHpBar")));
	ensure(HpBar);

	CharacterStat = Cast<UABCharacterStatWidget>(GetWidgetFromName(TEXT("WidgetCharacterStat")));
	ensure(CharacterStat);

	//Custom Interface에 정의한 함수사용
	IABCharacterHUDInterface* HUDPawn = Cast<IABCharacterHUDInterface>(GetOwningPlayerPawn());
	if (HUDPawn)
	{
		//현재 Widget을 Player에 넘겨줌
		HUDPawn->SetupHUDWidget(this);
	}
}
```
   
### 3-4. Widget과 Player의 데이터 연동
1) Widget에 Update 함수 추가
```cpp
void UpdateStat(const FABCharacterStat& BaseStat, const FABCharacterStat& ModifierStat);
void UpdateHpBar(float NewCurrentHp);
```

```cpp
void UABHUDWidget::UpdateStat(const FABCharacterStat& BaseStat, const FABCharacterStat& ModifierStat)
{
	FABCharacterStat TotalStat = BaseStat + ModifierStat;
	HpBar->SetMaxHp(TotalStat.MaxHp);

	CharacterStat->UpdateStat(BaseStat, ModifierStat);
}

void UABHUDWidget::UpdateHpBar(float NewCurrentHp)
{
	HpBar->UpdateHpBar(NewCurrentHp);
}
```

2) Player의 Stat Component에 있는 정보를 Widget에 넘겨주고 Delegate도 정의
```cpp
DECLARE_MULTICAST_DELEGATE_OneParam(FOnHpChangedDelegate, float /*CurrentHp*/);
DECLARE_MULTICAST_DELEGATE_TwoParams(FOnStatChangedDelegate, const FABCharacterStat& /*BaseStat*/, const FABCharacterStat& /*ModifierStat*/);

...

FOnHpChangedDelegate OnHpChanged;
FOnStatChangedDelegate OnStatChanged;
```

```cpp
void AABCharacterPlayer::SetupHUDWidget(UABHUDWidget* InHUDWidget)
{
	if (InHUDWidget)
	{
		//가장 처음 초기화
		InHUDWidget->UpdateStat(Stat->GetBaseStat(), Stat->GetModifierStat());
		InHUDWidget->UpdateHpBar(Stat->GetCurrentHp());

		//Delegete binding -> Update 등록
		Stat->OnStatChanged.AddUObject(InHUDWidget, &UABHUDWidget::UpdateStat);
		Stat->OnHpChanged.AddUObject(InHUDWidget, &UABHUDWidget::UpdateHpBar);
	}
}
```
3) 넘겨준 Delegate에 Stat 값이 변할때마다 Update할 수 있게 Broadcasting 정의
```cpp
FORCEINLINE void SetBaseStat(const FABCharacterStat& InBaseStat) { BaseStat = InBaseStat; OnStatChanged.Broadcast(GetBaseStat(), GetModifierStat()); }

FORCEINLINE void SetModifierStat(const FABCharacterStat& InModifierStat) { ModifierStat = InModifierStat; OnStatChanged.Broadcast(GetBaseStat(), GetModifierStat()); }
```
Stat 값이 변할 때는 이제 항상 Set함수를 써서 값을 변경시키고 Set 함수에서는 Broadcasting으로 Widget에 정보전달
   
   
## 4. Reflection과 Iterator를 이용한 구조체 값 설정
---
1) Widget의 TextBlock을 저장할 Container 선언
```cpp
UPROPERTY()
TMap<FName, class UTextBlock*> BaseLookup;

UPROPERTY()
TMap<FName, class UTextBlock*> ModifierLookup;
```
2) TMap에 UTextBlock 저장
```cpp
void UABCharacterStatWidget::NativeConstruct()
{
	Super::NativeConstruct();

	//Iterator 사용 -> TFieldIterator<FNumericProperty> PropIt
	//struct의 요소들의 정보를 Reflection을 이용해 가져옴 -> FABCharacterStat::StaticStruct()
	for (TFieldIterator<FNumericProperty> PropIt(FABCharacterStat::StaticStruct()); PropIt; ++PropIt)
	{
		//속성이름 GetName()
		const FName PropKey(PropIt->GetName());
		//WBP에 설정한 이름
		const FName TextBaseControlName = *FString::Printf(TEXT("Txt%sBase"), *PropIt->GetName());
		const FName TextModifierControlName = *FString::Printf(TEXT("Txt%sModifier"), *PropIt->GetName());

		//WBP에 설정한 Text Block 가져옴
		UTextBlock* BaseTextBlock = Cast<UTextBlock>(GetWidgetFromName(TextBaseControlName));
		if (BaseTextBlock)
		{
			BaseLookup.Add(PropKey, BaseTextBlock);
		}

		UTextBlock* ModifierTextBlock = Cast<UTextBlock>(GetWidgetFromName(TextModifierControlName));
		if (ModifierTextBlock)
		{
			ModifierLookup.Add(PropKey, ModifierTextBlock);
		}
	}
}
```
3) Widget에 Stat 정보 셋팅
```cpp
void UABCharacterStatWidget::UpdateStat(const FABCharacterStat& BaseStat, const FABCharacterStat& ModifierStat)
{
	for (TFieldIterator<FNumericProperty> PropIt(FABCharacterStat::StaticStruct()); PropIt; ++PropIt)
	{
		const FName PropKey(PropIt->GetName());

		float BaseData = 0.0f;
		//GetValue_InContainer
		//구조체의 포인터를 넘겨주면 현재 반복자인 FNumericProperty 속성의 값을 반환
		PropIt->GetValue_InContainer((const void*)&BaseStat, &BaseData);
		float ModifierData = 0.0f;
		PropIt->GetValue_InContainer((const void*)&ModifierStat, &ModifierData);

		//TMap.Find -> 반환값으로 *형식 뱉음
		//TMap<FName, class UTextBlock*> BaseLookup 이므로
		//더블 포인터로 받아야 함
		UTextBlock** BaseTextBlockPtr = BaseLookup.Find(PropKey);
		if (BaseTextBlockPtr)
		{
			//SetText : Widget에 Text 셋팅
			//FromString : FString > FText
			//SanitizeFloat : float > FString
			(*BaseTextBlockPtr)->SetText(FText::FromString(FString::SanitizeFloat(BaseData)));
		}

		UTextBlock** ModifierTextBlockPtr = ModifierLookup.Find(PropKey);
		if (ModifierTextBlockPtr)
		{
			(*ModifierTextBlockPtr)->SetText(FText::FromString(FString::SanitizeFloat(ModifierData)));
		}
	}
}
```
