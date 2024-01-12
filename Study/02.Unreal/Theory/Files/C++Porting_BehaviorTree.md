---
Index: "[[UnrealIndex]]"
tags:
  - unreal
  - Study
---
   
## 1. 행동트리 기본 이론
---
행동트리의 기본 노드 등의 이론은 [[이득우Framework_11_행동트리 모델의 이해|행동트리 개론]] 참고
   
## 2. Build.cs 내용추가
---
[[이득우Framework_12_행동트리 구현#2. Behavior Tree C++ 설정|Build.cs]] 파일에 모듈을 추가해야 함
   
## 3. Decorator C++
---
```cpp
virtual bool CalculateRawConditionValue(UBehaviorTreeComponent& OwnerComp, uint8* NodeMemory) const override;
```
[[이득우Framework_12_행동트리 구현#6. BTDecorator Class 생성|Decorator C++ 생성]] 참고
   
## 4. Service C++
---
[[이득우Framework_12_행동트리 구현#4. BTService Class 생성|Service C++ 생성]] 참고
추가적으로 Overriding 할 수 있는 함수들은 다음과 같다
```cpp
UCLASS()  
class D1_API US_UpdateBehavior : public UBTService  
{  
    GENERATED_BODY()  
protected:  
	//해당 노드에 진입했을 때 호출
    virtual void OnSearchStart(FBehaviorTreeSearchData& SearchData) override;  
    //Tick마다 호출
    virtual void TickNode(UBehaviorTreeComponent& OwnerComp, uint8* NodeMemory, float DeltaSeconds) override;  
    ...
}
```
   
## 5. Task C++
---
```cpp
virtual EBTNodeResult::Type ExecuteTask(UBehaviorTreeComponent& OwnerComp, uint8* NodeMemory) override;
```
[[이득우Framework_12_행동트리 구현#3. Behavior Tree Task Class 생성|Task C++ 생성]] 참고
   
   
## 6. Blackboard Key Get/Set
---
Behavior Tree 에서는 각 Node들에서 공통적으로 쓸 수 있는 값(전역변수)을
Blackboard에 저장하는데 해당 변수들은 Blueprint에서 정의되므로
변수들을 가져오기 위해서는 해줘야 하는 작업들이 있다.
   
### 6-1. Blackboard에서 변수 생성
![[Pasted image 20230918161829.png]]
Blueprint Blackboard에서 공통으로 쓸 변수들을 셋팅
   
### 6-2. C++ BlackboardComponent 가져오기
우선 BlackboardComponent를 먼저 가져와야 된다

1. Service, Task 등에서는 매개변수에 있어서 바로 가져올 수 있다
```cpp
void /*Service*/::OnSearchStart(FBehaviorTreeSearchData& SearchData)
{
	BlackboardComponent = SearchData.OwnerComp.GetBlackboardComponent();
}
```
```cpp
EBTNodeResult::Type /*Task*/::ExecuteTask(UBehaviorTreeComponent& OwnerComp, uint8* NodeMemory)
{
	BlackboardComponent = OwnerComp.GetBlackboardComponent();
}
```

2. 그 외에서는 Controller를 이용해서 가져올 수 있다
```cpp
#include "BehaviorTree/BlackboardComponent.h"

...

AAIController* aiController = Cast<AAIController>(GetController());  
if(aiController)  
{  
    UBlackboardComponent* BlackboardComponent = aiController->GetBlackboardComponent();  
    if(BlackboardComponent)  
       BlackboardComponent->SetValueAsBool(TEXT("bIsDead"), bIsDead);  
}
```
   
   
### 6-3. FBlackboardKeySelector
해당 자료형으로 Blackboard에서 Key를 가져오거나 Setting을 할 수 있다
```cpp
private:  
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="BlackBoard", meta=(AllowPrivateAccess="true"))  
    FBlackboardKeySelector BlackboardKey_BehaviorKey;  
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="BlackBoard", meta=(AllowPrivateAccess="true"))  
    FBlackboardKeySelector BlackboardKey_TargetKey;  
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="BlackBoard", meta=(AllowPrivateAccess="true"))  
    FBlackboardKeySelector BlackboardKey_AttackingRangeType;
```

6-2의 BlackboardComponent를 이용해서 BlackboardKey를 Setting 할 수 있다
```cpp
BlackboardComponent->SetValueAsEnum(BlackboardKey_BehaviorKey.SelectedKeyName, static_cast<uint8>(NewBehavior));

BlackboardComponent->GetValueAsObject(BlackboardKey_TargetKey.SelectedKeyName)
```
   
   
### 6-4. Editor Setting
![[Pasted image 20230919174035.png]]
FBlackboardKeySelector 를 Unreal Editor에서 노출시키면 다음과 같이 BlackBoard의 변수를 직접 매칭시킬 수 있다.
이렇게 하면 C++과 Blackboard의 변수를 동기화 할 수 있다
