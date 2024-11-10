---
Index:
  - "[[4. UnrealIndex]]"
tags:
  - unreal
  - Study
---
## 1. Listen Server에서의 Network Role
---
Listen Server 형태로 Client가 2개가 들어왔다고 했을 때 각자가 바라보는 Role은 다음과 같다

|   Server > Client를 볼때   | ![[Pasted image 20240208155033.png]] |
|:--------------------------:| ------------------------------------ |
| **Client > Client를 볼때** | ![[Pasted image 20240208155045.png]] |
   
### Local Role
|  | Server | Client1 | Client2 |
| ---- | ---- | ---- | ---- |
| **Server** | Authority | Simulated Proxy | Simulated Proxy |
| **Client1** | Authority | Autonomouse Proxy | Simulated Proxy |
| **Client2** | Authority | Simulated Proxy | Autonomouse Proxy |
   
### Remote Role
|  | Server | Client1 | Client2 |
| ---- | ---- | ---- | ---- |
| **Server** | Autonomouse Proxy | Authority | Authority |
| **Client1** | Simulated Proxy | Authority | Authority |
| **Client2** | Simulated Proxy | Authority | Authority |
  

