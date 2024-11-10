---
Index: "[[4. UnrealIndex]]"
tags:
  - unreal
  - Study
---
   
## 1. 사용이유
---
특정 객체가 FGameplayTagContainer를 가지고 있을 때
IGameplayTagAssetInterface 를 상속시키면 해당 객체가 소유한 GameplayTag에 관해서 여러 방식으로 접근할 수 있다
   
   
## 2. IGameplayTagAssetInterface 사용방식
---
### 2-1. GameplayTag 상속
```cpp
UCLASS()  
class D1_API AMasterAI : public ACharacter, ... , public IGameplayTagAssetInterface  
{  
    GENERATED_BODY()
    ...
}
```
   
### 2-2. GetOwnedGameplayTags 오버라이딩
---
```cpp
protected:
	// Inherited via IGameplayTagAssetInterface  
	// 가지고있는 Contanier를 넘겨줌
	virtual void GetOwnedGameplayTags(FGameplayTagContainer& TagContainer) const override { TagContainer = OwnedGameplayTags; }

private:
	FGameplayTagContainer OwnedGameplayTags; // 이 객체가 소유한 GameplayContainer
```
   
### 2-3. 사용방법
---
```cpp
UFUNCTION(BlueprintCallable, Category=GameplayTags)  
virtual bool HasMatchingGameplayTag(FGameplayTag TagToCheck) const;

UFUNCTION(BlueprintCallable, Category=GameplayTags)  
virtual bool HasAllMatchingGameplayTags(const FGameplayTagContainer& TagContainer) const;

UFUNCTION(BlueprintCallable, Category=GameplayTags)  
virtual bool HasAnyMatchingGameplayTags(const FGameplayTagContainer& TagContainer) const;
```
미리 정의된 함수들을 그대로 사용해서 가져올 수 있다

```cpp
IGameplayTagAssetInterface* CurActor = Cast<IGameplayTagAssetInterface>(SensoredActor); //Player가 아닌 Actor는 탐지되도 반응 없게 함  
if(!CurActor)  
    continue;  
if(!CurActor->HasMatchingGameplayTag(FCombatGameplayTags::Get().Character_Player))  
    continue;
```
사용할 때는 Cast를 이용해서 원하는 함수를 호출하면 됨
