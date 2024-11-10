---
Index: "[[2. PS&AlgorithmIndex]]"
tags:
  - Study
  - PS
Algorithm_type:
  - simulation
---

## 문제
https://www.acmicpc.net/problem/16985
평화롭게 문제를 경작하며 생활하는 BOJ 마을 사람들은 더 이상 2차원 미로에 흥미를 느끼지 않는다. 2차원 미로는 너무나 쉽게 탈출이 가능하기 때문이다. 미로를 이 세상 그 누구보다 사랑하는 준현이는 이런 상황을 매우 안타깝게 여겨 아주 큰 상금을 걸고 BOJ 마을 사람들의 관심을 확 끌 수 있는 3차원 미로 탈출 대회를 개최하기로 했다.

대회의 규칙은 아래와 같다.

- 5×5 크기의 판이 5개 주어진다. 이중 일부 칸은 참가자가 들어갈 수 있고 일부 칸은 참가자가 들어갈 수 없다. 그림에서 하얀 칸은 참가자가 들어갈 수 있는 칸을, 검은 칸은 참가자가 들어갈 수 없는 칸을 의미한다.
![[16985_pro_1.png]]

- 참가자는 주어진 판들을 시계 방향, 혹은 반시계 방향으로 자유롭게 회전할 수 있다. 그러나 판을 뒤집을 수는 없다.
![[16985_pro_2.png]]

- 회전을 완료한 후 참가자는 판 5개를 쌓는다. 판을 쌓는 순서는 참가자가 자유롭게 정할 수 있다. 이렇게 판 5개를 쌓아 만들어진 5×5×5 크기의 큐브가 바로 참가자를 위한 미로이다. 이 때 큐브의 입구는 정육면체에서 참가자가 임의로 선택한 꼭짓점에 위치한 칸이고 출구는 입구와 면을 공유하지 않는 꼭짓점에 위치한 칸이다.
![[16985_pro_3.png]]

- 참가자는 현재 위치한 칸에서 면으로 인접한 칸이 참가자가 들어갈 수 있는 칸인 경우 그 칸으로 이동할 수 있다.
- 참가자 중에서 본인이 설계한 미로를 가장 적은 이동 횟수로 탈출한 사람이 우승한다. 만약 미로의 입구 혹은 출구가 막혀있거나, 입구에서 출구에 도달할 수 있는 방법이 존재하지 않을 경우에는 탈출이 불가능한 것으로 간주한다.

이 대회에서 우승하기 위해서는 미로를 잘 빠져나올 수 있기 위한 담력 증진과 체력 훈련, 그리고 적절한 운이 제일 중요하지만, 가장 적은 이동 횟수로 출구에 도달할 수 있게끔 미로를 만드는 능력 또한 없어서는 안 된다. 주어진 판에서 가장 적은 이동 횟수로 출구에 도달할 수 있게끔 미로를 만들었을 때 몇 번 이동을 해야하는지 구해보자. 

## 입력

첫째 줄부터 25줄에 걸쳐 판이 주어진다. 각 판은 5줄에 걸쳐 주어지며 각 줄에는 5개의 숫자가 빈칸을 사이에 두고 주어진다. 0은 참가자가 들어갈 수 없는 칸, 1은 참가자가 들어갈 수 있는 칸을 의미한다.

## 출력

첫째 줄에 주어진 판으로 설계된 미로를 탈출하는 가장 적은 이동 횟수를 출력한다. 단, 어떻게 설계하더라도 탈출이 불가능할 경우에는 -1을 출력한다.

## 예제 입력 1 복사

```
1 1 1 1 1
0 0 0 0 0
0 0 0 0 0
0 0 0 0 0
0 0 0 0 0
1 1 1 1 1
0 0 0 0 0
0 0 0 0 0
0 0 0 0 0
0 0 0 0 0
1 1 1 1 1
0 0 0 0 0
0 0 0 0 0
0 0 0 0 0
0 0 0 0 0
1 1 1 1 1
0 0 0 0 0
0 0 0 0 0
0 0 0 0 0
0 0 0 0 0
1 1 1 1 1
0 0 0 0 0
0 0 0 0 0
0 0 0 0 0
0 0 0 0 0
```

## 예제 출력 1 복사

12

## 예제 입력 2 복사

```
1 1 1 1 1
1 0 0 0 1
1 0 0 0 1
1 0 0 0 1
1 1 1 1 1
0 0 0 0 0
0 1 1 1 0
0 1 0 1 0
0 1 1 1 0
0 0 0 0 0
0 0 0 0 0
0 0 0 0 0
0 0 1 0 0
0 0 0 0 0
0 0 0 0 0
0 0 0 0 0
0 1 1 1 0
0 1 0 1 0
0 1 1 1 0
0 0 0 0 0
1 1 1 1 1
1 0 0 0 1
1 0 0 0 1
1 0 0 0 1
1 1 1 1 1
```

## 예제 출력 2 복사

-1

## 예제 입력 3 복사

```
1 1 1 1 1
0 0 0 0 0
0 0 0 0 0
0 0 0 0 0
0 0 0 0 0
0 0 0 0 0
1 1 1 1 1
0 0 0 0 0
0 0 0 0 0
0 0 0 0 0
0 0 0 0 0
0 0 0 0 0
1 1 1 1 1
0 0 0 0 0
0 0 0 0 0
0 0 0 0 0
0 0 0 0 0
0 0 0 0 0
1 1 1 1 1
0 0 0 0 0
0 0 0 0 0
0 0 0 0 0
0 0 0 0 0
0 0 0 0 0
1 1 1 1 1
```

## 예제 출력 3 복사

12

## 예제 입력 4 복사

```
1 1 1 1 1
1 1 1 1 1
1 1 1 1 1
1 1 1 1 1
1 1 1 1 1
1 1 1 1 1
1 1 1 1 1
1 1 1 1 1
1 1 1 1 1
1 1 1 1 1
1 1 1 1 1
1 1 1 1 1
1 1 1 1 1
1 1 1 1 1
1 1 1 1 1
1 1 1 1 1
1 1 1 1 1
1 1 1 1 1
1 1 1 1 1
1 1 1 1 1
1 1 1 1 1
1 1 1 1 1
1 1 1 1 1
1 1 1 1 1
1 1 1 1 1
```

## 예제 출력 4 복사

12

## 예제 입력 5 복사

```
0 0 0 1 0
0 0 0 0 0
1 0 1 1 1
0 0 0 1 0
0 0 1 0 0
0 1 0 0 0
1 1 0 0 0
1 0 0 1 0
0 1 1 1 0
0 1 0 1 0
0 0 1 0 0
1 0 0 0 0
0 1 0 0 0
0 0 1 0 0
1 1 1 0 0
1 0 0 0 1
1 0 0 0 0
0 0 1 0 1
0 1 1 0 0
0 1 0 0 0
0 0 0 1 0
1 0 0 0 0
0 0 1 0 0
0 1 0 0 1
0 1 0 0 0
```

## 예제 출력 5 복사

22

## 예제 입력 6 복사

```
0 0 0 0 0
0 0 0 0 0
1 0 0 0 1
0 0 1 0 0
0 0 1 1 1
0 1 0 0 1
0 0 0 0 1
0 0 0 0 0
0 0 0 0 0
0 1 0 0 0
0 1 0 0 1
1 0 0 1 0
0 0 0 1 0
0 1 1 0 0
0 1 0 0 0
1 0 1 0 0
0 0 0 0 0
1 0 0 0 0
0 0 0 1 0
1 0 0 0 0
0 0 0 1 0
0 0 0 0 1
1 1 0 0 0
1 0 0 1 1
1 0 0 0 0
```

## 예제 출력 6 복사

-1

## 예제 입력 7 복사

```
1 1 0 0 0
0 0 0 0 1
0 0 1 0 0
0 0 0 0 0
0 0 0 0 0
0 0 1 1 1
1 0 0 0 0
0 0 1 0 0
0 0 1 1 1
0 0 1 0 0
0 0 0 0 0
0 0 1 0 1
0 0 0 0 0
0 0 0 1 0
0 0 1 0 1
0 0 1 0 0
1 0 0 0 0
0 0 1 1 0
1 0 1 0 0
0 0 1 0 1
0 0 1 1 0
1 1 0 1 1
0 0 0 0 1
0 1 0 1 0
0 1 0 0 0
```

## 예제 출력 7 복사

16

## 예제 입력 8 복사

```
0 0 1 0 0
0 0 0 0 0
1 1 0 0 0
0 0 1 0 0
1 1 1 0 0
0 0 0 0 1
1 0 0 0 0
0 1 0 0 1
0 0 0 0 0
0 1 0 1 0
1 0 0 0 1
1 1 1 1 1
1 1 0 0 0
0 0 0 1 0
0 0 0 1 0
0 0 0 1 1
0 0 1 0 0
0 1 1 1 0
1 0 0 0 0
0 1 1 0 1
0 1 0 0 0
0 0 0 1 0
1 0 0 0 0
0 0 0 1 0
0 0 0 1 0
```
   
---
## 풀이
> [!question] 3차원배열 + 백트레킹 + 배열 회전 + N진법 + BFS 문제
> 지금까지 배운 문제들을 한 문제로 합쳐놓은 문제
> 진법 : [[시뮬레이션_백준15683 감시]]
> 배열회전 : [[시뮬레이션_백준18808 스티커 붙이기]]
> 3차원배열 : [[BFS_백준7569 토마토]]

> [!bug] 주의할 부분
> 1. 각 회차를 진행할 때 기존 데이터를 초기화 시켜야 함
> 2. 3차원 배열 index 계산
> 3. 회전과 판의 정렬을 동시에 적용해줘야 함

```cpp
#include <bits/stdc++.h>
using namespace std;

int inputBoard[5][5][5];
int board[5][5][5];
int ans;

int dx[] = { 1, -1, 0, 0, 0, 0 };
int dy[] = { 0, 0, 1, -1, 0, 0 };
int dz[] = { 0, 0, 0, 0, 1, -1 };

void SolveMaze();
void SortBoard(int order[5]);
void RotateBoard(int height);
void BFSBoard();

//Debug
void PrintBoard();
void PrintArray(int arr[5][5][5]);

int main(void)
{
	ios::sync_with_stdio(0);
	cin.tie(0);

	//3차원 배열에 주의
	//이번에는 i,j,k형태로 z,x,y로 값을 받게 처리
	for (int i = 0; i < 5; i++)
		for (int j = 0; j < 5; j++)
			for (int k = 0; k < 5; k++)
				cin >> inputBoard[i][j][k];

	ans = 9999;
	SolveMaze();

	if (ans == 9999)
		cout << -1;
	else
		cout << ans;
}

void SolveMaze()
{
	int order[5] = { 0, 1, 2, 3, 4 };
	int maxRotate = 1;
	for (int i = 0; i < 5; i++)
		maxRotate *= 4;
	do
	{
		for (int temp = 0; temp < maxRotate; temp++)	//0 ~ 4^5까지 반복(5개의 판이 4번씩 회전)
		{
			for (int i = 0; i < 5; i++)
				for (int j = 0; j < 5; j++)
					for (int k = 0; k < 5; k++)					//초회차 및 전회차 board 초기화 작업
						board[i][j][k] = inputBoard[i][j][k];	//전회차의 회전 및 순서를 초기화

			int brute = temp;
			for (int i = 0; i < 5; i++)	//판이 5개
			{
				int dir = brute % 4;	//각각의 판이 4개의 회전방향을 가진다
				brute /= 4;				//4진법 사용

				//각 판을 회전
				for (int rotNum = 0; rotNum < dir; rotNum++)
					RotateBoard(i);
			}
			SortBoard(order);	//각 판의 순서를 바꿈
			BFSBoard();	//BFS로 현재 판들의 회전, 순서의 최단거리 구함
		}
	} while (next_permutation(order, order + 5));
}

//판을 시계방향으로 한번 회전시키는 함수
void RotateBoard(int height)
{
	int boardTemp[5][5][5];
	for (int j = 0; j < 5; j++)
		for (int k = 0; k < 5; k++)
			boardTemp[height][j][k] = board[height][5 - 1 - k][j];

	for (int j = 0; j < 5; j++)
		for (int k = 0; k < 5; k++)
			board[height][j][k] = boardTemp[height][j][k];
}

//판들의 순서를 입력받고 board 배열에 적용시키는 함수
void SortBoard(int order[5])
{
	int boardTemp[5][5][5];
	for (int i = 0; i < 5; i++)
		for (int j = 0; j < 5; j++)
			for (int k = 0; k < 5; k++)
				boardTemp[i][j][k] = board[order[i]][j][k];

	for (int i = 0; i < 5; i++)
		for (int j = 0; j < 5; j++)
			for (int k = 0; k < 5; k++)
				board[i][j][k] = boardTemp[i][j][k];
}

//최단거리를 구하는 BFS
void BFSBoard()
{
	//시작점과 끝점이 0으로 막혀있다면 BFS 수행 필요없음
	if (board[0][0][0] == 0 || board[4][4][4] == 0)
		return;
	int DistBoard[5][5][5] = { 0, };

	queue<tuple<int, int, int>> Q;
	Q.push({ 0, 0, 0 });
	DistBoard[0][0][0] = 1;

	while (!Q.empty())
	{
		int curX, curY, curZ;
		auto cur = Q.front();
		tie(curX, curY, curZ) = cur;
		Q.pop();
		for (int dir = 0; dir < 6; dir++)
		{
			int nx = curX + dx[dir];
			int ny = curY + dy[dir];
			int nz = curZ + dz[dir];

			if (nx < 0 || nx >= 5 || ny < 0 || ny >= 5 || nz < 0 || nz >= 5)
				continue;
			if (board[nx][ny][nz] != 1 || DistBoard[nx][ny][nz] != 0)
				continue;
			if (nx == 4 && ny == 4 && nz == 4)
			{
				ans = min(ans, DistBoard[curX][curY][curZ]);
				return;
			}

			DistBoard[nx][ny][nz] = DistBoard[curX][curY][curZ] + 1;
			Q.push({ nx, ny, nz });
		}
	}
}

//Debug 용
void PrintBoard()
{
	cout << "==================\n";
	for (int i = 0; i < 5; i++)
	{
		for (int j = 0; j < 5; j++)
		{
			for (int k = 0; k < 5; k++)
			{
				cout << board[i][j][k];
			}
			cout << "\n";
		}
		cout << "\n";
	}
	cout << "\n";
}

void PrintArray(int arr[5][5][5])
{
	cout << "==================\n";
	for (int i = 0; i < 5; i++)
	{
		for (int j = 0; j < 5; j++)
		{
			for (int k = 0; k < 5; k++)
			{
				cout << arr[i][j][k];
			}
			cout << "\n";
		}
		cout << "\n";
	}
	cout << "\n";
}
```

## 다른풀이
```cpp
#include <bits/stdc++.h>
using namespace std;

// board[dir][i][j][k] :  i번째 판을 시계방향으로 dir번 돌렸을 때 (j, k)의 값
int board[4][5][5][5];
int maze[5][5][5];

int dx[6] = { 1,0,0,0,0,-1 };
int dy[6] = { 0,1,-1,0,0,0 };
int dz[6] = { 0,0,0,1,-1,0 };

int solve();

int main() 
{
	ios::sync_with_stdio(0);
	cin.tie(0);
	for (int i = 0; i < 5; i++) 
	{
		for (int j = 0; j < 5; j++)
			for (int k = 0; k < 5; k++)
				cin >> board[0][i][j][k];

		for (int j = 0; j < 5; j++)
			for (int k = 0; k < 5; k++)
				board[1][i][j][k] = board[0][i][4 - k][j];

		for (int j = 0; j < 5; j++)
			for (int k = 0; k < 5; k++)
				board[2][i][j][k] = board[1][i][4 - k][j];

		for (int j = 0; j < 5; j++)
			for (int k = 0; k < 5; k++)
				board[3][i][j][k] = board[2][i][4 - k][j];
	}

	int order[5] = { 0,1,2,3,4 }; // 판을 쌓는 순서
	int ans = 9999;
	do
	{
		for (int tmp = 0; tmp < 1024; tmp++) 
		{
			int brute = tmp; // 5개의 판에 대해 dir을 정해주기 위한 변수
			for (int i = 0; i < 5; i++)
			{
				int dir = brute % 4; // brute & 3 이라고 멋있게 쓸 수도 있음
				brute /= 4; // brute >>= 2 라고 멋있게 쓸 수도 있음
				for (int j = 0; j < 5; j++)
					for (int k = 0; k < 5; k++)
						maze[i][j][k] = board[dir][order[i]][j][k];
			}
			ans = min(ans, solve());
		}
	} while (next_permutation(order, order + 5));

	if (ans == 9999) ans = -1;
	cout << ans;
}

int solve()
{
	int dist[5][5][5] = { 0, };
	if (maze[0][0][0] == 0 or maze[4][4][4] == 0)
		return 9999;

	queue<tuple<int, int, int>> q;
	q.push({ 0,0,0 });
	dist[0][0][0] = 1;

	while (!q.empty())
	{
		tuple<int, int, int> cur = q.front();
		q.pop();
		for (int dir = 0; dir < 6; dir++)
		{
			int x, y, z;
			tie(x, y, z) = cur;
			int nx = x + dx[dir];
			int ny = y + dy[dir];
			int nz = z + dz[dir];

			if (0 > nx || 5 <= nx || 0 > ny || 5 <= ny || 0 > nz || 5 <= nz)
				continue;
			if (maze[nx][ny][nz] == 0 || dist[nx][ny][nz] != 0)
				continue;
			if (nx == 4 && ny == 4 && nz == 4)
				return dist[x][y][z];

			dist[nx][ny][nz] = dist[x][y][z] + 1;
			q.push({ nx,ny,nz });
		}
	}
	return 9999;
}
```
   
   
## 틀린답안
```cpp
#include <bits/stdc++.h>
using namespace std;

#define X first
#define Y second

int inputBoard[5][5][5];
int board[5][5][5];
int ans;

int dx[] = { 1, 0, -1, 0, 0, 0 };
int dy[] = { 0, 1, 0, -1, 0, 0 };
int dz[] = { 0, 0, 0, 0, 1, -1 };

void SortBoard();
void RotateBoard();
void BFSBoard();

//Debug
void PrintBoard();

int main(void)
{
	ios::sync_with_stdio(0);
	cin.tie(0);

	for (int k = 0; k < 5; k++)
		for (int i = 0; i < 5; i++)
			for (int j = 0; j < 5; j++)
				cin >> inputBoard[k][i][j];

	for (int k = 0; k < 5; k++)
		for (int i = 0; i < 5; i++)
			for (int j = 0; j < 5; j++)
				board[k][i][j] = inputBoard[k][i][j];

	ans = 9999;
	SortBoard();

	if (ans == 9999)
		cout << -1;
	else
		cout << ans;
}

void SortBoard()
{
	int order[5] = { 0, 1, 2, 3, 4 };
	do
	{
		for (int k = 0; k < 5; k++)
			for (int i = 0; i < 5; i++)
				for (int j = 0; j < 5; j++)
					board[k][i][j] = inputBoard[order[k]][i][j];
		RotateBoard();
	} while (next_permutation(order, order + 5));
}

void RotateBoard()
{
	int boardTemp[5][5][5];

	//=============================================
	//해당 코드 추가 시 정답
	int boardInit[5][5][5];	//이번회차 판들의 순서 저장

	//이번회차 판들의 순서 저장
	for (int i = 0; i < 5; i++)
		for (int j = 0; j < 5; j++)
			for (int k = 0; k < 5; k++)
				boardInit[i][j][k] = board[i][j][k];
	//=============================================

	int maxRotate = 1;
	for (int i = 0; i < 5; i++) maxRotate *= 4;
	for (int temp = 0; temp < maxRotate; temp++)
	{
		//=============================================
		//해당 코드 추가 시 정답
		//회전 초기화 -> 회전초기화 하지 않을 경우 4진수에 따른 제대로 된 방향을 보지 않게 됨
		//즉, 확인하지 못하고 지나치는 판이 발생할 수 있다
		for (int i = 0; i < 5; i++)
			for (int j = 0; j < 5; j++)
				for (int k = 0; k < 5; k++)
					board[i][j][k] = boardInit[i][j][k];
		//=============================================

		int brute = temp;
		for (int k = 0; k < 5; k++)
		{
			int dir = brute % 4;
			brute /= 4;

			for (int rotNum = 0; rotNum < dir; rotNum++)
			{
				for (int i = 0; i < 5; i++)
					for (int j = 0; j < 5; j++)
						boardTemp[k][i][j] = board[k][5 - 1 - j][i];

				for (int i = 0; i < 5; i++)
					for (int j = 0; j < 5; j++)
						board[k][i][j] = boardTemp[k][i][j];
			}
		}
		//PrintBoard();
		BFSBoard();
	}
}

void BFSBoard()
{
	if (board[0][0][0] == 0 || board[4][4][4] == 0)
		return;
	int DistBoard[5][5][5] = { 0, };

	queue<tuple<int, int, int>> Q;
	Q.push({ 0, 0, 0 });
	DistBoard[0][0][0] = 1;

	while (!Q.empty())
	{
		int curX, curY, curZ;
		auto cur = Q.front();
		tie(curX, curY, curZ) = cur;
		Q.pop();
		for (int dir = 0; dir < 6; dir++)
		{
			int nx = curX + dx[dir];
			int ny = curY + dy[dir];
			int nz = curZ + dz[dir];

			if (nx < 0 || nx >= 5 || ny < 0 || ny >= 5 || nz < 0 || nz >= 5)
				continue;
			if (board[nx][ny][nz] != 1 || DistBoard[nx][ny][nz] != 0)
				continue;
			if (nx == 4 && ny == 4 && nz == 4)
			{
				ans = min(ans, DistBoard[curX][curY][curZ]);
				return;
			}

			DistBoard[nx][ny][nz] = DistBoard[curX][curY][curZ] + 1;
			Q.push({ nx, ny, nz });
		}
	}
}

void PrintBoard()
{
	cout << "===============================\n";
	for (int k = 0; k < 5; k++)
	{
		for (int i = 0; i < 5; i++)
		{
			for (int j = 0; j < 5; j++)
			{
				cout << board[k][i][j];
			}
			cout << "\n";
		}
		cout << "\n";
	}
	cout << "\n";
}
```