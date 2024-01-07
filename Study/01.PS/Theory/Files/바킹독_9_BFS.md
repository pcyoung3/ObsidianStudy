# Index : [[PSIndex]]
## Tag : #Study #PS
---

> [!note] 다차원 배열에서 각 칸을 방문할 때 너비를 우선으로 방문하는 알고리즘

## 설명
---
### 파란칸을 BFS로 순회한다고 했을 때 과정

| ![[Pasted image 20230529144801.png]] | ![[Pasted image 20230529145054.png]]                                         |
| :------------------------------------: | :------------------------------------: |
| 시작                                 | 시작하는 칸을 큐에 넣고 방문 표시를 남김 |


|        ![[Pasted image 20230529145147.png]]        |     ![[Pasted image 20230529145150.png]]      |
|:--------------------------------------------------:|:---------------------------------------------:|
| 큐에서 원소를 꺼내 상하좌우로 인접한 칸에 4번 진행 | 해당칸을 이전에 방문했다면 아무것도 하지 않고,</br> 처음으로 방문했다면 방문했다는 표시를 남기고 해당 칸을 큐에 삽입 |


| ![[Pasted image 20230529145830.png]] | ![[Pasted image 20230529150039.png]]                                                     |
|:------------------------------------:|:----------------------------------------------------:|
|     큐에서 빼고 상하좌우를 확인      | 방문하지 않은 파란칸에는 방문표시를 남기고 큐에 넣음 |

> [!success] 정리
> ![[Pasted image 20230529150128.png]]
   
> [!danger] 주의
> ![[Pasted image 20230529150542.png]]
   
### 큐에 쌓이는 순서
![[Pasted image 20230529201025.png]]
queue에 쌓이는 순서는 반드시 거리순
   
   
## 예시코드
---
```cpp
#include <bits/stdc++.h>
using namespace std;

#define X first
#define Y second
int board[502][502] =
{
    {1,1,1,0,1,0,0,0,0,0},
    {1,0,0,0,1,0,0,0,0,0},
    {1,1,1,0,1,0,0,0,0,0},
    {1,1,0,0,1,0,0,0,0,0},
    {0,1,0,0,0,0,0,0,0,0},
    {0,0,0,0,0,0,0,0,0,0},
    {0,0,0,0,0,0,0,0,0,0},
};  //1이 파란칸 0이 빨간칸
bool vis[502][502]; // 해당 칸을 방문했는지 여부를 저장
int n = 7, m = 10;  //n = 행의 수, m = 열의 수
int dx[4] = { 1, 0, -1, 0 };
int dy[4] = { 0, 1, 0, -1 };    // 상하좌우 네 방향 의미

int main(void) 
{
    ios::sync_with_stdio(0);
    cin.tie(0);

    queue<pair<int, int>> Q;
    vis[0][0] = 1;  // (0, 0) 을 방문했다고 명시
    Q.push({ 0, 0 });   // 큐에 시작점인 (0, 0)을 삽입

    while (!Q.empty())
    {
        pair<int, int> cur = Q.front();
        Q.pop();
        cout << '(' << cur.X << ", " << cur.Y << ") -> ";
        for (int dir = 0; dir < 4; dir++)   // 상하좌우 칸을 확인
        {
            int nx = cur.X + dx[dir];
            int ny = cur.Y + dy[dir];   //nx, ny에 dir에서 정한 방향의 인접한 칸의 좌표가 들어감
            if (nx < 0 || nx >= n || ny < 0 || ny >= m) //범위 밖일 경우 넘어감
                continue;
            if (vis[nx][ny] || board[nx][ny] != 1)  // 이미 방문한 칸이거나 파란칸이 아닌경우
                continue;
            vis[nx][ny] = 1; //(nx, ny)를 방문했다고 명시
            Q.push({ nx, ny });
        }
    }
}
```
   
### 결과
```
(0, 0) -> (1, 0) -> (0, 1) -> (2, 0) -> (0, 2) -> (3, 0) -> (2, 1) -> (3, 1) -> (2, 2) -> (4, 1) ->
```
   
해당 코드를 보지 않고 바로 짤 수 있도록 외워야 함
   
   
## 연습문제 1926
---
## 문제
https://www.acmicpc.net/problem/1926

어떤 큰 도화지에 그림이 그려져 있을 때, 그 그림의 개수와, 그 그림 중 넓이가 가장 넓은 것의 넓이를 출력하여라. 단, 그림이라는 것은 1로 연결된 것을 한 그림이라고 정의하자. 가로나 세로로 연결된 것은 연결이 된 것이고 대각선으로 연결이 된 것은 떨어진 그림이다. 그림의 넓이란 그림에 포함된 1의 개수이다.

## 입력

첫째 줄에 도화지의 세로 크기 n(1 ≤ n ≤ 500)과 가로 크기 m(1 ≤ m ≤ 500)이 차례로 주어진다. 두 번째 줄부터 n+1 줄 까지 그림의 정보가 주어진다. (단 그림의 정보는 0과 1이 공백을 두고 주어지며, 0은 색칠이 안된 부분, 1은 색칠이 된 부분을 의미한다)

## 출력

첫째 줄에는 그림의 개수, 둘째 줄에는 그 중 가장 넓은 그림의 넓이를 출력하여라. 단, 그림이 하나도 없는 경우에는 가장 넓은 그림의 넓이는 0이다.

## 예제 입력 1

```
6 5
1 1 0 1 1
0 1 1 0 0
0 0 0 0 0
1 0 1 1 1
0 0 1 1 1
0 0 1 1 1
```

## 예제 출력 1

4
9

## 정답코드
---
> [!question] 팁
> 1. 그림의 개수를 세는 방법 : 2중for문을 돌면서 BFS를 수행하고 방문하지 않은 1인지점을 체크
> 2. 그림의 넓이를 세는 방법 : queue에서 pop 할 때마다 더해주면 넓이를 셀 수 있음

```cpp
#include <bits/stdc++.h>
using namespace std;

#define X first
#define Y second
int board[502][502];
int vis[502][502];
int n = 0, m = 0;
int dx[4] = { 0, 1, 0, -1 };
int dy[4] = { 1, 0, -1, 0 };
int main(void)
{
    ios::sync_with_stdio(0);
    cin.tie(0);

    int input;
    cin >> n >> m;
    for (int i = 0; i < n; i++)
    {
        for (int j = 0; j < m; j++)
        {
            cin >> input;
            board[i][j] = input;
        }
    }
    
    int num = 0;
    int maxArea = 0;
    for (int i = 0; i < n; i++)
    {
        for (int j = 0; j < m; j++)
        {
            if (board[i][j] == 1 && vis[i][j] == 0) // i, j 가 시작점일 때
            {
                int area = 0;   //그림의 넓이
                num++;      //그림의 개수
                queue<pair<int, int>> Q;
                Q.push({ i, j });
                vis[i][j] = 1;
                while (!Q.empty())
                {
                    pair<int, int> cur = { Q.front() };
                    Q.pop();
                    area++; //큐에 들어있는 원소를 뺼 때마다 넓이가 1씩 증가
                    for (int dir = 0; dir < 4; ++dir)
                    {
                        int nx = cur.X + dx[dir];
                        int ny = cur.Y + dy[dir];
                        if (nx < 0 || nx >= n || ny < 0 || ny >= m)
                            continue;
                        if (vis[nx][ny] == 1 || board[nx][ny] == 0)
                            continue;
                        Q.push({ nx, ny });
                        vis[nx][ny] = 1;
                    }
                }

                if (maxArea < area)//최대 그림넓이 찾는 부분 
                    maxArea = area;
                //maxArea = max(maxArea, area); // 해당 코드도 가능
            }
        }
    }

    cout << num << '\n' << maxArea;
}
```
   
   
## 연습문제 2178
---
## 문제
https://www.acmicpc.net/problem/2178

N×M크기의 배열로 표현되는 미로가 있다.

|   |   |   |   |   |   |
|---|---|---|---|---|---|
|1|0|1|1|1|1|
|1|0|1|0|1|0|
|1|0|1|0|1|1|
|1|1|1|0|1|1|

미로에서 1은 이동할 수 있는 칸을 나타내고, 0은 이동할 수 없는 칸을 나타낸다. 이러한 미로가 주어졌을 때, (1, 1)에서 출발하여 (N, M)의 위치로 이동할 때 지나야 하는 최소의 칸 수를 구하는 프로그램을 작성하시오. 한 칸에서 다른 칸으로 이동할 때, 서로 인접한 칸으로만 이동할 수 있다.

위의 예에서는 15칸을 지나야 (N, M)의 위치로 이동할 수 있다. 칸을 셀 때에는 시작 위치와 도착 위치도 포함한다.

## 입력

첫째 줄에 두 정수 N, M(2 ≤ N, M ≤ 100)이 주어진다. 다음 N개의 줄에는 M개의 정수로 미로가 주어진다. 각각의 수들은 **붙어서** 입력으로 주어진다.

## 출력

첫째 줄에 지나야 하는 최소의 칸 수를 출력한다. 항상 도착위치로 이동할 수 있는 경우만 입력으로 주어진다.

## 예제 입력 1 복사

```
4 6
101111
101010
101011
111011
```

## 예제 출력 1 복사

15
   
   
## 정답코드
---
> [!question] 팁
> queue에서 Push할 때 +1 씩 시키면 시작점에서부터 거리가 얼마인지 알 수 있다

### 내가 푼 코드
```cpp
#include <bits/stdc++.h>
using namespace std;

#define X first
#define Y second

int board[105][105];
int dist[105][105]; //거리를 구하는 문제이기 때문에 vis대신 dist 사용
int dx[4] = { 0, 1, 0, -1 };
int dy[4] = { 1, 0, -1, 0 };
int main(void)
{
    ios::sync_with_stdio(0);
    cin.tie(0);

    int n, m;
    cin >> n >> m;

    string input;
    for (int i = 0; i < n; i++)
    {
        cin >> input;
        for (int j = 0; j < input.size(); ++j)
        {
            char ch = input[j];
            board[i][j] = atoi(&ch);
        }
    }

    for (int i = 0; i < 105; i++)
        fill(dist[i], dist[i] + 105, -1);

    queue<pair<int, int>> Q;
    Q.push({ 0, 0 });
    dist[0][0] = 0;

    while (!Q.empty())
    {
        pair<int, int> cur = Q.front();
        Q.pop();
        for (int dir = 0; dir < 4; ++dir)
        {
            int nx = cur.X + dx[dir];
            int ny = cur.Y + dy[dir];

            if (nx < 0 || nx >= n || ny < 0 || ny >= m)
                continue;
            if (dist[nx][ny] != -1 || board[nx][ny] != 1)
                continue;
            dist[nx][ny] = dist[cur.X][cur.Y] + 1;  // 이 부분에서 +1씩 더해주면 거리를 구할 수 있음
            Q.push({ nx, ny });
        }
    }
    cout << dist[n - 1][m - 1]+1;
}
```
   
### 참고용 코드(string 처리법)
```cpp
#include <bits/stdc++.h>
using namespace std;
#define X first
#define Y second
string board[102];
int dist[102][102];
int n,m;
int dx[4] = {1,0,-1,0};
int dy[4] = {0,1,0,-1};
int main(void)
{
  ios::sync_with_stdio(0);
  cin.tie(0);
  
  cin >> n >> m;
  for(int i = 0; i < n; i++)
    cin >> board[i];
  for(int i = 0; i < n; i++) 
	  fill(dist[i],dist[i]+m,-1);
  queue<pair<int,int> > Q;
  Q.push({0,0});
  dist[0][0] = 0;
  while(!Q.empty())
  {
    auto cur = Q.front(); Q.pop();
    for(int dir = 0; dir < 4; dir++)
    {
      int nx = cur.X + dx[dir];
      int ny = cur.Y + dy[dir];
      if(nx < 0 || nx >= n || ny < 0 || ny >= m) 
	      continue;
      if(dist[nx][ny] >= 0 || board[nx][ny] != '1') 
	      continue;
      dist[nx][ny] = dist[cur.X][cur.Y]+1;
      Q.push({nx,ny});
    }
  }
  cout << dist[n-1][m-1]+1; // 문제의 특성상 거리+1이 정답
}
```
   
   
## 연습문제 7576
---
## 문제
https://www.acmicpc.net/problem/7576

철수의 토마토 농장에서는 토마토를 보관하는 큰 창고를 가지고 있다. 토마토는 아래의 그림과 같이 격자 모양 상자의 칸에 하나씩 넣어서 창고에 보관한다. 

![](https://upload.acmicpc.net/de29c64f-dee7-4fe0-afa9-afd6fc4aad3a/-/preview/)

창고에 보관되는 토마토들 중에는 잘 익은 것도 있지만, 아직 익지 않은 토마토들도 있을 수 있다. 보관 후 하루가 지나면, 익은 토마토들의 인접한 곳에 있는 익지 않은 토마토들은 익은 토마토의 영향을 받아 익게 된다. 하나의 토마토의 인접한 곳은 왼쪽, 오른쪽, 앞, 뒤 네 방향에 있는 토마토를 의미한다. 대각선 방향에 있는 토마토들에게는 영향을 주지 못하며, 토마토가 혼자 저절로 익는 경우는 없다고 가정한다. 철수는 창고에 보관된 토마토들이 며칠이 지나면 다 익게 되는지, 그 최소 일수를 알고 싶어 한다.

토마토를 창고에 보관하는 격자모양의 상자들의 크기와 익은 토마토들과 익지 않은 토마토들의 정보가 주어졌을 때, 며칠이 지나면 토마토들이 모두 익는지, 그 최소 일수를 구하는 프로그램을 작성하라. 단, 상자의 일부 칸에는 토마토가 들어있지 않을 수도 있다.

## 입력

첫 줄에는 상자의 크기를 나타내는 두 정수 M,N이 주어진다. M은 상자의 가로 칸의 수, N은 상자의 세로 칸의 수를 나타낸다. 단, 2 ≤ M,N ≤ 1,000 이다. 둘째 줄부터는 하나의 상자에 저장된 토마토들의 정보가 주어진다. 즉, 둘째 줄부터 N개의 줄에는 상자에 담긴 토마토의 정보가 주어진다. 하나의 줄에는 상자 가로줄에 들어있는 토마토의 상태가 M개의 정수로 주어진다. 정수 1은 익은 토마토, 정수 0은 익지 않은 토마토, 정수 -1은 토마토가 들어있지 않은 칸을 나타낸다.

토마토가 하나 이상 있는 경우만 입력으로 주어진다.

## 출력

여러분은 토마토가 모두 익을 때까지의 최소 날짜를 출력해야 한다. 만약, 저장될 때부터 모든 토마토가 익어있는 상태이면 0을 출력해야 하고, 토마토가 모두 익지는 못하는 상황이면 -1을 출력해야 한다.

## 예제 입력 1 복사

```
6 4
0 0 0 0 0 0
0 0 0 0 0 0
0 0 0 0 0 0
0 0 0 0 0 1
```

## 예제 출력 1 복사

8
   
   
## 정답코드
---
> [!question] 시작점이 여러개일 수 있는 BFS
> Dist를 잘 처리한다면 시작점이 여러개 있을 경우 BFS를 시작하기 전에 미리 Queue에 넣어줘 처리하면 된다.

### 내가 푼 코드
```cpp
#include <bits/stdc++.h>
using namespace std;

#define X first
#define Y second

int board[1005][1005];
int dist[1005][1005];
int dx[4] = { 1, 0 ,-1, 0 };
int dy[4] = { 0, 1, 0, -1 };
int main(void)
{
    ios::sync_with_stdio(0);
    cin.tie(0);

    int n, m;
    cin >> m >> n;

    
    for (int i = 0; i < n; i++)
        fill(dist[i], dist[i] + m, -1);     //배열 -1로 초기화

    queue<pair<int, int>> Q;
    for (int i = 0; i < n; ++i)
    {
        for (int j = 0; j < m; ++j)
        {
            int input;
            cin >> input;
            board[i][j] = input;

            if (input == 1)
            {
                Q.push({ i, j });   //미리 시작점을 큐에 담아놓는다
                dist[i][j] = 0;     //시작점에서는 dist를 0으로 셋팅
                /*dist 배열 : 초기화 -1, 시작 0, 방문하지 않은 곳 -1*/
            }
        }
    }

    int mx = 0;
    while (!Q.empty())
    {
        pair<int, int> cur = Q.front();
        Q.pop();
        for (int dir = 0; dir < 4; ++dir)
        {
            int nx = cur.X + dx[dir];
            int ny = cur.Y + dy[dir];

            if (nx < 0 || nx >= n || ny < 0 || ny >= m)
                continue;
            if (dist[nx][ny] != -1 || board[nx][ny] == -1)
                continue;
            dist[nx][ny] = dist[cur.X][cur.Y] + 1;
            Q.push({ nx, ny });
            mx = max(mx, dist[nx][ny]);
        }
    }

    for (int i = 0; i < n; i++) //거리중에서 가장 큰 값을 찾아냄
    {
        for (int j = 0; j < m; j++)
        {
            if (dist[i][j] == -1 && board[i][j] != -1)  //순회하지 못한 토마토가 있을 경우
            {
                cout << -1;
                return 0;
            }
        }
    }
    cout << mx;
}
```
   
### 참고용코드(dist 배열 초기화)
```cpp
#include <bits/stdc++.h>
using namespace std;

#define X first
#define Y second

int board[1002][1002];
int dist[1002][1002];
int n, m;
int dx[4] = { 1,0,-1,0 };
int dy[4] = { 0,1,0,-1 };

int main(void) 
{
	ios::sync_with_stdio(0);
	cin.tie(0);
	cin >> m >> n;

	queue<pair<int, int> > Q;
	for (int i = 0; i < n; i++)
	{
		for (int j = 0; j < m; j++) 
		{
			cin >> board[i][j];

			if (board[i][j] == 1)
				Q.push({ i,j });
			if (board[i][j] == 0)/*dist 배열 : 초기화 0, 시작 0, 방문하지 않은 곳 -1*/
				dist[i][j] = -1;
		}
	}

	while (!Q.empty()) 
	{
		auto cur = Q.front(); 
		Q.pop();
		for (int dir = 0; dir < 4; dir++) 
		{
			int nx = cur.X + dx[dir];
			int ny = cur.Y + dy[dir];
			if (nx < 0 || nx >= n || ny < 0 || ny >= m) 
				continue;
			if (dist[nx][ny] >= 0) 
				continue;
			dist[nx][ny] = dist[cur.X][cur.Y] + 1;
			Q.push({ nx,ny });
		}
	}

	int ans = 0;
	for (int i = 0; i < n; i++) 
	{
		for (int j = 0; j < m; j++) 
		{
			if (dist[i][j] == -1) 
			{
				cout << -1;
				return 0;
			}
			ans = max(ans, dist[i][j]);
		}
	}

	cout << ans;
}
```
   
   
## 연습문제 4179
---
## 문제
https://www.acmicpc.net/problem/4179

지훈이는 미로에서 일을 한다. 지훈이를 미로에서 탈출하도록 도와주자!

미로에서의 지훈이의 위치와 불이 붙은 위치를 감안해서 지훈이가 불에 타기전에 탈출할 수 있는지의 여부, 그리고 얼마나 빨리 탈출할 수 있는지를 결정해야한다.

지훈이와 불은 매 분마다 한칸씩 수평또는 수직으로(비스듬하게 이동하지 않는다) 이동한다.

불은 각 지점에서 네 방향으로 확산된다.

지훈이는 미로의 가장자리에 접한 공간에서 탈출할 수 있다.

지훈이와 불은 벽이 있는 공간은 통과하지 못한다.

## 입력

입력의 첫째 줄에는 공백으로 구분된 두 정수 R과 C가 주어진다. 단, 1 ≤ R, C ≤ 1000 이다. R은 미로 행의 개수, C는 열의 개수이다.

다음 입력으로 R줄동안 각각의 미로 행이 주어진다.

각각의 문자들은 다음을 뜻한다.

- #: 벽
- .: 지나갈 수 있는 공간
- J: 지훈이의 미로에서의 초기위치 (지나갈 수 있는 공간)
- F: 불이 난 공간

J는 입력에서 하나만 주어진다.

## 출력

지훈이가 불이 도달하기 전에 미로를 탈출 할 수 없는 경우 IMPOSSIBLE 을 출력한다.

지훈이가 미로를 탈출할 수 있는 경우에는 가장 빠른 탈출시간을 출력한다.

## 예제 입력 1 복사

```
4 4
####
#JF#
#..#
#..#
```


## 예제 출력 1 복사
3

## 정답코드
---
> [!question] 시작점이 2종류일 때
> 시작점이 2종류일때는 BFS를 2번 돌리면 된다.


### 반례들
```
//일반적인 상황
6 5
#####
#J#F#
#...#
#...#
#...#
#..##

//불가능한 상황
6 5
#####
#JF.#
#...#
#...#
#....
#####

//불가능한 상황 2
6 5
#####
#J.F#
#...#
#...#
#....
##.##

//시작하자마자 탈출
6 5
#J###
#.#F#
#...#
#...#
#...#
#####

//불과 분리되어있는 상황
6 5
#####
#J#F#
#.#.#
#.#.#
#.#.#
#..##

```
   
   
```cpp
#include <bits/stdc++.h>
using namespace std;

#define X first
#define Y second

char board[1005][1005];
int fire[1005][1005];   //불의 전파 시간
int dist[1005][1005];   //지훈이의 이동시간
int dx[4] = { 1, 0 , -1, 0 };
int dy[4] = { 0, 1, 0, -1 };

int main(void)
{
    ios::sync_with_stdio(0);
    cin.tie(0);

    int n, m;
    cin >> n >> m;
    
    queue<pair<int, int>> fireQ;    //불
    queue<pair<int, int>> Q;    //지훈

    for (int i = 0; i < n; i++) //거리 배열 -1로 초기화
    {
        fill(fire[i], fire[i] + m, -1);
        fill(dist[i], dist[i] + m, -1);
    }

    // 시작지점에 미리 표시
    for (int i = 0; i < n; i++)
    {
        for (int j = 0; j < m; j++)
        {
            cin >> board[i][j]; // cin >> board[i];  // String 이라서 for문 하나쓰고 한줄로 받아도 됨
            if (board[i][j] == 'F')
            {
                fireQ.push({ i, j });
                fire[i][j] = 0;
            }
            if (board[i][j] == 'J')
            { 
                Q.push({ i, j });
                dist[i][j] = 0;
            }
                
        }
    }

    //불의 BFS - 일반적인 BFS
    while (!fireQ.empty())
    {
        auto cur = fireQ.front();
        fireQ.pop();
        for (int dir = 0; dir < 4; ++dir)
        {
            int nx = cur.X + dx[dir];
            int ny = cur.Y + dy[dir];
            if (nx < 0 || nx >= n || ny < 0 || ny >= m)
                continue;
            if (fire[nx][ny] != -1 || board[nx][ny] == '#')
                continue;
            fire[nx][ny] = fire[cur.X][cur.Y] + 1;
            fireQ.push({ nx, ny });
        }
    }

    //지훈이의 BFS
    while (!Q.empty())
    {
        auto cur = Q.front();
        Q.pop();
        for (int dir = 0; dir < 4; ++dir)
        {
            int nx = cur.X + dx[dir];
            int ny = cur.Y + dy[dir];
            
            //cur 의 주변 타일을 검사했을 때 경계선을 넘었다면 현재 위치는 배열의 끝자락이니 탈출 성공
            if (nx < 0 || nx >= n || ny < 0 || ny >= m)
            {
                cout << dist[cur.X][cur.Y] + 1;
                return 0;
            }            
            if (dist[nx][ny] != -1 || board[nx][ny] == '#' || board[nx][ny] == 'F')
                continue;
            //불이 이미 지나갔다면 갈 수 없음
            //불이 붙지 않은 타일은 -1인데 이거랑 대소비교를 하면 안되서 예외처리
            if (dist[cur.X][cur.Y] + 1 >= fire[nx][ny] && fire[nx][ny] != -1)
                continue;
            dist[nx][ny] = dist[cur.X][cur.Y] + 1;
            Q.push({ nx, ny });
        }
    }

    //전수조사를 해도 경계선에 다다르지 못했으므로 실패
    cout << "IMPOSSIBLE";
}
```
   
> [!warning] 주의
> 문제에서는 불의 진행경로가 지훈이의 진행경로에는 영향을 끼쳤지만 그 반대상황에는 끼치지 않았다. 이처럼 어느 하나가 독립적이지 않고 서로에게 영향을 끼친다면 백트레킹 기법이 필요
   
   
## 연습문제 1697
---
## 문제

수빈이는 동생과 숨바꼭질을 하고 있다. 수빈이는 현재 점 N(0 ≤ N ≤ 100,000)에 있고, 동생은 점 K(0 ≤ K ≤ 100,000)에 있다. 수빈이는 걷거나 순간이동을 할 수 있다. 만약, 수빈이의 위치가 X일 때 걷는다면 1초 후에 X-1 또는 X+1로 이동하게 된다. 순간이동을 하는 경우에는 1초 후에 2\*X의 위치로 이동하게 된다.

수빈이와 동생의 위치가 주어졌을 때, 수빈이가 동생을 찾을 수 있는 가장 빠른 시간이 몇 초 후인지 구하는 프로그램을 작성하시오.

## 입력

첫 번째 줄에 수빈이가 있는 위치 N과 동생이 있는 위치 K가 주어진다. N과 K는 정수이다.

## 출력

수빈이가 동생을 찾는 가장 빠른 시간을 출력한다.

## 예제 입력 1 복사
```
5 17
```

## 예제 출력 1 복사

4
   
## 정답코드
---
> [!question] 1차원 BFS
> BFS처럼 보이지 않지만 BFS 문제이다
   
### 내가 푼 코드
```cpp
#include <bits/stdc++.h>
using namespace std;

int board[100005];  //1차원 배열로 충분
int dist[100005];
int dx[3] = { -1, 1, 2 };   // -1, +1, *2
int main(void)
{
    ios::sync_with_stdio(0);
    cin.tie(0);

    int n, k;
    cin >> n >> k;

    fill(dist, dist + 100004, -1);

    queue<int> Q;
    Q.push(n);
    dist[n] = 0;
    while (!Q.empty())
    {
        int cur = Q.front();
        Q.pop();

        if (cur == k)   // 목적지에 도착했을 경우
        {
            cout << dist[cur];
            return 0;
        }
        for (int dir = 0; dir < 3; ++dir)
        {
            int nx;
            if (dir == 2)
                nx = cur * dx[dir];
            else
                nx = cur + dx[dir];

            if (nx < 0 || nx > 100000)
                continue;
            if (dist[nx] != -1)
                continue;
            Q.push(nx);
            dist[nx] = dist[cur] + 1;
        }
    }
}
```
   
### 참고코드
```cpp
#include <bits/stdc++.h>
using namespace std;
#define X first
#define Y second
int dist[100002];
int n,k;
int main(void){
  ios::sync_with_stdio(0);
  cin.tie(0);
  cin >> n >> k;
  fill(dist, dist+100001,-1);
  dist[n] = 0;
  queue<int> Q;
  Q.push(n);
  while(dist[k] == -1){
    int cur = Q.front(); Q.pop();
    for(int nxt : {cur-1, cur+1, 2*cur}){  //범위기반 for문을 다음과 같이 사용할 수도 있으니 참고
      if(nxt < 0 || nxt > 100000) continue;
      if(dist[nxt] != -1) continue;
      dist[nxt] = dist[cur]+1;
      Q.push(nxt);
    }        
  }
  cout << dist[k];
}
```