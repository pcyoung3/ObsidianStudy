# Index : [[UnrealIndex]]
## Tag : #work #unreal
---

> [!note] 언리얼에서 c++로 작업할 때 변경점 적용하는 법

## 1. Visual Studio 등의 IDE에서 빌드 후 바로 에디터 실행
에디터 자체를 빌드해서 바로 실행할 수 있다

## 2. IDE에서 변경점을 저장하고 Unreal Editor에서 Live Coding 실행
![[Pasted image 20230502213400.png]]
에디터 하단에 있는 해당 아이콘을 누르면 Live coding이 실행되면서 
저장한 c++의 변경점이 compile 이후에 Editor에 적용

> [!danger] Live Coding 할 때 주의점
> 변경점이 제대로 적용되지 않아서 에디터의 값이 날라갈 수도 있다.


   
   
---
> [!success] 참고

C++에서 변수 셋팅을 한 뒤 에디터에서 오버라이딩 하면 에디터의 값으로 덮어씌여진다.