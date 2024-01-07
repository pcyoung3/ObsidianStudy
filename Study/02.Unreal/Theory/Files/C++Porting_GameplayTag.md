# Index : [[UnrealIndex]]
## Tag : #work #unreal #Study 
---
   
## 1. Gameplay Tag?
---
계층 구조가 있는 Enum.
Unreal에서 추가된 시스템이다.
![[Pasted image 20230825083227.png]]
Project Settings > Gameplay Tags에서 설정 가능
   
![[Pasted image 20230825083405.png]]
![[Pasted image 20230825083253.png]]
Editor에서 .으로 계층구조를 정해 추가 가능하다
   
   
## 2. Blueprint 사용법
---
![[Pasted image 20230825083618.png]]
GameplayTag 자료형으로 선언
   
![[Pasted image 20230825083652.png]]
여러 Tag를 검사하고 싶은 경우 GameplayTagContainer를 사용
   
   
## 3. C++에서 사용법
---
### 3-1. Struct 추가
Struct 형태로 추가해서 Singleton을 이용해서 여러 부분에서 호출하는 방식으로 사용
```cpp
#include "CoreMinimal.h"
#include "GameplayTagContainer.h"

struct FCombatGameplayTags
{
public:
	static const FCombatGameplayTags& Get() { return GameplayTags; }
	static void InitializeNativeGameplayTags();

public:
	//State
	FGameplayTag Character_State_Attacking;
	FGameplayTag Character_State_Dead;
	...

	//Action
	FGameplayTag Character_Action_Dodge;
	FGameplayTag Character_Action_EnterCombat;
	...

private:
	static FCombatGameplayTags GameplayTags;
};
```

```cpp
#include "Definitions/CombatGameplayTags.h"
#include "GameplayTagsManager.h"

FCombatGameplayTags FCombatGameplayTags::GameplayTags;

void FCombatGameplayTags::InitializeNativeGameplayTags()
{
	if(GameplayTags.Character_State_Attacking.IsValid())	//Already Execute? then Do not Execute
		return;
	
	/**
	 * State
	 */
	GameplayTags.Character_State_Attacking = UGameplayTagsManager::Get().AddNativeGameplayTag(
		FName("Character.State.Attacking"),
		FString("State Attacking")
	);

	GameplayTags.Character_State_Dead = UGameplayTagsManager::Get().AddNativeGameplayTag(
		FName("Character.State.Dead"),
		FString("State Dead")
	);
	
	...
	
		GameplayTags.Character_Action_Attack_SprintAttack = UGameplayTagsManager::Get().AddNativeGameplayTag(
		FName("Character.Action.Attack.SprintAttack"),
		FString("Action Attack SprintAttack")
	);
}
```
AddNativeGameplayTag를 사용해서 Editor에 GameplayTag를 추가한다.
   
### 3-2. Singleton 초기화
이제 위에서 선언한 struct의 초기화 구문을 호출해줘야 하는데 보통 singleton 초기화는 AssetManager에서 이루어진다.
따라서 AssetManager를 상속한 Class를 생성
```cpp
#pragma once

#include "CoreMinimal.h"
#include "Engine/AssetManager.h"
#include "CombatAssetManager.generated.h"

/**
 * 
 */
UCLASS()
class D1_API UCombatAssetManager : public UAssetManager
{
	GENERATED_BODY()
	
public:
	static UCombatAssetManager& Get();

protected:
	virtual void StartInitialLoading() override;
};

```

```cpp
#include "Definitions/CombatAssetManager.h"
#include "Definitions/CombatGameplayTags.h"

UCombatAssetManager& UCombatAssetManager::Get()
{
	check(GEngine);

	UCombatAssetManager* CombatAssetManager = Cast<UCombatAssetManager>(GEngine->AssetManager);
	return *CombatAssetManager;
}

void UCombatAssetManager::StartInitialLoading()
{
	Super::StartInitialLoading();

	//AssetManager에서 Singleton 초기화
	//Editor의 Engine General Settings에서 설정 변경 필요
	FCombatGameplayTags::InitializeNativeGameplayTags();
}
```

![[Pasted image 20230825084932.png|800]]   
Project Settings > General Settings 에서 Asset Manager Class를 설정하는 곳이 있는데
해당 부분에 만든 Class를 설정
   
### 3-3. 사용
```cpp
void UStateManagerComponent::SetCurrentState(FGameplayTag NewState)
{
	if (NewState != CurrentState)
	{
		OnStateEnd.Broadcast(CurrentState);
		CurrentState = NewState;
		OnStateBegin.Broadcast(CurrentState);
	}
}
```
일반 자료형처럼 사용가능
   
```cpp
void UStateManagerComponent::ResetState()
{
	CurrentState = FGameplayTag::EmptyTag;
}
```
아무것도 표시하고 싶지 않다면 EmptyTag 사용
   
```cpp
bool UStateManagerComponent::IsCurrentStateEqualToAny(FGameplayTagContainer StatesToCheck)
{
	return StatesToCheck.HasTagExact(CurrentState);
}
```
여러 GameplayTag를 묶어서 사용하고 싶다면 FGameplayTagContainer를 사용
HasTagExact의 경우 정확하게 GameplayTag가 일치하는 것이 있으면 True 반환
   
```cpp
//Header
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Status")
	TMap<FGameplayTag, float> ActionStatCost;

//Cpp
ABaseWeapon::ABaseWeapon()
{
	...
	if(!FCombatGameplayTags::Get().Character_Action_Attack_LightAttack.IsValid())
		FCombatGameplayTags::InitializeNativeGameplayTags();
	
	ActionStatCost.Add(FCombatGameplayTags::Get().Character_Action_Attack_LightAttack, 10.f);
	ActionStatCost.Add(FCombatGameplayTags::Get().Character_Action_Attack_HeavyAttack, 15.f);
	...
	ActionDamageMultiplier.Add(FCombatGameplayTags::Get().Character_Action_Attack_LightAttack, 1.f);
	ActionDamageMultiplier.Add(FCombatGameplayTags::Get().Character_Action_Attack_HeavyAttack, 1.4f);
	...
}
```
생성자에서 추가할 경우 Editor에서 해당 GameplayTag에 관한걸 조정할 수도 있다.
![[Pasted image 20230825090802.png]]
Editor에서는 다음과 같이 표시
   
```cpp
void ACombatCharacter::CharacterStateBegin(FGameplayTag CharState)
{
	if (FGameplayTag::EmptyTag == CharState)
	{/*None*/}
	else if (FCombatGameplayTags::Get().Character_State_Attacking == CharState)
	{/*None*/}
	else if (FCombatGameplayTags::Get().Character_State_Dodging == CharState)
	{/*None*/}
	else if (FCombatGameplayTags::Get().Character_State_GeneralActionState == CharState)
	{/*None*/}
	else if (FCombatGameplayTags::Get().Character_State_Dead == CharState)
	{
		PerformDeath();
	}
	else if (FCombatGameplayTags::Get().Character_State_Disable == CharState)
	{/*None*/}
}
```
설정한 Singleton을 이용해서 다음과 같이 비교할 수 있다.
   
   
## 참고 - Signleton초기화 시점
---
Singleton 초기화를 할 때 해당 초기화 시점이 GameplayTag를 사용하는 Class의 생성자가 호출되는 시점보다 늦을 때가 있다
이 경우 초기화 구문을 한번만 탈 수 있게 조정할 필요가 있다.

### CF-1. 예외처리를 이용
```cpp
void FCombatGameplayTags::InitializeNativeGameplayTags()
{
	if(GameplayTags.Character_State_Attacking.IsValid())	//Already Execute? then Do not Execute
		return;
	...
}
```
다음과 같이 초기화 문구에 return 구문을 넣어놓는다
```cpp
ABaseWeapon::ABaseWeapon()
{
	...
	if(!FCombatGameplayTags::Get().Character_Action_Attack_LightAttack.IsValid())
		FCombatGameplayTags::InitializeNativeGameplayTags();
	...
}
```
호출하는 생성자에서는 이런 방식으로 부른다
   
### CF-2. Editor에서 설정한 뒤 ini 파일에서 수정
![[Pasted image 20230825090928.png]]
Project Settings에서 하나라도 추가하게 되면 
![[Pasted image 20230825091025.png]]
DefaultGameplayTags.ini 파일이 생성되게 된다.
따라서 Editor에서 미리 생성을 해놓아서 파일로 저장을 시켜놓던가 ini 파일을 수정하는 방식으로도
초기부터 GameplayTag를 사용할 수 있다
   
   
## 참고링크
---
* 구현 참고
https://dev.epicgames.com/community/learning/tutorials/aqrD/unreal-engine-enhanced-input-binding-with-gameplay-tags-c

* FGameplayTag
https://docs.unrealengine.com/4.26/en-US/API/Runtime/GameplayTags/FGameplayTag/

* FGameplayTagContainer
https://docs.unrealengine.com/4.26/en-US/API/Runtime/GameplayTags/FGameplayTagContainer/