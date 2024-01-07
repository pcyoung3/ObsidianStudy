# Index : [[PSIndex]]
## Tag : #Study #PS
---

> [!note] 다차원 배열에서 각 칸을 방문할 때 깊이를 우선으로 방문하는 알고리즘

## 설명
---

| ![[Pasted image 20230609110608.png]] | ![[Pasted image 20230609110732.png]]                              |
|:------------------------------------:|:-----------------------------:|
|                 시작                 | 스택에 칸을 넣고 방문표시남김 |

| ![[Pasted image 20230609110839.png]] | ![[Pasted image 20230609110901.png]]                |
|:------------------------------------:|:---------------:|
|   Stack에 Pop을 하고 주변 칸 방문    | 방문표시 후 스택에 집어넣음 |


| ![[Pasted image 20230609111019.png]] | ![[Pasted image 20230609111033.png]]          |
|:------------------------------------:|:---------:|
|      스택에서 빼고 주변칸 방문       | 계속 반복 |

> [!success] 정리
> ![[Pasted image 20230609105934.png]]

> [!tip] BFS와는 방문순서의 차이가 존재
> ![[Pasted image 20230609112000.png]]

> [!warning] DFS와 BFS의 차이점
> DFS는 거리순으로 방문이 되지 않기 때문에 거리를 재는데 사용할 수 없다.
> 하지만 나중에 그래프, 트리를 배울 때 사용하게 됨
   
   
## 예시코드
---
```cpp
#include <bits/stdc++.h>
using namespace std;
#define X first
#define Y second // pair에서 first, second를 줄여서 쓰기 위해서 사용
int board[502][502] =
{{1,1,1,0,1,0,0,0,0,0},
 {1,0,0,0,1,0,0,0,0,0},
 {1,1,1,0,1,0,0,0,0,0},
 {1,1,0,0,1,0,0,0,0,0},
 {0,1,0,0,0,0,0,0,0,0},
 {0,0,0,0,0,0,0,0,0,0},
 {0,0,0,0,0,0,0,0,0,0} }; // 1이 파란 칸, 0이 빨간 칸에 대응
bool vis[502][502]; // 해당 칸을 방문했는지 여부를 저장
int n = 7, m = 10; // n = 행의 수, m = 열의 수
int dx[4] = {1,0,-1,0};
int dy[4] = {0,1,0,-1}; // 상하좌우 네 방향을 의미
int main(void){
  ios::sync_with_stdio(0);
  cin.tie(0);
  stack<pair<int,int> > S;
  vis[0][0] = 1; // (0, 0)을 방문했다고 명시
  S.push({0,0}); // 스택에 시작점인 (0, 0)을 삽입.
  while(!S.empty()){
    pair<int,int> cur = S.top(); S.pop();
    cout << '(' << cur.X << ", " << cur.Y << ") -> ";
    for(int dir = 0; dir < 4; dir++){ // 상하좌우 칸을 살펴볼 것이다.
      int nx = cur.X + dx[dir];
      int ny = cur.Y + dy[dir]; // nx, ny에 dir에서 정한 방향의 인접한 칸의 좌표가 들어감
      if(nx < 0 || nx >= n || ny < 0 || ny >= m) continue; // 범위 밖일 경우 넘어감
      if(vis[nx][ny] || board[nx][ny] != 1) continue; // 이미 방문한 칸이거나 파란 칸이 아닐 경우
      vis[nx][ny] = 1; // (nx, ny)를 방문했다고 명시
      S.push({nx,ny});
    }
  }
}
```
기본적인 틀은 BFS와 같다. 스택에 담는 것만 차이