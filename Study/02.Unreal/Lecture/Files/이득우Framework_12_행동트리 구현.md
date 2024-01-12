---
Index: "[[UnrealIndex]]"
tags:
  - work
  - unreal
  - Study
---
   
참고 : [[이득우Framework_11_행동트리 모델의 이해]]
## 1. Navigation Mesh 추가
---
![[Pasted image 20230627164022.png]]
1. Place Actor 에서 NavMeshBoundsVolume을 드래그 해 Level에 추가
![[Pasted image 20230627164114.png]]
2. 추가한 NaviMesh에서 Brush Settings에서 범위를 늘려서 원하는 영역을 설정
3. Project에서 실시간으로 Navigation Mesh가 변경된다면
	Project Settings > Navigation Mesh > Runtime Generation을 Dynamic으로 설정
   
   
## 2. Behavior Tree C++ 설정
---
### 2-1. Build에 Behavior Tree Module 추가
```cpp
PublicDependencyModuleNames.AddRange(new string[] { "Core", ... "NavigationSystem", "AIModule", "GameplayTasks" });
```
   
### 2-2. BlackBoard에 값설정
```cpp
UBlackboardComponent* BlackboardPtr = Blackboard.Get();
if (UseBlackboard(BBAsset, BlackboardPtr))
{
	Blackboard->SetValueAsVector(BBKEY_HOMEPOS, GetPawn()->GetActorLocation());
	...
```
   
   
## 3. Behavior Tree Task Class 생성
---
### 3-1. BTTask를 상속받아 Class 생성
![[Pasted image 20230627164909.png]]
   
### 3-2. BTTask ExecuteTask 설정
#### 1) 오버라이딩
```cpp
virtual EBTNodeResult::Type ExecuteTask(UBehaviorTreeComponent& OwnerComp, uint8* NodeMemory) override;
```

```cpp
EBTNodeResult::Type UBTTask_FindPatrolPos::ExecuteTask(UBehaviorTreeComponent& OwnerComp, uint8* NodeMemory)
{
	//super 실행
	EBTNodeResult::Type Result = Super::ExecuteTask(OwnerComp, NodeMemory);
	...
}
```
#### 2) Pawn 불러오기
```cpp
APawn* ControllingPawn = OwnerComp.GetAIOwner()->GetPawn();
if (nullptr == ControllingPawn)
{
	return EBTNodeResult::Failed;
}
```
#### 3) Navigation 불러오기
```cpp
UNavigationSystemV1* NavSystem = UNavigationSystemV1::GetNavigationSystem(ControllingPawn->GetWorld());
if (nullptr == NavSystem)
{
	return EBTNodeResult::Failed;
}
```
#### 4) Task 기능 구현(ex. 정찰 위치 정하기)
```cpp
FVector Origin = OwnerComp.GetBlackboardComponent()->GetValueAsVector(BBKEY_HOMEPOS);
float PatrolRadius = AIPawn->GetAIPatrolRadius();
FNavLocation NextPatrolPos;

if (NavSystem->GetRandomPointInNavigableRadius(Origin, PatrolRadius, NextPatrolPos))
{
	OwnerComp.GetBlackboardComponent()->SetValueAsVector(BBKEY_PATROLPOS, NextPatrolPos.Location);
	return EBTNodeResult::Succeeded;
}
```
#### 6) BT에서 추가
![[Pasted image 20230627170705.png]]
Behavior Tree로 이동하면 만든 BTTask가 표시됨

> [!question] return 값
> EBTNodeResult::Failed : 실패했을 때 (비정상적인 종료)
> EBTNodeResult::Succeeded : 성공했을 때
> EBTNodeResult::InProgress : 진행중일 때(종료처리를 따로 주어야 함)
```cpp
FAICharacterAttackFinished OnAttackFinished; //Unreal Delegate
OnAttackFinished.BindLambda( //Delegate에 Lambda로 묶어서 작업 가능
	[&]()
	{
		//EBTNodeResult::InProgress 일 때 FinishLatentTask를 호출해서
		//종료조건을 전달해줘야 Task가 정상종료됨
		FinishLatentTask(OwnerComp, EBTNodeResult::Succeeded);
	}
);
```
   
   
## 4. BTService Class 생성
---
### 4-1. BTService를 상속받아 Class 생성
![[Pasted image 20230627171358.png]]
   
### 4-2. 생성자 정의
```cpp
UBTService_Detect::UBTService_Detect()
{
	NodeName = TEXT("Detect"); //Behavior Tree에서 표시될 노드 이름
	Interval = 1.0f; //TickNode가 실행될 Interval
}
```
   
### 4-3. TickNode 정의
> [!question] BTService에서는 TickNode 함수가 셋팅된 interval로 주기적으로 호출됨

```cpp
virtual void TickNode(UBehaviorTreeComponent& OwnerComp, uint8* NodeMemory, float DeltaSeconds) override;
```

```cpp
void UBTService_Detect::TickNode(UBehaviorTreeComponent& OwnerComp, uint8* NodeMemory, float DeltaSeconds)
{
	Super::TickNode(OwnerComp, NodeMemory, DeltaSeconds);

	//Pawn 가져오기
	APawn* ControllingPawn = OwnerComp.GetAIOwner()->GetPawn();
	if (nullptr == ControllingPawn)
	{
		return;
	}

	//World 가져오기
	FVector Center = ControllingPawn->GetActorLocation();
	UWorld* World = ControllingPawn->GetWorld();
	if (nullptr == World)
	{
		return;
	}

	//Custom Interface Cast
	IABCharacterAIInterface* AIPawn = Cast<IABCharacterAIInterface>(ControllingPawn);
	if (nullptr == AIPawn)
	{
		return;
	}

	float DetectRadius = AIPawn->GetAIDetectRange();

	//배열형태로 충돌한 Object들 가져오기
	TArray<FOverlapResult> OverlapResults;
	FCollisionQueryParams CollisionQueryParam(SCENE_QUERY_STAT(Detect), false, ControllingPawn);
	bool bResult = World->OverlapMultiByChannel( //여러 물체 검사
		OverlapResults, //배열
		Center, //이 위치 기준으로 검사
		FQuat::Identity,
		CCHANNEL_ABACTION, //Custom 채널
		FCollisionShape::MakeSphere(DetectRadius), //구체생성
		CollisionQueryParam //자신의 충돌제외
	);

	if (bResult)
	{
		//감지된 모든 pawn에 대해서 검사
		for (auto const& OverlapResult : OverlapResults)
		{
			APawn* Pawn = Cast<APawn>(OverlapResult.GetActor());
			if (Pawn && Pawn->GetController()->IsPlayerController()) //pawn을 조종하는 것이 Player일 때만
			{
				OwnerComp.GetBlackboardComponent()->SetValueAsObject(BBKEY_TARGET, Pawn); //타겟지정
				DrawDebugSphere(World, Center, DetectRadius, 16, FColor::Green, false, 0.2f); // 녹색으로 해당 영역 표시

				DrawDebugPoint(World, Pawn->GetActorLocation(), 10.0f, FColor::Green, false, 0.2f); //플레이어 위치 점으로 표시
				DrawDebugLine(World, ControllingPawn->GetActorLocation(), Pawn->GetActorLocation(), FColor::Green, false, 0.27f); //적과 플레이어 위치 선으로 표시
				return;
			}
		}
	}

	//찾지못한 경우
	OwnerComp.GetBlackboardComponent()->SetValueAsObject(BBKEY_TARGET, nullptr);
	DrawDebugSphere(World, Center, DetectRadius, 16, FColor::Red, false, 0.2f);
}
```
   
### 4-4. 정의한 Service Composite 에 추가
![[Pasted image 20230627174334.png]]
![[Pasted image 20230627174321.png]]
설정할 Composite 우클릭 > Add Service 에서 추가 가능
   
   
## 5. Default Decorator 설정
---
### 5-1. Decorator 추가
![[Pasted image 20230627175250.png]]
Add Decorator > Blackboard 로 언리얼에서 기본 제공해주는 Decorator 추가가능
   
### 5-2. 설정
![[Pasted image 20230627181014.png]]
Details 탭에서 각종 설정을 할 수 있다.
* Key Query : Is Set(값이 있으면 실행) / Is Not Set(값이 없으면 실행)
* Blackboard Key : 원하는 변수 설정
* Node Name : Node의 이름 변경
   
#### Abort 설정
1. Notify Observer 
	On Value Change : 값이 변경되면 인터럽트
2. Observer aborts
	Self : 현재 진행되는 Task 모두 정지
	Both : 진행되는 Task 모두 정지
   
   
## 6. BTDecorator Class 생성
---
### 6-1. BTDecorator 상속받는 Class 생성
![[Pasted image 20230627181117.png]]
   
### 6-2. Decorator의 Condition 설정
![[Pasted image 20230627181816.png]]
Inverse Condition 에 체크하면 False일 때 실행
   
### 6-3. CalculateRawConditionValue 재정의
```cpp
virtual bool CalculateRawConditionValue(UBehaviorTreeComponent& OwnerComp, uint8* NodeMemory) const override;
```

```cpp
bool UBTDecorator_AttackInRange::CalculateRawConditionValue(UBehaviorTreeComponent& OwnerComp, uint8* NodeMemory) const
{
	bool bResult = Super::CalculateRawConditionValue(OwnerComp, NodeMemory);
	...
	//GetValueAsObject : Blackboard object 가져오는 함수
	APawn* Target = Cast<APawn>(OwnerComp.GetBlackboardComponent()->GetValueAsObject(BBKEY_TARGET));
	if (nullptr == Target)
	{
		return false;
	}
	...
	return bResult;
}
```
   
   
## 7. Parallel 사용
---
병렬적으로 노드들을 동시에 실행시킬 수 있는 Composite이다.
![[Pasted image 20230627203114.png]]
Composites > Simple Parallel 로 생성가능
![[Pasted image 20230627203029.png]]
몬스터가 플레이어를 향해 공격하는 것과 플레이어를 쳐다볼 수 있게 돌리는 것을 동시에 실행

> [!tip] 캐릭터를 천천히 틱마다 움직이는 기능 참고
```cpp
float TurnSpeed = AIPawn->GetAITurnSpeed();	//Rotation speed
FVector LookVector = TargetPawn->GetActorLocation() - ControllingPawn->GetActorLocation(); //Look 방향 구하기
LookVector.Z = 0.0f; //Z값은 0으로 고정
FRotator TargetRot = FRotationMatrix::MakeFromX(LookVector).Rotator(); //매트릭스에서 회전값을 가져옴
//천천히 돌아가는 함수
ControllingPawn->SetActorRotation(FMath::RInterpTo(ControllingPawn->GetActorRotation(), TargetRot, GetWorld()->GetDeltaSeconds(), TurnSpeed));
```
