---
Index: "[[4. UnrealIndex]]"
tags:
  - work
  - unreal
  - Study
  - cpp
---
   
> [!note] UObject를 생성할 때 생성되는 파일들과 문구에 관한 문서
   
## 1. UObject란?
---
Unreal Object라는 의미로 Unreal c++에서 사용되는 객체들은 해당 클래스를 상속한다

UObject를 따로 언리얼에서 만든 이유?
> 1. C++의 사용이유 : 성능
> 
> 2. C++만으로는 모던객체지향의 메타에 맞지 않음
> 	-> c++ 에 모던객체지향 설계를 합침(UObject)
> 
> 즉, 성능과 구현의 안정성 전부를 추구한 것

UObject를 상속하면서 생기는 기능은 다음과 같다.
1.  **CDO(Class Default Object)** : 객체의 초기 값을 자체적으로 관리
2.  **Reflection** : 객체 정보를 런타임에서 실시간 조회가 가능
3.  **GC(Garbage Collection)** : 참조되지 않는 객체를 메모리에서 자동 해제할 수 있음
4.  **Serialization** : 객체와 속성 정보를 통으로 안전하게 보관하고 로딩
5.  **Delegate** : 함수를 묶어서 효과적으로 관리하고 호출할 수 있음
6.  **Replication** : 네트워크 상에서 객체간에 동기화를 시킬 수 있음
7.  **Editor Integration** : 언리얼 에디터 인터페이스를 통해 값을 편집할 수 있음
   

> [!example] 참고) 오브젝트 명명규칙

![[unreal_naming_rule_01.png]]
언리얼 오브젝트는 U로 시작
일반 오브젝트는 F로 시작한다.(FString)
   

## 2. UObject 코드
---
```cpp
#pragma once

#include "CoreMinimal.h"
#include "UObject/NoExportTypes.h"
#include "MyObject.generated.h"

UCLASS()
class UNREALOBJECT_API UMyObject : public UObject
{
	GENERATED_BODY()
	
};
```
1. UCLASS : 해당 클래스가 UObject 클래스임을 명시해줌 (리플렉션관련 기능도 있는데 나중에 다시 배움)
2. UNREALOBJECT_API : "UnrealObject"라는 프로젝트 이름 뒤에 API 가 붙은 형태
	해당 클래스를 다른 모듈(dll)에서도 사용할 수 있게 한다

3. GENERATED_BODY()
```cpp
#define BODY_MACRO_COMBINE_INNER(A,B,C,D) A##B##C##D
#define GENERATED_BODY(...) BODY_MACRO_COMBINE(CURRENT_FILE_ID,_,__LINE__,_GENERATED_BODY);
#define GENERATED_USTRUCT_BODY(...) GENERATED_BODY()
```
매크로 함수로 들어가보면
File ID와 Line 수를 가지고 문장을 만들고 
##.generated.h 에 해당 구조를 만든다

## 3. \##.generated.h 헤더파일구조
---
UObject를 상속한 파일은 빌드와 동시에 generated.h 파일이 생성되는데 경로는 다음과 같다
```
[Project 폴더]\Intermediate\Build\Win64\UnrealEditor\Inc\UnrealObject\UHT
```
   
해당 헤더파일의 내용을 보면 여러 매크로의 집합인 것을 확인할 수 있다.
UObject의 기능을 사용할 수 있도록 매크로로 만들어 놓은 것
   
```cpp
#define CURRENT_FILE_ID FID_Users_pcy35_Task_UnrealProject_UnrealObject_Source_UnrealObject_MyObject_h
```
   
```cpp
#define FID_Users_pcy35_Task_UnrealProject_UnrealObject_Source_UnrealObject_MyObject_h_15_GENERATED_BODY \
PRAGMA_DISABLE_DEPRECATION_WARNINGS \
public: \
	FID_Users_pcy35_Task_UnrealProject_UnrealObject_Source_UnrealObject_MyObject_h_15_SPARSE_DATA \
	FID_Users_pcy35_Task_UnrealProject_UnrealObject_Source_UnrealObject_MyObject_h_15_RPC_WRAPPERS_NO_PURE_DECLS \
	FID_Users_pcy35_Task_UnrealProject_UnrealObject_Source_UnrealObject_MyObject_h_15_ACCESSORS \
	FID_Users_pcy35_Task_UnrealProject_UnrealObject_Source_UnrealObject_MyObject_h_15_INCLASS_NO_PURE_DECLS \
	FID_Users_pcy35_Task_UnrealProject_UnrealObject_Source_UnrealObject_MyObject_h_15_ENHANCED_CONSTRUCTORS \
```
GENERATED_BODY() 의 매크로가 실제로 헤더파일에 다음과 같이 쓰여진다
* CURRENT_FILE_ID : FID_Users_pcy35_Task_UnrealProject_UnrealObject_Source_UnrealObject_MyObject_h
* \_\_LINE__ : 15
   
라인이 들어가기 때문에 header 파일이 바뀔 때마다 새 Generated 헤더 파일이 생성된다.
> [!success] UHT(Unreal Header Tool)
> UObject에 필요한 generated.h 파일을 만들어주는 툴
   
따라서 Unreal 의 빌드 과정은 다음과 같다.
> [!tip] Unreal 빌드과정
> 1. UHT를 이용하여 매크로를 통해서 필요한 파일생성(generated.h 등)
> 2. 생성된 파일을 포함해서 프로젝트 빌드




