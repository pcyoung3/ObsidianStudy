---
Index: "[[4. UnrealIndex]]"
tags:
  - unreal
  - Study
---
   
## 1. 충돌시스템
---
어떤 방식으로 충돌했을 때 데미지를 입히는 경우 언리얼의 Damage 시스템을 이용하면
Damage를 쉽게 주고 받을 수 있다

**\[Whole Task]**
![[KakaoTalk_20230826_211002454.jpg|500]]
   
   
## 2. 충돌함수로 충돌검사
---
```cpp
void UCollisionComponent::CollisionTrace()  
{  
    FVector StartLocation, EndLocation;  
    StartLocation = CollisionMeshComponent->GetSocketLocation(StartSocketName);  
    EndLocation = CollisionMeshComponent->GetSocketLocation(EndSocketName);  
    
    TArray<FHitResult> OutHits;  
    UKismetSystemLibrary::SphereTraceMultiForObjects(GetWorld(), StartLocation, EndLocation, TraceRadius, CollisionObjectTypes, false, ActorsToIgnore, DrawDebugType, OutHits, true);  
  
    for (auto hitResult : OutHits)  
    {       LastHit = hitResult;  
       if (!AlreadyHitActors.Contains(LastHit.GetActor()))  
       {          HitActor = LastHit.GetActor();  
          AlreadyHitActors.Add(HitActor);  
          OnHitDelegate.ExecuteIfBound(LastHit); //Multi면 변경해야됨  
       }  
    }}
```
UKismetSystemLibrary::SphereTraceMultiForObjects 함수를 사용
> [!question] SphereTraceMultiForObjects
> 원 범위로 여러 개의 Object의 충돌검사를 하는 함수
   
또한 들어가는 인자값이 많기 때문에 참고
```cpp
	//들어가는 각 인자값 참고
	float TraceRadius;

	//충돌할 그룹들을 결정. 충돌 그룹은 Editor에서 설정가능
	TArray<TEnumAsByte<EObjectTypeQuery>> CollisionObjectTypes;
	CollisionObjectTypes.Add(UEngineTypes::ConvertToObjectType(ECollisionChannel::ECC_Pawn));

	//충돌을 무시할 Actor들의 배열
	TArray<TObjectPtr<AActor>> ActorsToIgnore;
	ActorsToIgnore.Add(GetOwner());

	//충돌구체의 범위 랜더링 방식 결정(Debug 전용)
	TEnumAsByte<EDrawDebugTrace::Type> DrawDebugType;
	DrawDebugType = EDrawDebugTrace::None;
```
우선 데미지를 입히기 위해서 충돌 부분을 구현하는데
충돌이 성공했다면 Delegate를 호출하고 동시에 Hit 정보를 넘겨줌
   
   
## 3. 충돌 시 호출되는 Delegate 함수
---
**\[Delegate Bind\]**
```cpp
ABaseWeapon::ABaseWeapon()  
{  
    CollisionComponent = CreateDefaultSubobject<UCollisionComponent>(TEXT("CollisionComponent"));  
    CollisionComponent->OnHitDelegate.BindUObject(this, &ABaseWeapon::OnHit);
    ...
}
```
   
**\[Delegate 함수]**
```cpp
void ABaseWeapon::OnHit(FHitResult hitResult)  
{  
    ICombatInterface* pActor = Cast<ICombatInterface>(hitResult.GetActor());  
    if (pActor)  
    {       if (pActor->Execute_CanReceiveDamage(hitResult.GetActor()))  
          UGameplayStatics::ApplyPointDamage(hitResult.GetActor(), GetDamage(), GetOwner()->GetActorForwardVector(), hitResult, GetInstigatorController(), this, DamageTypeClass);  
    }}
```
UGameplayStatics::ApplyPointDamage 함수를 사용
> [!question] ApplyPointDamage
> Damage를 적용하는데 충돌의 위치를 같이 넘겨줌

해당 함수는 Damage를 입히는 공격자가 호출하는 함수이고 
해당 함수에서 값이 제대로 들어왔다면 피격자는 TakeDamage 함수를 호출

인자 중에서 DamageTypeClass가 있는데 DamageType을 넘겨줘서 Damage를 받는 입장에서
어떤 방식으로 처리할 지 커스터마이징이 가능

해당 부분은 다음과 같이 정의
![[Pasted image 20230826204808.png]]
```cpp
//1. DamageType이 없다면 그냥 nullptr 로 줘도 됨
//또는 그냥 이렇게 빈 class 자체를 넘겨줘도 됨
TSubclassOf<UDamageType> DamageType;

//////////////////////////////////////////////////////////////////

//2. DamageType을 만들 경우 - Blueprint에서 만들어서 넘기는 방법
//UDamageType을 상속하는 객체를 직접 생성
UCLASS()  
class D1_API UAttackDamageType : public UDamageType  
{  
    GENERATED_BODY()  
  
public:  
    EDamageType DamageType = EDamageType::MeleeDamage; //DamageType을 나타내기 위한 Enum
};
...

//Blueprint 상에서 UDamageType를 상속한 객체를 지정한 뒤
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Initialization")
TSubclassOf<UDamageType> DamageTypeClass;
...

//해당 객체의 StaticClass를 가져오는 식으로 사용가능
DamageTypeClass = UAttackDamageType::StaticClass();
```
   
   
## 4. TakeDamage
---
> [!danger] (주의!!) ApplyDamage에서 Damage가 0으로 들어오면 TakeDamage 자체가 호출되지 않음
```cpp
float ACombatCharacter::TakeDamage(float Damage, FDamageEvent const& DamageEvent, AController* EventInstigator, AActor* DamageCauser)
{
	float fDamage = Super::TakeDamage(Damage, DamageEvent, EventInstigator, DamageCauser);

	//UDamageType 의 경우 FDamageEvent에 Class정보가 담겨져서 전달
	UAttackDamageType* damageTypeClass = Cast<UAttackDamageType>(DamageEvent.DamageTypeClass->GetDefaultObject());
	if(!IsValid(damageTypeClass))
		return false;

	//FDamageEvent을 FPointDamageEvent로 변환해서 위치 정보를 얻어냄
	if (DamageEvent.IsOfType(FPointDamageEvent::ClassID))
	{
		const FPointDamageEvent* PointDamageEvent = static_cast<const FPointDamageEvent*>(&DamageEvent);

		//스텟 관련 처리
		StatsComponent->TakeDamageOnStat(Damage);

		//앞에서 맞았는지 뒤에서 맞았는지 판별
		bHitFront = UKismetMathLibrary::InRange_FloatFloat(this->GetDotProductTo(EventInstigator->GetPawn()), -0.1f, 1.f);
		LastHitInfo = PointDamageEvent->HitInfo;

		//play sound, effect 
		ApplyImpactEffect(damageTypeClass->DamageType);
		
		if (CanReceiveHitReaction())
			ApplyHitReaction(damageTypeClass->DamageType);
	}
	return fDamage;
}
```