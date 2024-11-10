---
Index: "[[4. UnrealIndex]]"
tags:
  - work
  - unreal
  - Study
---
   
## 1. Collision Channel 추가
---
### 1-1. Project Settings > Collision > Trace Channels 에서 추가
![[Pasted image 20230602211403.png]]
   
### 1-2. Preset > New 로 새 Preset 추가
![[Pasted image 20230602211527.png]]
원하는 다른 Type과 충돌 채널 설정
   
   
## 2. Anim Notify
---
> [!note] 애니메이션의 특정 타임에 Event를 발생시킬 수 있다

### 2-1. Anim Notify 상속받는 C++ 추가
![[Pasted image 20230602211647.png]]
   
### 2-2. Editor Animation Montage 에서 Notify 추가
![[Pasted image 20230602212005.png|600]]
Notifies > 1번채널에 우클릭 해야 표시됨
   
### 2-3. Notify 함수 추가
```cpp
void UAnimNotify_AttackHitCheck::Notify(USkeletalMeshComponent* MeshComp, UAnimSequenceBase* Animation, const FAnimNotifyEventReference& EventReference)
{
	Super::Notify(MeshComp, Animation, EventReference);

	if (MeshComp)
	{
		IABAnimationAttackInterface* AttackPawn = Cast<IABAnimationAttackInterface>(MeshComp->GetOwner());
		if (AttackPawn)
		{
			AttackPawn->AttackHitCheck();
		}
	}
}
```

## 3. 충돌처리
---
![[Pasted image 20230602212357.png|700]]
![[Pasted image 20230602212529.png|700]]
```cpp
FHitResult OutHitResult;
FCollisionQueryParams Params(SCENE_QUERY_STAT(Attack), false, this);

const float AttackRange = 40.0f;
const float AttackRadius = 50.0f;
const float AttackDamage = 30.0f;
const FVector Start = GetActorLocation() + GetActorForwardVector() * GetCapsuleComponent()->GetScaledCapsuleRadius();
const FVector End = Start + GetActorForwardVector() * AttackRange;

bool HitDetected = GetWorld()->SweepSingleByChannel(OutHitResult, Start, End, FQuat::Identity, CCHANNEL_ABACTION, FCollisionShape::MakeSphere(AttackRadius), Params);
if (HitDetected)
{
	FDamageEvent DamageEvent;
	OutHitResult.GetActor()->TakeDamage(AttackDamage, DamageEvent, GetController(), this);
}
```
   
   
## 4. Debung Capsule
---
> [!note] Debug 용으로 충돌범위를 알고 싶을 때 캡슐형태로 그림 표시

```cpp
#if ENABLE_DRAW_DEBUG
FVector CapsuleOrigin = Start + (End - Start) * 0.5f;
float CapsuleHalfHeight = AttackRange * 0.5f;
FColor DrawColor = HitDetected ? FColor::Green : FColor::Red;

DrawDebugCapsule(GetWorld(), CapsuleOrigin, CapsuleHalfHeight, AttackRadius, FRotationMatrix::MakeFromZ(GetActorForwardVector()).ToQuat(), DrawColor, false, 5.0f);
#endif
```
   
   
## 5. Collision C++ 
---
### 5-1. Profile 설정
```cpp
GetCapsuleComponent()->SetCollisionProfileName(TEXT("ABCapsule"));
...
GetMesh()->SetCollisionProfileName(TEXT("NoCollision"));
```
   
### 5-2. TakeDamage overriding
```cpp
float AABCharacterBase::TakeDamage(float DamageAmount, FDamageEvent const& DamageEvent, AController* EventInstigator, AActor* DamageCauser)
{
	Super::TakeDamage(DamageAmount, DamageEvent, EventInstigator, DamageCauser);

	SetDead();

	return DamageAmount;
}
```
   
### 5-3. 데미지 받을 경우 5초뒤에 사망 설정
```cpp
//움직임 및 충돌 비활성화
void AABCharacterBase::SetDead()
{
	GetCharacterMovement()->SetMovementMode(EMovementMode::MOVE_None);
	PlayDeadAnimation();
	SetActorEnableCollision(false);
}
...
//람다함수를 이용하여 일정시간 뒤 객체삭제
void AABCharacterNonPlayer::SetDead()
{
	Super::SetDead();

	FTimerHandle DeadTimerHandle;
	GetWorld()->GetTimerManager().SetTimer(DeadTimerHandle, FTimerDelegate::CreateLambda(
		[&]()
		{
			Destroy();
		}
	), DeadEventDelayTime, false);
}

//Animation 설정
void AABCharacterBase::PlayDeadAnimation()
{
	UAnimInstance* AnimInstance = GetMesh()->GetAnimInstance();
	AnimInstance->StopAllMontages(0.0f);
	AnimInstance->Montage_Play(DeadMontage, 1.0f);
}
```
