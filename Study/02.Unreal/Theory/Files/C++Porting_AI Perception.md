---
Index: "[[UnrealIndex]]"
tags:
  - unreal
  - Study
---
   
## 1. AI Perception?
---
AI가 Player를 인식할 수 있게하는 시스템이다
거리, 및 시야, 데미지를 받았을 때 등의 상황에서 Player를 인식하고 Behavior Tree와 연계할 수 있게 한다
   
   
## 2. Setting C++
---
### 2-1. AAIController 를 상속받은 Controller 생성
```cpp
UCLASS()  
class D1_API ACombatAIController : public AAIController  
{  
    GENERATED_BODY()  
  
public:  
    ACombatAIController();  
protected:  //Inherited Func  
    virtual void OnPossess(APawn* InPawn) override;  
public:    //Delegate Func  
    //UAIPerceptionComponent Delegate
    UFUNCTION(BlueprintCallable)  
    void OnUpdatePerception(const TArray<AActor*>& PerceivedActors);  
  
    //UCombatComponent Delegate  
    void OnCombatToggle(bool IsCombatEnabled);  
private:  
    UPROPERTY()  
    TObjectPtr<class UAISenseConfig_Sight> SightConfig;  
    UPROPERTY()  
    TObjectPtr<class UAISenseConfig_Damage> DamageConfig;  
};
```
여기서 메인은 PerceptionComponent인데 해당 변수는 AAIController에 이미 선언되어 있다.
UAISenseConfig_Sight와 UAISenseConfig_Damage는 AI Perception 에서 사용할 감지센서의 셋팅값
   
### 2-2. PerceptionComponent 셋팅
생성자에서 PerceptionComponent를 셋팅
```cpp
ACombatAIController::ACombatAIController()
{
	//PerceptionComponent 생성
	PerceptionComponent = CreateDefaultSubobject<UAIPerceptionComponent>(TEXT("PerceptionComponent"));
	
	//UAISenseConfig_Sight 설정-> 몬스터의 시야에 따른 인식
	SightConfig = CreateDefaultSubobject<UAISenseConfig_Sight>(TEXT("SightConfig"));
	SightConfig->SightRadius = 3000.f;
	SightConfig->LoseSightRadius = 3500.f;
	SightConfig->PeripheralVisionAngleDegrees = 45.f;
	FAISenseAffiliationFilter AffiliationFilter;
	AffiliationFilter.bDetectEnemies = true;
	AffiliationFilter.bDetectFriendlies = true;
	AffiliationFilter.bDetectNeutrals = true;
	SightConfig->DetectionByAffiliation = AffiliationFilter;
	SightConfig->AutoSuccessRangeFromLastSeenLocation = 1000.f;
	PerceptionComponent->ConfigureSense(*SightConfig); //Component 에 셋팅
	
	//UAISenseConfig_Damage 설정 -> 몬스터가 데미지를 입었을 때 인식
	DamageConfig = CreateDefaultSubobject<UAISenseConfig_Damage>(TEXT("DamageConfig"));
	DamageConfig->SetMaxAge(3.f);	// 인식한 타겟을 3초뒤에 리셋
	PerceptionComponent->ConfigureSense(*DamageConfig);
	
	//Delegate 설정 -> 해당 함수는 AI Update에서 호출됨
	PerceptionComponent->OnPerceptionUpdated.AddDynamic(this, &ACombatAIController::OnUpdatePerception);
}
```
<AIController를 상속받은 Blueprint 객체>
![[Pasted image 20230918102029.png]]
   
### 2-3. Perception Update 함수
Setting 부분에서 Delegate를 걸어준 함수
해당 함수는 주기적으로 실행되는 Delegate 함수
```cpp
void ACombatAIController::OnUpdatePerception(const TArray<AActor*>& PerceivedActors)
{
	for (auto SensoredActor : PerceivedActors)	//탐지된 Actor
	{
		IGameplayTagAssetInterface* CurActor = Cast<IGameplayTagAssetInterface>(SensoredActor);	//Player가 아닌 Actor는 탐지되도 반응 없게 함
		if(!CurActor)
			continue;
		if(!CurActor->HasMatchingGameplayTag(FCombatGameplayTags::Get().Character_Player)) //GameplayTag로 분류
			continue;
		
		FActorPerceptionBlueprintInfo PerceptionInfo;  //현재 감지된 Actor를 감지한 방식
		GetPerceptionComponent()->GetActorsPerception(SensoredActor, PerceptionInfo);

		for(int i = 0; i < PerceptionInfo.LastSensedStimuli.Num(); i++)	//탐지방식에 따른 for문
		{
			auto Info = PerceptionInfo.LastSensedStimuli[i];
			auto SensedClass = UAIPerceptionSystem::GetSenseClassForStimulus(this, Info);
			if(SensedClass == UAISense_Sight::StaticClass())  //시야기반 탐지방식일 경우
			{
				SetTargetActor(SensoredActor);
			}
			else if(SensedClass == UAISense_Damage::StaticClass())  //데미지기반 탐지방식일 경우
			{
				if(Info.WasSuccessfullySensed() && !Info.IsExpired())	//After DamageConfig->GetMaxAge then Expired
					SetTargetActor(SensoredActor);
			}
		}
	}
}
```
다음 생성은 Blueprint에서도 할 수 있다
   
   
## 3. Setting Blueprint
---
사용하려는 BP Monster에서
![[Pasted image 20230918140956.png]]
다음과 같이 Ai Controller Class를 지정해주면 작동한다
   
   
## 참고링크
---
https://forums.unrealengine.com/t/ai-perception-get-sense-class-fails-to-compare-with-its-static-class/333483
https://awayteamkiller.tistory.com/5
