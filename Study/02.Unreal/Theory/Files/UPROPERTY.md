# Index : [[UnrealIndex]]
## Tag : #work #unreal
---

> [!note] C++에서 선언한 변수를 Editor에서 수정
   
### 사용방법
```cpp
UPROPERTY(EditAnywhere, Category = "Moveing Platform")
FVector PlatformVelocity = FVector(100, 0, 0);
```
해당 키워드를 통해서 Editor에 표시해줄 수 있다.
   
   
## 옵션

| 옵션            | 효과                                 |
| --------------- | ------------------------------------ |
| EditAnywhere    | 에디터에서 수정가능                  |
| VisibleAnywhere | 에디터에서 수정 불가능 값만 확인가능 |
| 추가요망                 |                                      |
