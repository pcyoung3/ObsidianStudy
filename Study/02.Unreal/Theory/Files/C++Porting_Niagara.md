# Index : [[UnrealIndex]]
## Tag : #work #unreal #Study 
---
   
## 1. Niagara
---
unreal 5에서 새롭게 추가된 이팩트 시스템이다.
기존 Cascade Effect를 대체
   
   
## 2. 사용법
---
### 2-1. 선언
```cpp
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Hit", meta = (AllowPrivateAccess = "true"))  
TObjectPtr<class UNiagaraSystem> HitEmitter;
```
Blueprint에서 link 시킬 수 있게 구현
   
### 2-2. 사용
```cpp
//Hit Effect  
UNiagaraFunctionLibrary::SpawnSystemAtLocation(GetWorld(), HitEmitter, LastHitInfo.Location);
```
   
   
## 참고링크
---
https://dev.epicgames.com/community/learning/tutorials/Gx5j/using-niagara-in-c