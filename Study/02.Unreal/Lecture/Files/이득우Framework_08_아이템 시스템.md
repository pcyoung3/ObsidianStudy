# Index : [[UnrealIndex]]
## Tag : #work #unreal
---
   
## 1. ItemBox 충돌 설정
---
### 1-1. 충돌전 선언
```cpp
...

Trigger->SetCollisionProfileName(CPROFILE_ABTRIGGER);//collision profile 설정
Trigger->SetBoxExtent(FVector(40.0f, 42.0f, 30.0f));//BoxComponent size 지정

//OnComponentBeginOverlap은 dynamic으로 되어있어서 AddDynamic을 사용해야함
//또한 OnOverlapBegin도 UFUNCTION 선언 필요
Trigger->OnComponentBeginOverlap.AddDynamic(this, &AABItemBox::OnOverlapBegin);

...

OnOverlapBegin(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepHitResult)
```
   
### 1-2. 충돌발생 시 처리
```cpp
void AABItemBox::OnOverlapBegin(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepHitResult)
{
	if (nullptr == Item)
	{
		Destroy();
		return;
	}

	IABCharacterItemInterface* OverlappingPawn = Cast<IABCharacterItemInterface>(OtherActor);
	if (OverlappingPawn)
	{
		OverlappingPawn->TakeItem(Item);
	}

	Effect->Activate(true); //이팩트 재생
	Mesh->SetHiddenInGame(true); //메쉬 숨김
	SetActorEnableCollision(false); //Actor 콜리전 없앰
	Effect->OnSystemFinished.AddDynamic(this, &AABItemBox::OnEffectFinished); //이팩트가 종료가 되면 처리
}
...
void AABItemBox::OnEffectFinished(UParticleSystemComponent* ParticleSystem)
{
	Destroy();
}
```
   
   
## 2. 아이템 타입
---
### 2-1. Blueprint에서도 사용할 수 있게 처리
```cpp
UENUM(BlueprintType)
```
열거형의 선언지시자에 BlueprintType 키워드를 추가하면 Blueprint에서도 사용가능
   
### 2-2. DataAsset 활용
```cpp
class ARENABATTLE_API UABItemData : public UPrimaryDataAsset
...
class ARENABATTLE_API UABWeaponItemData : public UABItemData
```
이 후 상속시킨 Data Asset을 Blueprint로 만들면 됨
Data Asset 사용법은 [[이득우Framework_03_캐릭터 컨트롤 설정#4. Data Asset]] 참고

그 후 아이템 박스와 충돌 처리는 Delegate로 처리
> [!quote] 새 로그 타입 만드는 법
```cpp
DECLARE_LOG_CATEGORY_EXTERN(LogABCharacter, Log, All); //Header에 선언
DEFINE_LOG_CATEGORY(LogABCharacter); //cpp에 선언
UE_LOG(LogABCharacter, Log, TEXT("Read Scroll")); // 사용
```
   
   
## 3. 캐릭터에 무기 부착
---
```cpp
UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = Equipment, Meta = (AllowPrivateAccess = "true"))
TObjectPtr<class USkeletalMeshComponent> Weapon;
//--------------------------------------------------------------------------------
Weapon = CreateDefaultSubobject<USkeletalMeshComponent>(TEXT("Weapon"));
Weapon->SetupAttachment(GetMesh(), TEXT("hand_rSocket"));
```
SetupAttachment를 수행할 때 전달되는 Text는 소켓이름이다.
   
   
## 4. SoftReferencing
---
* 액터에서 TObjectPtr로 선언한 오브젝트들은 게임 시작 시에 전부 로딩됨
* 게임 처음에 시작할 때 모든 오브젝트들을 다 로딩하면 게임 진행할 때 많은 부담이 됨
* 필요한 데이터만 로딩하도록 TSoftObjectPtr를 사용
* TSoftObjectPtr 는 로딩하지 않고 에셋 주소 문자열만 저장하고 필요시에 로딩시킬 수 있음
   
#### 선언
```cpp
UPROPERTY(EditAnywhere, Category = Weapon)
TSoftObjectPtr<USkeletalMesh> WeaponMesh;
```
   
#### 사용
```cpp
if (WeaponItemData->WeaponMesh.IsPending()) //현재 로딩되어있는지 확인
{
	WeaponItemData->WeaponMesh.LoadSynchronous(); //로딩
}
Weapon->SetSkeletalMesh(WeaponItemData->WeaponMesh.Get()); //.Get함수를 이용해서 가져올 수 있다
```
   
> [!note] 게임 중에 로딩된 메쉬를 띄우는 방법
> 콘솔창을 띄운 뒤
> Obj List Class=SkeletalMesh

