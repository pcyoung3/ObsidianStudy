# Index : [[UnrealIndex]]
## Tag : #work #unreal #Study 
---
   
## 1. Module
---
> [!bug] 모듈이 이 부분에서 굳이 언급되어야 할까?
### 1-1. Module이란?
일반 프로젝트 파일들에서 dll에 연관되는 개념
.cpp + .h 의 조합. 언리얼에서 어떤 기능을 만들 때 최소단위라고 볼 수 있음
Unreal Project는 Module과 Plugin이 모여서 만들어진 개념

> [!note] LyraStarterGame.uproject
```json
	"Modules": [
		{
			"Name": "LyraGame",
			"Type": "Runtime",
			"LoadingPhase": "Default",
			"AdditionalDependencies": [
				"DeveloperSettings",
				"Engine"
			]
		},
		{
			"Name": "LyraEditor",
			"Type": "Editor",
			"LoadingPhase": "Default"
		}
	],
	"Plugins": [
		{
			"Name": "ActorPalette",
			"Enabled": true
		},
		{
			"Name": "GameplayAbilities",
			"Enabled": true
			...
```

이 때 언리얼 프로젝트는 최소 1개의 필수 Module을 가지고 있어야 함
Plugin은 Module을 1개 이상 포함하며 Plugin안에 Plugin이 추가로 들어갈 수도 있음
![[KakaoTalk_20231130_133205684.jpg|300]]
구조를 보자면 다음과 같다
   
### 1-2. Lyra의 Module 적용
필수모듈을 적용하는 방법 -> 일반모듈과 방법이 다름
모듈의 Start는 일종의 Main 함수로 볼 수 있다
```cpp
#include "LyraClone.h"
#include "Modules/ModuleManager.h"

//필수 모듈의 경우 FDefaultGameModuleImpl 상속이 필요
class FLyraCloneGameModule : public FDefaultGameModuleImpl
{
	virtual void StartupModule() override;
	virtual void ShutdownModule() override;
};

//Unreal Editor가 켜질 때 실행
void FLyraCloneGameModule::StartupModule()
{
	UE_LOG(LogTemp, Warning, TEXT("StartupModule"));
}

//Unreal Editor가 꺼질 때 실행
void FLyraCloneGameModule::ShutdownModule()
{
	UE_LOG(LogTemp, Warning, TEXT("ShutdownModule"));
}

//일반 모듈의 경우 IMPLEMENT_MODULE 사용
//필수 모듈의 경우 IMPLEMENT_PRIMARY_GAME_MODULE 사용
IMPLEMENT_PRIMARY_GAME_MODULE(FLyraCloneGameModule, LyraClone, "LyraClone");
```
   
   
## 2. AssetManager
---
### 2-1. Unreal 파일 생성 방법
1. Windows 파일창에서 텍스트 파일로 바로 생성
2. Engine의 GenerateProjectFiles.bat 파일 실행
3. Visual Studio에서 Reload

   
### 2-3. Asset Manager 개요
프로젝트의 Asset(Resource)들을 로딩 및 캐싱해주는 Class 일종의 Resource Manager
ConstructorHelper 에서 일일히 Asset들의 경로를 하드코딩하지 않기 위해 만드는 것
당장 사용은 하지 않지만 프로젝트 내에서 중요도가 높은 Class 이므로 이것으로 시작
   
### 2-2. 코드작성 중 알아야 할 배경지식
> [!note] 헤더파일
```cpp
#pragma once

#include "Engine/AssetManager.h"
#include "LyraCloneAssetManager.generated.h"  //Generated.h 파일은 리플랙션 때문에 필수
UCLASS()  //Class 작성 시 필수 리플랙션
class ULyraCloneAssetManager : public UAssetManager
{
	GENERATED_BODY()  //리플랙션 때문에 필수로 기입해줘야 함
public:
	ULyraCloneAssetManager();
public:
	static bool ShouldLogAssetLoads();
	static void TestClone(); //static 함수 사용의 이유는 바로 호출가능하기 때문에 디버깅에 유리함
};
```

> [!note] cpp파일
```cpp
#include "LyraCloneAssetManager.h"

// UE_INLINE_GENERATED_CPP_BY_NAME
// generated.h 와 연계되는 개념
// generated.cpp 를 inline으로 추가해서 컴파일 시간을 줄여줌
#include UE_INLINE_GENERATED_CPP_BY_NAME(LyraCloneAssetManager)

ULyraCloneAssetManager::ULyraCloneAssetManager() : Super() //상속된 클래스므로 항상 상위 생성자 호출 필수
{
	
}

// [PRAGMA_DISABLE_OPTIMIZATION, PRAGMA_ENABLE_OPTIMIZATION] 
// Development Editor 로 컴파일을 진행했을 경우 컴파일러가 코드를 알아서 생성 및 리팩토링을 진행
// 이 때 코드가 의도하지 않게 변경될 수도 있으므로 해당 부분을 막는 코드
// Debug 시에 Delevelopment 모드면 의도한대로 Debug가 안되는데 해당 부분을 방지 
PRAGMA_DISABLE_OPTIMIZATION
bool ULyraCloneAssetManager::ShouldLogAssetLoads()
{
	// FCommandLine::Get() : Command Arguments를 가져오는 함수
	const TCHAR* commandLineContent = FCommandLine::Get();
	
	// FParse::Param() : 문자열 중 특정 Argument가 있는지를 검사하는 함수
	// static으로 선언?
	// 함수 내부에 static으로 선언하면 해당 함수에서만 사용 가능한 전역변수가 됨(함수 나가도 삭제 안됨)
	// 그리고 초기화 구문은 다시 타지 않기 때문에 최초 한번만 타게 됨
	static bool bLogAssetLoads = FParse::Param(commandLineContent, TEXT("LogAssetLoads"));
	return bLogAssetLoads;
}

void ULyraCloneAssetManager::TestClone()
{
	ShouldLogAssetLoads();
}
PRAGMA_ENABLE_OPTIMIZATION

```
   
### 2-3. 전체코드 설명
코드 작성 뒤에 Editor Asset Manager에서 지정까지 해야 StartInitialLoading 함수가 실행
> [!note] 헤더파일
```cpp
#pragma once

#include "Engine/AssetManager.h"
#include "LyraCloneAssetManager.generated.h"

UCLASS()
class ULyraCloneAssetManager : public UAssetManager
{
	GENERATED_BODY()
public:
	ULyraCloneAssetManager();

public:
	static void TestClone();
	static bool ShouldLogAssetLoads();

	static ULyraCloneAssetManager& Get();
	static UObject* SynchronousLoadAsset(const FSoftObjectPath& AssetPath);

	//UAssetManager's interfaces
	virtual void StartInitialLoading() final;

	void AddLoadedAsset(const UObject* Asset);

	template<typename AssetType>
	static AssetType* GetAsset(const TSoftObjectPtr<AssetType>& AssetPointer, bool bKeepInMemory = true);

	template<typename AssetType>
	static TSubclassOf<AssetType> GetSubclass(const TSoftClassPtr<AssetType>& AssetPointer, bool bKeepInMemory = true);

public:
	//관리할 Asset들을 저장하는 메인변수
	UPROPERTY()		//GC를 위해 UPROPERTY() 선언
	TSet<TObjectPtr<const UObject>> LoadedAssets;

	//Object 단위 Locking
	FCriticalSection SyncObject;
};

/*
* bKeepInMemory : 메모리에 캐싱할 것인가?
* 캐싱은 쉽게 말하자면 메모리에 계속 남겨놓을 것인가를 의미
* 
* instance를 Return 하는 것과 TSubclassOf를 Return 하는 것이 있는데
* instance : 이미 생성된 instance를 가져올 때 사용
* TSubclassOf : UClass 유형의 값을 받기 때문에 클래스 자료형 자체를 받을 수 있음
*/
template<typename AssetType>
AssetType* ULyraCloneAssetManager::GetAsset(const TSoftObjectPtr<AssetType>& AssetPointer, bool bKeepInMemory)
{
	AssetType* LoadedAsset = nullptr;

	const FSoftObjectPath& AssetPath = AssetPointer.ToSoftObjectPath();	//Path로 변환

	if (AssetPath.IsValid())	//path가 있다면
	{
		LoadedAsset = AssetPointer.Get();
		if (!LoadedAsset) //로딩이 되지 않았다면?
		{
			LoadedAsset = Cast<AssetType>(SynchronousLoadAsset(AssetPath)); //다시 로딩시킴
			ensureAlwaysMsgf(LoadedAsset, TEXT("Failed to load asset [%s]"), *AssetPointer.ToString());
		}

		if (LoadedAsset && bKeepInMemory)
		{
			//LoadedAssets 변수에 들어온 Asset을 저장
			//UPROPERTY로 되어있기 때문에 GC 관리가 자동을 되고 해당 UObject를 직적 내리지 않는 한 unload가 되지않음(=캐싱)
			Get().AddLoadedAsset(Cast<UObject>(LoadedAsset));
		}
	}

	return LoadedAsset;
}

template<typename AssetType>
TSubclassOf<AssetType> ULyraCloneAssetManager::GetSubclass(const TSoftClassPtr<AssetType>& AssetPointer, bool bKeepInMemory)
{
	TSubclassOf<AssetType> LoadedSubclass;

	const FSoftObjectPath& AssetPath = AssetPointer.ToSoftObjectPath();

	if (AssetPath.IsValid())
	{
		LoadedSubclass = AssetPointer.Get();
		if (!LoadedSubclass)
		{
			LoadedSubclass = Cast<UClass>(SynchronousLoadAsset(AssetPath));
			ensureAlwaysMsgf(LoadedSubclass, TEXT("Failed to load asset class [%s]"), *AssetPointer.ToString());
		}

		if (LoadedSubclass && bKeepInMemory)
		{
			// Added to loaded asset list.
			Get().AddLoadedAsset(Cast<UObject>(LoadedSubclass));
		}
	}

	return LoadedSubclass;
}
```

> [!note] cpp파일
```cpp
#include "LyraCloneAssetManager.h"
#include UE_INLINE_GENERATED_CPP_BY_NAME(LyraCloneAssetManager)

ULyraCloneAssetManager::ULyraCloneAssetManager() : Super()
{

}

void ULyraCloneAssetManager::TestClone()
{
	ShouldLogAssetLoads();
}

bool ULyraCloneAssetManager::ShouldLogAssetLoads()
{
	const TCHAR* commandLineContent = FCommandLine::Get();
	static bool bLogAssetLoads = FParse::Param(commandLineContent, TEXT("LogAssetLoads"));
	return bLogAssetLoads;
}

ULyraCloneAssetManager& ULyraCloneAssetManager::Get()
{
	check(GEngine);	//GEngine : Unreal의 코어한 전역변수

	//ULyraCloneAssetManager는 UAssetManager의 상속이기 때문에 GEngine이 가지고 있다
	if (ULyraCloneAssetManager* Singleton = Cast<ULyraCloneAssetManager>(GEngine->AssetManager))
	{
		return *Singleton;
	}

	UE_LOG(LogTemp, Display, TEXT("Invalid AssetManagerClassName in DefaultEngine.ini.  It must be set to LyraCloneAssetManager!"));

	return *NewObject<ULyraCloneAssetManager>();	//여기까지 오면 에러긴 하지만 컴파일을 위해 더미리턴
}

/*
* SynchronousLoadAsset 함수는 로딩함수인데 동기화 함수
* 로딩함수는 보통 다른 작업과 병행하면서 진행하기 때문에 비동기화 함수를 자주 사용
* 동기화 함수를 구현한 이유?
* 1. 동기화 로딩이 비동기화 로딩보다 좀 더 빠르다
* 2. SynchronousLoadAsset 함수를 진행할 때 로딩에 걸리는 시간을 측정하고 오래 걸리는 것들을 가려냄
* 로딩 시간이 오래걸리면 동기화함수의 경우 게임 전체가 버벅거리는 현상이 발생하기 때문에 비동기화 함수로 옮겨야 함
*/
UObject* ULyraCloneAssetManager::SynchronousLoadAsset(const FSoftObjectPath& AssetPath)
{
	if (AssetPath.IsValid())
	{
		TUniquePtr<FScopeLogTime> LogTimePtr;

		if (ShouldLogAssetLoads())	//Argument가 들어왔다면?
		{
			//현재 로딩시간이 얼마나 걸렸는지 체크
			LogTimePtr = MakeUnique<FScopeLogTime>(*FString::Printf(TEXT("Synchronously loaded asset [%s]"), *AssetPath.ToString()), nullptr, FScopeLogTime::ScopeLog_Seconds);
		}

		//AssetManager가 존재하면 AssetManager의 GetStreamableManager().LoadSynchronous 함수를 통해서 로딩
		if (UAssetManager::IsValid())
		{
			return UAssetManager::GetStreamableManager().LoadSynchronous(AssetPath, false);
		}

		// AssetManager가 아직 생성되지 않은 상황이라면 그냥 들어온 Path로 정적로딩
		// 해당 함수는 이미 로딩이 되어있으면 찾아오고 로딩이 안되어있을때만 로딩을 실행
		return AssetPath.TryLoad();
	}

	return nullptr;
}

/*
* GameplayTag, GamequeueManager, 시작 시점 변경 등
* 게임 가장 초기에 초기화해야 할 부분들을 구현하는 곳
*/
void ULyraCloneAssetManager::StartInitialLoading()
{
	Super::StartInitialLoading();

	//우선은 로그만 띄워주고 넘어감
	UE_LOG(LogTemp, Display, TEXT("ULyraCloneAssetManager::StartInitialLoading"));
}

/*
* 로딩된 에셋 추가 함수
*/
void ULyraCloneAssetManager::AddLoadedAsset(const UObject* Asset)
{
	/*
	* Thread Safe를 위해서 Lock 실행
	*/
	if (ensureAlways(Asset))	//Asset이 nullptr이라면 항상 경고, nullptr이 아니면 Asset 추가
	{
		FScopeLock LoadedAssetsLock(&SyncObject);	//Lock을 검
		LoadedAssets.Add(Asset);
	}
}
```
   
   
## 3. Log Chennel 생성
---
여러 로그를 띄울 때 복잡할 수 있으므로 Custom Log Chennel을 하나 생성
### 생성
> [!note] header 파일
```cpp
#pragma once

#include "Containers/UnrealString.h"
#include "Logging/LogMacros.h"

LYRACLONE_API DECLARE_LOG_CATEGORY_EXTERN(LogLyraClone, Log, All);
```

> [!note] cpp 파일
```cpp
#include "LyraClone/LyraCloneChannels.h"

DEFINE_LOG_CATEGORY(LogLyraClone)
```

### 사용
```cpp
#include "LyraClone/LyraCloneChannels.h"	//Log용
...
UE_LOG(LogLyraClone, Warning, TEXT("ShutdownModule"));
```
   
   
## 4. Experience
---
GameFeature를 이용해서 한 게임에서 여러 종류의 모드를 나눌 수 있게 하는 셋팅
아직 모드가 바뀌지는 않고 그것을 위한 기반작업이라고 보면 됨
Experience는 구현되는 각 모드라고 생각하면 됨. 유저가 경험할 Rule과 게임의 정보를 담고 있는 객체
Experience에 들어가는 것들이 GameFeature로서 모듈처럼 들어감
다만 아직 Game Feature는 이 단계에서 활성화하지는 않는다. 차후에 할 예정

또한 전부 UPrimaryDataAsset을 상속하는데 여기서 정보를 관리해줄 객체라는 것을 알 수 있음

만들 Class 구조는 다음과 같음
![[GameFeature.png]]
   
> [!success] ULyraCloneUserFacingExperienceDefinition
> 맵과 Experience의 정보를 담고 있는 객체

> [!note] ULyraCloneUserFacingExperienceDefinition.h
```cpp
#pragma once

#include "Engine/DataAsset.h"
#include "LyraCloneUserFacingExperienceDefinition.generated.h"

/*
* 맵과 Experience의 정보를 담고 있는 객체
*/

UCLASS(BlueprintType)
class ULyraCloneUserFacingExperienceDefinition : public UPrimaryDataAsset
{
	GENERATED_BODY()
public:
	//해당 Game Feature에서 사용할 Map
	UPROPERTY(BlueprintReadWrite, EditAnywhere, Category = Experience, meta = (AllowedTypes = "Map"))
	FPrimaryAssetId MapID;

	//해당 Game Feature에서 사용할 Experience
	UPROPERTY(BlueprintReadWrite, EditAnywhere, Category=Experience, meta=(AllowedTypes="LyraCloneExperienceDefinition"))
	FPrimaryAssetId ExperienceID;

};
```

> [!note] ULyraCloneUserFacingExperienceDefinition.cpp
```cpp
#include "LyraCloneUserFacingExperienceDefinition.h"
#include UE_INLINE_GENERATED_CPP_BY_NAME(LyraCloneUserFacingExperienceDefinition)
```
   
   
> [!success] ULyraCloneExperienceDefinition
> Spawn 할 Pawn의 정보(Camera mode, Ability Set, Input type 등)를 담고
> 활성화 할 Game Feature의 목록을 담는 객체

> [!note] LyraCloneExperienceDefinition.h
```cpp
#pragma once

#include "Engine/DataAsset.h"
#include "LyraCloneExperienceDefinition.generated.h"

class ULyraClonePawnData;

/*
* 게임의 정보를 담고 있는 Class
*/
UCLASS()
class ULyraCloneExperienceDefinition : public UPrimaryDataAsset
{
	GENERATED_BODY()
public:
	ULyraCloneExperienceDefinition(const FObjectInitializer& ObjectInitializer = FObjectInitializer::Get());

	// 활성화 할 Game Feature의 목록
	UPROPERTY(EditDefaultsOnly, Category = Gameplay)
	TArray<FString> GameFeaturesToEnable;

	// Pawn을 Spawn하기 위한 정보를 담고 있음
	// Input Type, Ability Set, Camera Mode 등
	UPROPERTY(EditDefaultsOnly, Category = Gameplay)
	TObjectPtr<ULyraClonePawnData> DefaultPawnData;
};
```

> [!note] LyraCloneExperienceDefinition.cpp
```cpp
#include "LyraCloneExperienceDefinition.h"
#include UE_INLINE_GENERATED_CPP_BY_NAME(LyraCloneExperienceDefinition)

ULyraCloneExperienceDefinition::ULyraCloneExperienceDefinition(const FObjectInitializer& ObjectInitializer) : Super(ObjectInitializer)
{
}
```
   
   
> [!success] ULyraClonePawnData
> 게임 내에서 사용될 Pawn이 가져야 하는 데이터 및 속성을 정의

> [!note] LyraClonePawnData.h
```cpp
#pragma once

#include "Coreminimal.h"
#include "Engine/DataAsset.h"
#include "LyraClonePawnData.generated.h"

/*
* Pawn 상속이 아닌 UPrimaryDataAsset를 상속
* 해당 클래스는 게임 내에서 사용될 Pawn이 가져야 하는
* 데이터 및 속성을 정의
*/
UCLASS(BlueprintType)
class ULyraClonePawnData : public UPrimaryDataAsset
{
	GENERATED_BODY()
public:
	ULyraClonePawnData(const FObjectInitializer& ObjectInitializer);
};
```

> [!note] LyraClonePawnData.cpp
```cpp
#include "LyraClonePawnData.h"
#include UE_INLINE_GENERATED_CPP_BY_NAME(LyraClonePawnData)

ULyraClonePawnData::ULyraClonePawnData(const FObjectInitializer& ObjectInitializer)
{
}
```
   
   
> [!note]  Blueprint 생성
1. Content 폴더에 System 및 DefaultEditorMap / Experience / Playlist 생성
![[Pasted image 20231205132735.png]]

2. DefaultEditorMap 내부에 파일 3개 생성
	2-1. Actor 상속 B_ExperienceList3D
	2-2. Actor 상속 B_TeleportToUserFacingExperience
	2-3. 새로운 Map > Basic > L_DefaultEditorOverView로 Map 저장

3. Blueprint 로 LyraCloneExperienceDefinition를 상속하여 B_LyraCloneDefaultExperience 생성
4. DataAsset 타입으로 LyraCloneUserFacingExperienceDefinition를 상속한 DA_ExamplePlaylist 생성

> [!note]  Editor 셋팅

셋팅을 하는 이유는 UPrimaryDataAsset 타입들은 Asset Manager에 등록해주면 검색이 가능해지기 때문
1. Project Settings > Asset Mananger > Primary Asset Types to Scan 에서 추가
2. Map Index에서 Specific Assets 에서 만든 Map을 지정(Maps 폴더를 실제로 만들지 않기 때문)
   ![[Pasted image 20231205133850.png]]
3. LyraCloneUserFacingExperienceDefinition 지정 및 Directories 지정
   ![[Pasted image 20231205134050.png]]
4. LyraCloneExperienceDefinition 지정 및 Directories 지정
   ![[Pasted image 20231205135311.png]]
   
> [!note]  Blueprint 셋팅

1. DA_ExamplePlaylist 에 Map과 Experience 지정
   > [!warning] 주의 : 위에서 Primary Asset Type의 이름과 Asset Base Class 이름에 오타나면 이쪽에서 표시안됨

	![[Pasted image 20231205141144.png]]

2. B_LyraCloneDefaultExperience는 우선 설정 안함
3. B_TeleportToUserFacingExperience 셋팅
   ![[Pasted image 20231205141344.png]]![[Pasted image 20231205141405.png]]
   Component에는 Capsule Collider와 Static Mesh 추가
   
   ![[Pasted image 20231205141709.png]]
   변수부에는 LyraCloneUserFacingExperienceDefinition 타입의 UserFacingExperienceToLoad 생성
   해당 변수는 Expose on Spawn도 필수로 해야됨

4. B_ExperienceList3D 셋팅
   **<변수>**
   ![[Pasted image 20231205142516.png]]
   
   **\<Begin Play>**
   ![[Pasted image 20231205142053.png]]
   ![[Pasted image 20231205142129.png]]
   ![[Pasted image 20231205142115.png]]
   

