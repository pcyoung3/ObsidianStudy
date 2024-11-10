---
Index: "[[4. UnrealIndex]]"
tags:
  - work
  - unreal
  - Study
---
   
> [!note] 언리얼 게임 제작 시 기초

## 1. Source 파일구조
---
> [!tip] Source 폴더 내에 또 다른 폴더를 만들 때 폴더구조 셋팅

![[Pasted image 20230530164425.png]]
Source 폴더 내에 또 다른 폴더를 만들 때 참조해야 하는 폴더 구조 셋팅

### 1. 각 Header 파일마다 폴더 구조 셋팅
```cpp
#include "ArenaBattle/Player/ABPlayerController.h"
```
각 헤더파일에서 모듈 이름 경로를 앞에 붙임
   
### 2. Build.cs 파일에 추가
```c#
using UnrealBuildTool;

public class ArenaBattle : ModuleRules
{
	public ArenaBattle(ReadOnlyTargetRules Target) : base(Target)
	{
		...
		PublicIncludePaths.AddRange(new string[] { "ArenaBattle" });
		...
```
{ModuleName}.Build.cs 파일에 추가하면
```cpp
#include "Player/ABPlayerController.h"
```
include 할 때 경로를 다음과 같이 일괄적으로 줄일 수 있음
   
   
## 2.  Class CDO에서 Loading
---
### 1. Game mode에서 Default Pawn Class 셋팅
```cpp
//DefaultPawnClass
static ConstructorHelpers::FClassFinder<APawn> ThirdPersonClassRef(TEXT("/Game/ThirdPerson/Blueprints/BP_ThirdPersonCharacter.BP_ThirdPersonCharacter_C"));
if (ThirdPersonClassRef.Class)
{
	DefaultPawnClass = ThirdPersonClassRef.Class;
}
```
> [!todo] 클래스형태의 에셋을 가져오는 방법
> BP_ThirdPersonCharacter_C 처럼 "\_C" 를 붙이면 된다
   
   
## 3. Gamemode 셋팅
---
### 3-1. Game mode에서 Player Controller 셋팅
```cpp
//Gamemode
AABGameMode::AABGameMode()
{
	#include "Player/ABPlayerController.h"
	...
	PlayerControllerClass = AABPlayerController::StaticClass();
}
```
   
### 3-2. include 없이 Player Controller 셋팅
```cpp
//Gamemode
AABGameMode::AABGameMode()
{
	static ConstructorHelpers::FClassFinder<APlayerController> PlayerControllerClassRef(TEXT("/Script/ArenaBattle.ABPlayerController"));
	if (PlayerControllerClassRef.Class)
	{
		PlayerControllerClass = PlayerControllerClassRef.Class;	//include를 사용하지 않고도 불러올 수 있다.
	}
}
```
   
   
> [!info] 게임 시작할 때 마우스를 가두는 셋팅
```cpp
//PlayerController
void AABPlayerController::BeginPlay()
{
	Super::BeginPlay();

	//게임이 시작할 때 마우스를 가두는 코드
	FInputModeGameOnly GameOnlyInputMode;
	SetInputMode(GameOnlyInputMode);
}
```
