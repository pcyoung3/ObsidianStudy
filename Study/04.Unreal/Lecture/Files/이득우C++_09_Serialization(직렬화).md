---
Index: "[[4. UnrealIndex]]"
tags:
  - work
  - unreal
  - Study
  - cpp
---
   
> [!note] Serialization(직렬화)의 개념
> Stream과 비슷한 개념이라고 보면 된다. 데이터를 일렬로 정렬시켜서 변환하는 기법을 의미

## 1. Serialization 의 개념
---
![[Pasted image 20230525113543.png]]
![[Pasted image 20230525114248.png]]
   
   
## 2. Unreal의 스마트 포인터 시스템
---
![[Pasted image 20230524214312.png]]
```cpp
//Unique Pointer
TUniquePtr<FArchive> FileWriterAr = TUniquePtr<FArchive>(IFileManager::Get().CreateFileWriter(*ObjectDataAbsolutePath))

//Shared Pointer
TSharedPtr<FJsonObject> JsonObjectDest;

//Shared Reference
TSharedRef<FJsonObject> JsonObjectSrc = MakeShared<FJsonObject>();

//TSharedPtr -> TSharedRef 로 형변환
TSharedPtr<FJsonObject> JsonObjectDest;
JsonObjectDest.ToSharedRef()
```
* C++의 [[Smart Pointer]] 참고
   
   
## 3. 파일경로 함수
---
### 3-1. 현재 프로젝트 위치와 경로 String 합쳐주는 함수
```cpp
//저장할 파일 폴더 ../../../../../../Users/pcy35/Task/UnrealProject/UnrealSerialization/Saved
const FString SaveDir = FPaths::Combine(FPlatformMisc::ProjectDir(), TEXT("Saved"));

const FString RawDataFileName(TEXT("RawData.bin"));
//상대경로 : ../../../../../../Users/pcy35/Task/UnrealProject/UnrealSerialization/Saved/RawData.bin
FString RawDataAbsolutePath = FPaths::Combine(*SaveDir, *RawDataFileName);
```
   
### 3-2. 상대경로를 절대경로로 변경
```cpp
//절대경로 : C:/Users/pcy35/Task/UnrealProject/UnrealSerialization/Saved/RawData.bin
FPaths::MakeStandardFilename(RawDataAbsolutePath);
```
   
   
## 4. UStruct Serialization
---
### 4-1. Write
```cpp
//쓰기
FStudentData RawDataSrc(16, TEXT("박치영"));
//Write Archive 생성
FArchive* RawFileWriterAr = IFileManager::Get().CreateFileWriter(*RawDataAbsolutePath);
if (nullptr != RawFileWriterAr)
{
	//operator<< 연산자 오버로딩 없을 경우 나눠서 전달
	//*RawFileWriterAr << RawDataSrc.Order;
	//*RawFileWriterAr << RawDataSrc.Name;
	*RawFileWriterAr << RawDataSrc;
	RawFileWriterAr->Close();	//아카이브 닫기
	delete RawFileWriterAr;		//아카이브 포인터 삭제(UObject가 아님)
	RawFileWriterAr = nullptr;	//댕글링 포인터 대비
}
```
   
> [!tip] 구조체에 operator<<가 오버로딩 되어있지 않으면 Struct의 인자 하나하나를 정의해야함

```cpp
USTRUCT()
struct FStudentData
{
	GENERATED_BODY()
	...
	//UStruct의 각 멤버들을 옮겨주는 연산자 오버로딩
	friend FArchive& operator<<(FArchive& Ar, FStudentData& InStudentData)
	{
		Ar << InStudentData.Order;
		Ar << InStudentData.Name;
		return Ar;
	}

	int32 Order;
	FString Name;
};
```
   
### 4-2. Read
```cpp
FStudentData RawDataDest;
//Read Archive 생성
FArchive* RawFileReaderAr = IFileManager::Get().CreateFileReader(*RawDataAbsolutePath);
if (nullptr != RawFileReaderAr)
{
	*RawFileReaderAr << RawDataDest;
	RawFileReaderAr->Close();
	delete RawFileReaderAr;
	RawFileReaderAr = nullptr;

	UE_LOG(LogTemp, Log, TEXT("[RawData] 이름 %s 순번 %d"), *RawDataDest.Name, RawDataDest.Order);
}
```
   
   
## 5. UObject Serialization
---
### 5-1. Write
```cpp
TArray<uint8> BufferArray;		//Byte Stream
FMemoryWriter MemoryWriterAr(BufferArray);		//Byte를 가진 Writer Archive 생성
StudentSrc->Serialize(MemoryWriterAr);		//UObject Class에 Serialize Overriding 필요

//SmartPointer : Unique Pointer
if (TUniquePtr<FArchive> FileWriterAr = TUniquePtr<FArchive>(IFileManager::Get().CreateFileWriter(*ObjectDataAbsolutePath)))
{
	*FileWriterAr << BufferArray;
	FileWriterAr->Close();
}
``` 
   
### 5-2. Read
```cpp
TArray<uint8> BufferArrayFromFile;
if (TUniquePtr<FArchive>FileReaderAr = TUniquePtr<FArchive>(IFileManager::Get().CreateFileReader(*ObjectDataAbsolutePath)))
{
	*FileReaderAr << BufferArrayFromFile;
	FileReaderAr->Close();
}
FMemoryReader MemoryReaderAr(BufferArrayFromFile);
UStudent* StudentDest = NewObject<UStudent>();
StudentDest->Serialize(MemoryReaderAr);
```
   
### 5-3. UObject의 Serialize 함수를 사용하기 위해서 오버라이딩 필요
```cpp
virtual void Serialize(FArchive& Ar) override;
...
void UStudent::Serialize(FArchive& Ar)
{
	Super::Serialize(Ar);

	Ar << Order;
	Ar << Name;
}
```
   
   
## 6. Json Serialization
---
### 6-1. 사전준비
> [!success] Json converter를 사용하기전 사전준비가 필요
> 1. \#include "JSonObjectConverter.h" 포함
> 2. {프로젝트이름}.Build.cs에 추가적인 모듈작성을 해줘야 함 

```cpp
//cpp
#include "JSonObjectConverter.h"	//JSon Serialization을 위해 추가

//{프로젝트이름}.Build.cs
...
PublicDependencyModuleNames.AddRange(new string[] { "Core", "CoreUObject", "Engine", "InputCore", "Json", "JsonUtilities" });
...

```
   
### 6-2. Write
```cpp
//header file
UPROPERTY()
TObjectPtr<class UStudent> StudentSrc;
...

//스마트포인터인 Shared Reference 활용
TSharedRef<FJsonObject> JsonObjectSrc = MakeShared<FJsonObject>();
//FJsonObjectConverter::UStructToJsonObject : Struct > Json으로 변환
FJsonObjectConverter::UStructToJsonObject(StudentSrc->GetClass(), StudentSrc, JsonObjectSrc);

FString JsonOutString;
//TJsonWriterFactory에 의해서 Json으로 써주는 Archive가 생성
TSharedRef<TJsonWriter<TCHAR>> JsonWriterAr = TJsonWriterFactory<TCHAR>::Create(&JsonOutString);
if (FJsonSerializer::Serialize(JsonObjectSrc, JsonWriterAr))	//Serialize로 JsonWriterAr 아카이브의 JsonOutString 셋팅
{
	FFileHelper::SaveStringToFile(JsonOutString, *JsonDataAbsolutePath);	//파일에 저장
}
```
   
### 6-3. Read
```cpp
FString JsonInString;
//파일 -> String으로 불러옴
FFileHelper::LoadFileToString(JsonInString, *JsonDataAbsolutePath);

//String을 Read하는 Archive 생성
TSharedRef<TJsonReader<TCHAR>> JsonReaderAr = TJsonReaderFactory<TCHAR>::Create(JsonInString);

//Null이 들어갈 수도 있어서 TSharedPtr로 선언
TSharedPtr<FJsonObject> JsonObjectDest;
//Deserialize : 역직렬화 Json String을 Json 객체로 변경
if (FJsonSerializer::Deserialize(JsonReaderAr, JsonObjectDest))
{
	UStudent* JsonStudentDest = NewObject<UStudent>();
	//ToSharedRef : TSharedPtr->TSharedRef로 형변환
	//JsonObjectToUStruct : JsonObject -> UStruct 로 Convert
	if (FJsonObjectConverter::JsonObjectToUStruct(JsonObjectDest.ToSharedRef(), JsonStudentDest->GetClass(), JsonStudentDest))
	{
		PrintStudentInfo(JsonStudentDest, TEXT("JsonData"));
	}
}
```