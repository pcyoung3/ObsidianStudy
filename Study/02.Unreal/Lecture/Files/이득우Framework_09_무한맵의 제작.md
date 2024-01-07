# Index : [[UnrealIndex]]
## Tag : #work #unreal
---
   
## 1. OnConstruction
---
에디터에서 어떤 셋팅값을 바꿨을 때 호출되는 함수이다.
![[Pasted image 20230616132128.png]]
여기서는 CUrrent State를 바꾸면 호출됨
   
```cpp
virtual void OnConstruction(const FTransform& Transform) override;
//-------------------------------------------------------------------
void AABStageGimmick::OnConstruction(const FTransform& Transform)
{
	Super::OnConstruction(Transform);

	SetState(CurrentState);
}
```
디버그 할 때 많이 사용됨
   
   
## 2. TSubclassOf
---
지정한 클래스로부터 상속받은 클래스 목록만 한정적으로 보여주는 템플릿
```cpp
UPROPERTY(EditAnywhere, Category = Fight, Meta = (AllowPrivateAccess = "true"))
TSubclassOf<class AABCharacterNonPlayer> OpponentClass;
```
이 예시에서는 게임 제작이 계속될 수록 AABCharacterNonPlayer의 자식클래스가 점점 더 많아질텐데
이럴 때 모든 액터의 목록을 전부 검색하면 사용하려는 것보다 많은 선택창이 표시된다.
이러한 동작을 방지해주는 템플릿이라고 생각하면 됨
   
   
## 3. SpawnActor
---
Actor를 Spawn해주는 함수
```cpp
//spawn을 위해서 CDO에서 다른 클래스의 정보를 지
OpponentClass = AABCharacterNonPlayer::StaticClass();

...

const FVector SpawnLocation = GetActorLocation() + FVector::UpVector * 88.0f;

//클래스, 위치, 회전값을 지정
AActor* OpponentActor = GetWorld()->SpawnActor(OpponentClass, &SpawnLocation, &FRotator::ZeroRotator);

AABCharacterNonPlayer* ABOpponentCharacter = Cast<AABCharacterNonPlayer>(OpponentActor);
if (ABOpponentCharacter)
{
	ABOpponentCharacter->OnDestroyed.AddDynamic(this, &AABStageGimmick::OnOpponentDestroyed);
}
```
   
   
## 4. 캐릭터가 overlap 되었을 때 처리
---
### 4-1. Tag 처리
```cpp
//Stage는 맴버변수
TObjectPtr<class UStaticMeshComponent> Stage;

//Tag 생성
UBoxComponent* GateTrigger = CreateDefaultSubobject<UBoxComponent>(TriggerName);
GateTrigger->ComponentTags.Add(GateSocket);

//Tag 검사
check(OverlappedComponent->ComponentTags.Num() == 1); //ComponentTags.Num() : Tag 개수
FName ComponentTag = OverlappedComponent->ComponentTags[0]; //첫번째 Tag를 가져옴
FName SocketName = FName(*ComponentTag.ToString().Left(2)); //Tag의 앞 2글자를 가져옴
check(Stage->DoesSocketExist(SocketName)); //소켓이 있는지 검사
```
CDO에서 지정한 Tag들을 가져와서 미리 지정한 Socket 이름과 같은지 검사하는 부분이다.
> [!INFO]- Socket 이름 참고
> ![[Pasted image 20230616135227.png]]
   
### 4-2. 트리거에 닿았을 때 다음 블록 생성
```cpp
FVector NewLocation = Stage->GetSocketLocation(SocketName); //소켓 이름으로 위치 반환
TArray<FOverlapResult> OverlapResults;

//자기자신을 제외한 것을 설정
FCollisionQueryParams CollisionQueryParam(SCENE_QUERY_STAT(GateTrigger), false, this);

//해당 위치에 어떤 물체가 있는지 검사하는 함수
bool bResult = GetWorld()->OverlapMultiByObjectType(
	OverlapResults,
	NewLocation,
	FQuat::Identity,
	FCollisionObjectQueryParams::InitType::AllStaticObjects, //AllStaticObjects 모든 스테틱 오브젝트 타입 중
	FCollisionShape::MakeSphere(775.0f), //구체 생성
	CollisionQueryParam //자신의 충돌 제외 
);

//해당 공간에 아무것도 없으면 생성
if (!bResult)
{
	GetWorld()->SpawnActor<AABStageGimmick>(NewLocation, FRotator::ZeroRotator);
}
```
   
   
## 5. 아이템박스생성
---
### 5-1. TWeakObjectPtr(약참조)
```cpp
//TSubclassOf로 AABItemBox를 상속받은 클래스로 한정
UPROPERTY(VisibleAnywhere, Category = Reward, Meta = (AllowPrivateAccess = "true"))
TSubclassOf<class AABItemBox> RewardBoxClass;

//TWeakObjectPtr : 약참조
UPROPERTY(VisibleAnywhere, Category = Reward, Meta = (AllowPrivateAccess = "true"))
TArray<TWeakObjectPtr<class AABItemBox>> RewardBoxes;
```
* TObjectPtr : 강참조, 이걸 선언한 class가 삭제되면 해당 object의 reference count를 감소시킴
* TWeakObjectPtr : 약참조, 선언한 class가 삭제되도 해당 object의 reference count에 영향 X
여기서 약참조로 선언한 이유는 stage와 별도로 삭제시켜야 하는 상황일 때 강참조가 걸려있으면
참조값이 유효한 상태이므로 GC에서 삭제를 하지 않을 수가 있다.
따라서 약참조로 선언하고 소유객체와 상관없이 삭제할 수 있게 약참조로 선언한 것
   
### 5-2. 소켓 위치 가져오기
```cpp
RewardBoxClass = AABItemBox::StaticClass();
for (FName GateSocket : GateSockets)
{
	//소켓 위치를 가져오는 함수 GetSocketLocation
	FVector BoxLocation = Stage->GetSocketLocation(GateSocket) / 2;
	RewardBoxLocations.Add(GateSocket, BoxLocation);
}
```
   
   
## 6. Asset Manager
---
### 6-1. Asset Manager 에디터 셋팅
> [!abstract] Asset들을 관리하는 Unreal의 싱글톤 클래스
> 관리하고 싶은 Asset들을 Manager에 추가해서 필요할 때 불러올 수 있다

![[Pasted image 20230616143452.png]]
Project Settings > Asset Manager 에서 설정가능
   
### 6-2. Asset ID 지정
GetPrimaryAssetId 함수로 에셋의 이름을 직접 지정
```cpp
FPrimaryAssetId GetPrimaryAssetId() const override
{
	return FPrimaryAssetId("ABItemData", GetFName());//Tag이름 + Asset 이름 = 유일한 식별자 Id
}
```
   
### 6-3. Itembox에 Asset Data 랜덤지정
```cpp
//싱글턴 클래스인 UAssetManager을 가져옴
UAssetManager& Manager = UAssetManager::Get();

TArray<FPrimaryAssetId> Assets;
//태그이름을 입력하고 해당 태그이름과 같은 에셋을 배열로 받음
Manager.GetPrimaryAssetIdList(TEXT("ABItemData"), Assets);
ensure(0 < Assets.Num());

int32 RandomIndex = FMath::RandRange(0, Assets.Num() - 1);
//아이템 박스는 유지할 필요가 없기 때문에 약참조로 선언
FSoftObjectPtr AssetPtr(Manager.GetPrimaryAssetPath(Assets[RandomIndex]));
if (AssetPtr.IsPending()) //에셋이 존재하는지 검사
{
	AssetPtr.LoadSynchronous(); //없다면 로딩
}
Item = Cast<UABItemData>(AssetPtr.Get());
ensure(Item);
```
