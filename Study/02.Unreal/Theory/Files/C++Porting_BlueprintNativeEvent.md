# Index : [[UnrealIndex]]
## Tag : #work #unreal #Study 
---
   
## 1. BlueprintNativeEvent
---
> [!question] UFUNCTION 에 들어가는 옵션
> 1. BlueprintCallable : Blueprint에서 호출 가능
> 2. BlueprintImplementableEvent : 선언은 .h에서 하지만 구현은 Blueprint에서 가능
> 3. BlueprintNativeEvent : Blueprint에서 해당 함수를 overriding 할 수 있음

기본적인 정의는 다음과 같지만
interface에서 overriding 한 함수를 blueprint에서 호출하거나 overriding 하려면
무조건 BlueprintNativeEvent를 이용해야 함

그렇지 않으면 다음과 같은 상황이 발생
```cpp
//interface
UFUNCTION(BlueprintCallable, BlueprintNativeEvent)  
void ContinueAttack();  //정상상황

virtual void TestFunc() = 0;    //UFUNCTION(BlueprintCallable)를 붙일 경우 에러발생

////////////////////////////////////////////////////////////
//interface 상속한 class
UFUNCTION(BlueprintCallable, BlueprintNativeEvent)  
void ContinueAttack();  //정상상황
virtual void ContinueAttack_Implementation() override;  
  
UFUNCTION(BlueprintCallable)  
virtual void TestFunc() override; //이상상황

```

![[스크린샷 2023-08-26 215435.png]]
다음과 같이 BlueprintNativeEvent를 사용한 함수는 `Target is Combat Interface` 라고 표시되는데
TestFunc는 interface로 표시되지않고 호출도 이상하게 작동한다

즉, interface를 이용해서 blueprint에서 호출시키고 싶으면
UFUNCTION(BlueprintCallable, BlueprintNativeEvent) 를 사용해야 함
   
   

## 2. 사용법
---
### 2-1. Interface에서 선언
```cpp
class D1_API ICombatInterface  
{  
    GENERATED_BODY()  
  
    // Add interface functions to this class. This is the class that will be inherited to implement this interface.  
public:  
    UFUNCTION(BlueprintCallable, BlueprintNativeEvent)  
    void ContinueAttack();
    
    ...
}
```
기존 Interface와 다르게 virtual 키워드와 pure 키워드(=0)를 붙이지 않는다
그런 키워드는 generated.h 파일에서 붙여준 상태로 생성됨
   
### 2-2.  상속받는 class 부분
```cpp
UCLASS(config=Game)  
class ACombatCharacter : public ACharacter, public ICombatInterface  
{  
    GENERATED_BODY()
    ...
    
    public:  
    UFUNCTION(BlueprintCallable, BlueprintNativeEvent)  
    void ContinueAttack();  //이 함수는 여기다가 선언만 해놓고 구현부의 구현이 없다
    virtual void ContinueAttack_Implementation() override; //이 부분을 실제로 구현해주면 됨
    
    ...
}
////////////////////////////////////////////////////////////

void ACombatCharacter::ContinueAttack_Implementation() //_Implementation만 구현
{
	//Overrind 구현
}
```
실제로 구현해야 하는 함수는 \_Implementation가 붙은 함수
나머지 함수는 그냥 선언부에 선언만 해주면 된다.

이렇게 하면 blueprint에서 정상적으로 호출이 가능하다
