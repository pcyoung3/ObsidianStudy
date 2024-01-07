# Index : [[UnrealIndex]]
## Tag : #work #unreal #Study 
---
   
## 1. Lambda 활용
---
```cpp
FTimerHandle deathTimer;
GetWorld()->GetTimerManager().SetTimer(deathTimer, FTimerDelegate::CreateLambda([&]()
	{
		if (IsValid(CombatComponent->GetMainWeapon()))
			CombatComponent->GetMainWeapon()->Destroy();
		this->Destroy();
	}), 4.f, false); // 4초 후에 object 삭제
```
World의 TimerManager를 이용해서 SetTimer를 호출할 수 있다.
Callback을 등록할 수 있는데 맴버함수의 주소를 넘겨줘도 되고 FTimerDelegate::CreateLambda를 이용해도 됨

#### SetTimer 인자값
1. FTimerHandle
2. 함수주소
3. 몇 초뒤에 Timer 실행할 것인지
4. 반복할 것인지
   
   
## 2. 맴버함수 활용
---
```cpp
UWorld* World = GEngine->GetWorldFromContextObjectChecked(this);
if (World)
	World->GetTimerManager().SetTimer(StaminaTimerHandle, this, &ACombatCharacter::SprintStaminaCost, 0.1f, true);
```
   
   
## 3. Timer를 여러개 사용
---
> [!question] 1.5초 뒤에 특정 함수를 0.1초마다 실행시키는 상황
### Header
```cpp
...
private:
	FTimerHandle AfterRegenTimerHandle;
	FTimerHandle RegenTimerHandle;
...
```
Timer를 셋팅 및 Clear를 하고 중간에 갱신될 수도 있으므로 맴버변수로 선언 
### Cpp
```cpp
...
if (GEngine)
{
	UWorld* World = GEngine->GetWorldFromContextObjectChecked(this);
	if (World)
	{
		World->GetTimerManager().ClearTimer(RegenTimerHandle);
		World->GetTimerManager().SetTimer(AfterRegenTimerHandle, this, &UStatsComponent::AfterDelayExecuteRegenStamina, 1.5f, false);
	}
}
...
```
기존에 진행되고 있던 RegenTimerHandle 를 중지시킨 뒤 AfterRegenTimerHandle의 타이머를 셋팅

```cpp
void UStatsComponent::AfterDelayExecuteRegenStamina()
{
	if (GEngine)
	{
		UWorld* World = GEngine->GetWorldFromContextObjectChecked(this);
		if (World)
		{
			World->GetTimerManager().ClearTimer(RegenTimerHandle);
			World->GetTimerManager().SetTimer(RegenTimerHandle, this, &UStatsComponent::RegenerateStamina, 0.1f, true);
		}
	}
}
```
1.5초 뒤 RegenerateStamina를 0.1초마다 반복 시킴

```cpp
void UStatsComponent::RegenerateStamina()
{
	float curStamina = StaminaRegenRate + GetCurrentStatValue(EStats::Stamina);
	curStamina = FMath::Clamp(curStamina, 0.f, GetMaxStatValue(EStats::Stamina));
	SetCurrentStatValue(EStats::Stamina, curStamina);
	
	if (GetCurrentStatValue(EStats::Stamina) >= GetMaxStatValue(EStats::Stamina))
	{
		UWorld* World = GEngine->GetWorldFromContextObjectChecked(this);
		if (World)
			World->GetTimerManager().ClearTimer(RegenTimerHandle);
	}
}
```
특정 조건이 만족되면 루프되고 있는 타이머를 종료

