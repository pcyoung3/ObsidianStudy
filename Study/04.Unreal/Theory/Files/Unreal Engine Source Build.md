---
Index:
  - "[[4. UnrealIndex]]"
tags:
  - unreal
  - Study
---
## 1. 언리얼 엔진 소스 풀 빌드 전체과정
---
> [!note] 전체과정 참고
> https://dev.epicgames.com/documentation/ko-kr/unreal-engine/downloading-unreal-engine-source-code?application_version=5.2#downloadingthesourcecode

1. Unreal Git Repo에서 권한을 받아야 함
	* https://www.unrealengine.com/ko/ue-on-github 
2. Git 설치 필요
3. Visual Studio 설치 필요
4. Git clone으로 depth를 1로 일부커밋만 받는 옵션을 줘서 Branch만 따로 받음
	* 이 때 받을 버전은 release가 된 Tag를 찾아서 받도록 하자(빌드가 안되는 branch나 commit이 많다) 
```
git clone --depth 1 https://github.com/EpicGames/UnrealEngine.git -b 5.2.1-release
```

5. Setup.bat 파일 실행
6. GenerateProjectFiles.bat 파일 실행
7. Visual studio 프로젝트 파일인 sln이 나오면 풀빌드 시작
	![[Pasted image 20240407134351.png]]
	이 때 Development Editor, Win64로 해야 됨
8. UE5 프로젝트를 Set as Startup Project로 지정 후 F5로 하거나 Debug > Start new Instance로 실행
	![[Pasted image 20240407155702.png]]
   
   
## 2. 언리얼 소스 빌드 후 프로젝트 생성 및 관리
---
빌드한 언리얼엔진으로 만든 프로젝트는 내부에 언리얼 엔진 소스가 있는데 해당 엔진 소스를 수정할 수 있다.
따로 설정할 필요는 없고 자동으로 연동된다

> [!faq] 에픽런쳐로 실행시킬 경우 읽기전용이라고 표시되면서 소스가 수정되지 않음

> [!example] 프로젝트 예시
> ![[스크린샷 2024-04-07 204600.png]]
> 1. UnrealEngine을 받고 해당 엔진소스를 풀빌드한  /UnrealEngine 폴더, UE5.sln
> 2. 빌드한 엔진으로 만든 프로젝트가 있는 /P1 폴더, P1.sln
>    
> 가 있을때 P1.sln 를 열면 엔진 프로젝트와 소스를 볼 수 있는데 해당 소스를 수정한 뒤 빌드를 하게 되면
> P1.sln에서 수정한 코드가 UE5.sln에서도 자동으로 변경된다
> 
> |  P1.sln에서 변경 후 UE5.sln도 변경된 모습  |
> | :-----------------------------: |
> | ![[스크린샷 2024-04-07 203302.png]] |
>    
> 
> 
> |        언리얼 엔진 소스가 변경된 모습        |
> | :-----------------------------: |
> | ![[스크린샷 2024-04-07 203325.png]] |

