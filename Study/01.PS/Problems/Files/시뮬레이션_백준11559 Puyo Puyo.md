# Index : [[PSIndex]]
## Tag : #Study #PS
---

## 문제
https://www.acmicpc.net/problem/11559
뿌요뿌요의 룰은 다음과 같다.

> 필드에 여러 가지 색깔의 뿌요를 놓는다. 뿌요는 중력의 영향을 받아 아래에 바닥이나 다른 뿌요가 나올 때까지 아래로 떨어진다.
> 
> 뿌요를 놓고 난 후, 같은 색 뿌요가 4개 이상 상하좌우로 연결되어 있으면 연결된 같은 색 뿌요들이 한꺼번에 없어진다. 이때 1연쇄가 시작된다.
> 
> 뿌요들이 없어지고 나서 위에 다른 뿌요들이 있다면, 역시 중력의 영향을 받아 차례대로 아래로 떨어지게 된다.
> 
> 아래로 떨어지고 나서 다시 같은 색의 뿌요들이 4개 이상 모이게 되면 또 터지게 되는데, 터진 후 뿌요들이 내려오고 다시 터짐을 반복할 때마다 1연쇄씩 늘어난다.
> 
> 터질 수 있는 뿌요가 여러 그룹이 있다면 동시에 터져야 하고 여러 그룹이 터지더라도 한번의 연쇄가 추가된다.

남규는 최근 뿌요뿌요 게임에 푹 빠졌다. 이 게임은 1:1로 붙는 대전게임이라 잘 쌓는 것도 중요하지만, 상대방이 터뜨린다면 연쇄가 몇 번이 될지 바로 파악할 수 있는 능력도 필요하다. 하지만 아직 실력이 부족하여 남규는 자기 필드에만 신경 쓰기 바쁘다. 상대방의 필드가 주어졌을 때, 연쇄가 몇 번 연속으로 일어날지 계산하여 남규를 도와주자!

## 입력

총 12개의 줄에 필드의 정보가 주어지며, 각 줄에는 6개의 문자가 있다.

이때 `.`은 빈공간이고 `.`이 아닌것은 각각의 색깔의 뿌요를 나타낸다.

`R`은 빨강, `G`는 초록, `B`는 파랑, `P`는 보라, `Y`는 노랑이다.

입력으로 주어지는 필드는 뿌요들이 전부 아래로 떨어진 뒤의 상태이다. 즉, 뿌요 아래에 빈 칸이 있는 경우는 없다.

## 출력

현재 주어진 상황에서 몇연쇄가 되는지 출력한다. 하나도 터지지 않는다면 0을 출력한다.

## 예제 입력 1 복사

```
......
......
......
......
......
......
......
......
.Y....
.YG...
RRYG..
RRYGG.
```

## 예제 출력 1 복사

3
   
---
## 풀이
```cpp
#include <bits/stdc++.h>
using namespace std;

#define X first
#define Y second

int dx[4] = { 1, 0, -1, 0 };
int dy[4] = { 0, 1, 0, -1 };

char board[15][10];
bool vis[15][10];
const int row = 12;
const int col = 6;

bool CheckPuyo();
void MovePyuo();
bool BFS(int x, int y);

void Print();
int main()
{
	ios::sync_with_stdio(0);
	cin.tie(0);

	string input;
	for (int i = 0; i < row; i++)
		cin >> board[i];

	int result = 0;
	while (CheckPuyo())
	{
		MovePyuo();
		//Print(); // Debug 용
		result++;
	}

	cout << result;

}
bool CheckPuyo()
{
	bool isContinue = false;

	for (int i = 0; i < row; i++)
	{
		for (int j = 0; j < col; j++)
			isContinue |= BFS(i, j);
	}

	//방문기록 초기화
	for (int i = 0; i < row; i++)
		fill(vis[i], vis[i] + col, false);

	return isContinue;
}

void MovePyuo()
{
	for (int i = 0; i < col; i++)	//가로 6줄에 대해서 한번씩 실행
	{
		int index = row - 1;
		while (index >= 0)	//해당 줄에 있는 첫 '0'의 위치 찾기
		{
			if (board[index][i] == '0') //삭제된 뿌요니까 바꿔야됨
				break;
			index--;
		}
		if (index < 0) //삭제된게 없으면 continue
			continue;

		char line[row];
		for (int lineNum = 0; lineNum < row; lineNum++)	//기존 열 백업
			line[lineNum] = board[lineNum][i];

		//index 다음부터 0이 있으면 swap
		for (int lineNum = index; lineNum >= 0; lineNum--)
		{
			if (board[lineNum][i] != '0' && board[lineNum][i] != '.') //일반 뿌요와 삭제된 뿌요 위치교환
				line[index--] = board[lineNum][i];
			line[lineNum] = '.'; //남아있는 0을 '.'문자로 치환 (0만 해당열에 있을 경우)
		}

		//바꾼 열을 원본에 덮어씌움
		for (int lineNum = 0; lineNum < row; lineNum++)
			board[lineNum][i] = line[lineNum];
	}
}

bool BFS(int x, int y)
{
	char color = board[x][y];

	queue<pair<int, int>> Q;
	vector<pair<int, int>> vecCur;
	Q.push({ x, y });
	vis[x][y] = true;

	while (!Q.empty())
	{
		auto cur = Q.front();
		vecCur.push_back({ cur.X, cur.Y });
		Q.pop();
		for (int dir = 0; dir < 4; dir++)
		{
			int nx = cur.X + dx[dir];
			int ny = cur.Y + dy[dir];
			if (nx < 0 || nx >= row || ny < 0 || ny >= col)
				continue;
			if (board[nx][ny] == '.' || vis[nx][ny])
				continue;
			if (board[nx][ny] != color)
				continue;
			vis[nx][ny] = true;
			Q.push({ nx, ny });
		}
	}

	//인접뿌요가 4개 이상일 때
	if (vecCur.size() >= 4)
	{
		for (auto cur : vecCur)
			board[cur.X][cur.Y] = '0';	//삭제표시
		return true;
	}

	return false;
}


void Print()
{
	cout << "\n\n";
	for (int i = 0; i < row; i++)
	{
		for (int j = 0; j < col; j++)
			cout << board[i][j];
		cout << "\n";
	}
}

```