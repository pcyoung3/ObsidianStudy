---
Index: "[[UnrealIndex]]"
tags:
  - work
  - unreal
  - Study
---
   
## 1. 엑셀에서 데이터 읽어오기
---
1. FTableRowBase 상속받은 구조체 생성
2. 엑셀의 Name Column을 제외한 나머지들을 UPROPERY 속성으로 선언
3. 엑셀 데이터를 csv로 변경 후 언리얼에 import
   
### 1-1. 엑셀과 동일한 구조체 생성
```cpp
#include "CoreMinimal.h"
#include "Engine/DataTable.h"
#include "ABCharacterStat.generated.h"

USTRUCT(BlueprintType)
struct FABCharacterStat : public FTableRowBase
{
	GENERATED_BODY()

public:
	FABCharacterStat() : MaxHp(0.0f), Attack(0.0f), AttackRange(0.0f), AttackSpeed(0.0f) {}

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = Stat)
	float MaxHp; //엑셀 속성만큼 정의해주면 됨

	...
}
```

> [!tip] reinterpret_cast 와 struct 로 포인터연산
```cpp
//FABCharacterStat는 struct 형식
FABCharacterStat operator+(const FABCharacterStat& Other) const
{
	//FABCharacterStat를 float* (배열)로 캐스팅
	const float* const ThisPtr = reinterpret_cast<const float* const>(this);
	const float* const OtherPtr = reinterpret_cast<const float* const>(&Other);

	FABCharacterStat Result;
	float* ResultPtr = reinterpret_cast<float*>(&Result);

	//float 맴버를 몇개 가지고 있는 계산
	int32 StatNum = sizeof(FABCharacterStat) / sizeof(float);
	for (int32 i = 0; i < StatNum; i++)
	{
		ResultPtr[i] = ThisPtr[i] + OtherPtr[i];
	}

	return Result;
}
```

^a96e76

   
### 1-2. Data Table 생성
![[Pasted image 20230619180401.png]]
1. miscellaneous > Data Table을 선택하고 1-1에서 생성한 struct를 상속
2. Data Table을 열고 Reimport를 하고 csv 파일을 선택
   
   
## 2. 데이터를 관리할 싱글톤 클래스 생성
---
### 2-1. 에디터 셋팅
생성방법 : **Project Settings > Engine > General settings > Default Classes > Advanced > Game Singleton Class**
> 이곳에 등록하면 GEngine 전역변수에 자동으로 생성됨

프로젝트를 재시작해야 변경사항이 반영됨
   
### 2-2. 코드셋팅
1. 헤더
```cpp
UCLASS()
class ARENABATTLE_API UABGameSingleton : public UObject
{
	GENERATED_BODY()
	
public:
	UABGameSingleton();
	static UABGameSingleton& Get();
```
2. cpp
```cpp
UABGameSingleton& UABGameSingleton::Get()
{
	//2-1에서설정한 클래스를 가져오는 부분 : GEngine->GameSingleton 
	UABGameSingleton* Singleton = CastChecked<UABGameSingleton>(GEngine->GameSingleton);
	if (Singleton)
	{
		return *Singleton;
	}

	UE_LOG(LogABGameSingleton, Error, TEXT("Invalid Game Singleton"));
	return *NewObject<UABGameSingleton>();
}
```
   
### 2-3. 데이터 TArray로 옮기기
```cpp
UABGameSingleton::UABGameSingleton()  //생성자에서 데이터를 읽어들임
{
	static ConstructorHelpers::FObjectFinder<UDataTable> DataTableRef(TEXT("/Script/Engine.DataTable'/Game/ArenaBattle/GameData/ABCharacterStatTable.ABCharacterStatTable'"));
	if (nullptr != DataTableRef.Object)
	{
		const UDataTable* DataTable = DataTableRef.Object;
		check(DataTable->GetRowMap().Num() > 0);

		TArray<uint8*> ValueArray;
		//UDataTable은 Map형식으로 되어있지만 지금은 데이터가 순차적으로 입력되어있어서
		//Map 대신 TArray를 사용
		DataTable->GetRowMap().GenerateValueArray(ValueArray);
		Algo::Transform(ValueArray, CharacterStatTable,
			[](uint8* Value)
			{
				return *reinterpret_cast<FABCharacterStat*>(Value);
			}
		);
	}

	CharacterMaxLevel = CharacterStatTable.Num();
	ensure(CharacterMaxLevel > 0);
}
```
   
   
## 3. 지연생성(SpawnActorDeferred)
---
보통 액터를 Spawn 하게 되면 Begin Play 함수가 바로 호출된다.
하지만 그 전에 무언가 처리를 하고 싶다면 SpawnActorDeferred 함수를 사용하면 된다.
![[Pasted image 20230619192105.png]]
```cpp
const FTransform SpawnTransform(GetActorLocation() + FVector::UpVector * 88.0f);
//SpawnActorDeferred 사용
AABCharacterNonPlayer* ABOpponentCharacter = GetWorld()->SpawnActorDeferred<AABCharacterNonPlayer>(OpponentClass, SpawnTransform);
if (ABOpponentCharacter)
{
	ABOpponentCharacter->OnDestroyed.AddDynamic(this, &AABStageGimmick::OnOpponentDestroyed);
	ABOpponentCharacter->SetLevel(CurrentStageNum);
	//FinishSpawning로 이 이후부터 BeginPlay가 이루어짐
	ABOpponentCharacter->FinishSpawning(SpawnTransform);
}
```
   
   
## 4. ini 파일 데이터 가져오기
---
config 폴더의 ini파일의 데이터를 가져올 수 있다
   
### 4-1. ini 파일셋팅
**\<DefaultArenaBattle.ini>**
```
[/Script/ArenaBattle.ABCharacterNonPlayer]
+NPCMeshes=/Game/InfinityBladeWarriors/Character/CompleteCharacters/SK_CharM_Barbarous.SK_CharM_Barbarous 
+NPCMeshes=/Game/InfinityBladeWarriors/Character/CompleteCharacters/sk_CharM_Base.sk_CharM_Base 
```
+ 꺽쇠모양\[ ] 안은 경로를 의미한다. /Script 안쪽이면 C++ 파일이고 ArenaBattle모듈의 ABCharacterNonPlayer은 UObject를 의미
+ \+NPCMeshes에서 +는 TArray 의미
-> 즉, NPCMeshes라는 TArray변수가 ABCharacterNonPlayer에 선언되어있다면 값 지정이 가능
   
### 4-2. Class 셋팅
**\<Header>**
```cpp
#include "Engine/StreamableManager.h"
...

UCLASS(config=ArenaBattle) //config 이름 셋팅(DefaultArenaBattle.ini)
class ARENABATTLE_API AABCharacterNonPlayer : public AABCharacterBase
{
	GENERATED_BODY()
	
	...
	
	UPROPERTY(config) //config 파일로부터 불러오겠다는 의미
	TArray<FSoftObjectPath> NPCMeshes; //ini와 같은 이름
	
	TSharedPtr<FStreamableHandle> NPCMeshHandle; //비동기화로 로딩
}
```
**\<Cpp - AsyncLoad>**
```cpp
ensure(NPCMeshes.Num() > 0);
int32 RandIndex = FMath::RandRange(0, NPCMeshes.Num() - 1);
//UAssetManager를 활용
//RequestAsyncLoad : 비동기화 함수
//NPCMeshLoadCompleted : 완료 시 delegate 호출함수
NPCMeshHandle = UAssetManager::Get().GetStreamableManager().RequestAsyncLoad(NPCMeshes[RandIndex], FStreamableDelegate::CreateUObject(this, &AABCharacterNonPlayer::NPCMeshLoadCompleted));
```
**\<Cpp - Loading 완료 후 처리>**
```cpp
void AABCharacterNonPlayer::NPCMeshLoadCompleted()
{
	if (NPCMeshHandle.IsValid())
	{
		//GetLoadedAsset : Handle로부터 mesh를 가져오는 함수
		USkeletalMesh* NPCMesh = Cast<USkeletalMesh>(NPCMeshHandle->GetLoadedAsset());
		if (NPCMesh)
		{
			GetMesh()->SetSkeletalMesh(NPCMesh); //SkeletalMesh 셋팅
			GetMesh()->SetHiddenInGame(false); //Mesh 보이게 처리
		}
	}
	NPCMeshHandle->ReleaseHandle(); //Handle 해제
}
```
