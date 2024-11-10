---
Index: "[[4. UnrealIndex]]"
tags:
  - unreal
  - Study
---
   

> [!INFO] 기본 구조

### 1. Free 
	Grab component가 있으면 해당 물체를 그대로 집음
### 2. Obj to Grab
	Grab Component 기준으로 손에 물체를 전송
### 3. Grab to Obj
	손을 Grab Component가 있는 곳에 전송

   
   
> [!INFO] Comepont 연산

|                 Rotation                     | Location    | 
| :------------------------------------: | :---: |
| ![[Pasted image 20230425215029.png]] | ![[Pasted image 20230425215120.png]]    |
   
   

> [!INFO] 연산해설

| rotation 참고 | Location 참고 |
|:-------------:|:-------------:|
|   ![[KakaoTalk_20230425_221934490.jpg]]            |      ![[KakaoTalk_20230425_214512084.jpg]]         |

Rotation, Location 모두 
1. 기존 Object(여기서는 Gun)의 Transform
2. Grab Componet가 가지고 있는 Transform
이 다르기 때문에 해당 연산을 통해서 역산하여 손에 붙이는 것이라고 이해하면 됨



