# Index : [[UnrealIndex]]
## Tag : #work #unreal #Study 
---
   
## 1. 행동트리 이론
---
![[Pasted image 20230626174021.png]]
![[Pasted image 20230626174119.png]]
### Composite 종류
* Selector : 여러 행동 중 하나의 행동을 지정
* Sequence : 여러 행동을 차례로 모두 수행
* Parallel : 여러 행동을 함께 수행
   
### Composite 노드에 부착하는 기능들
* Decorator : Composite 노드가 실행되는 조건 지정
* Service : Composite 노드가 활성화 될 때 주기적으로 실행
* Abort : Decorator 조건에 부합되면 Composite 내 활동을 전부 중단
   
#### Decorator 사용예시
![[Pasted image 20230626175645.png]]
#### Abort 사용예시
![[Pasted image 20230626175830.png]]
   
   
## 2. Behavior Tree 추가
---
![[Pasted image 20230626195658.png]]
Content Browser 우클릭 > Artificial Intelligence 에서 Asset 생성가능
* Behavior Tree : 여러 패턴을 사용하게 될 행동트리
* Blackboard : Behavior Tree에서 사용할 변수들을 저장
   
   
## 3. Behavior Tree 코드
---
### 3-1. 선언
```cpp
UPROPERTY()
TObjectPtr<class UBlackboardData> BBAsset;

UPROPERTY()
TObjectPtr<class UBehaviorTree> BTAsset;
```
   
### 3-2. 생성
```cpp
static ConstructorHelpers::FObjectFinder<UBlackboardData> BBAssetRef(TEXT("/Script/AIModule.BlackboardData'/Game/ArenaBattle/AI/BB_ABCharacter.BB_ABCharacter'"));
if (nullptr != BBAssetRef.Object)
{
	BBAsset = BBAssetRef.Object;
}

static ConstructorHelpers::FObjectFinder<UBehaviorTree> BTAssetRef(TEXT("/Script/AIModule.BehaviorTree'/Game/ArenaBattle/AI/BT_ABCharacter.BT_ABCharacter'"));
if (nullptr != BTAssetRef.Object)
{
	BTAsset = BTAssetRef.Object;
}
```
   
### 3-3. Pawn이 빙의했을 때 함수 정의
---
```cpp
...
#include "BehaviorTree/BehaviorTree.h"
#include "BehaviorTree/BlackboardData.h"
#include "BehaviorTree/BlackboardComponent.h"
...
void AABAIController::OnPossess(APawn* InPawn)
{
	Super::OnPossess(InPawn);

	RunAI();
}

...

void AABAIController::RunAI()
{
	UBlackboardComponent* BlackboardPtr = Blackboard.Get();
	if (UseBlackboard(BBAsset, BlackboardPtr))
	{
		bool RunResult = RunBehaviorTree(BTAsset);
		ensure(RunResult);
	}
}
```
   
### 3-4. AI를 멈추는 함수 정의
---
```cpp
void AABAIController::StopAI()
{
	//BrainComponent AIController의 변수
	UBehaviorTreeComponent* BTComponent = Cast<UBehaviorTreeComponent>(BrainComponent);
	if (BTComponent)
	{
		BTComponent->StopTree();
	}
}
```
