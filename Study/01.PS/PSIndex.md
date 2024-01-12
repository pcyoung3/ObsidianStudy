---
Index: "[[PSIndex]]"
tags:
  - Study
  - PS
Algorithm_type:
  - Input_Output
---
   
   
# 목차_이론
```dataview
table file.cday as "생성날짜"
from "Study/01.PS/Theory"
FLATTEN Algorithm_type as Type
SORT default(((x) => {
"Input_Output": 1,
"Data_Type": 1,
"time_complexity":1,
"array":2,
"linked_list": 3,
"stack": 4,
"queue": 4,
"deque": 4,
"BFS": 6,
"DFS": 7,
"recursive": 8,
"backtracking": 9,
"simulation": 10,
"sort": 10,
"DP": 11,
"greedy": 12,
"brute force": 13
}[x])(Type), "99") ASC
```
   
   
## 진행상황
---
영상 :  https://www.youtube.com/playlist?list=PLtqbFd2VIQv4O6D6l9HcD732hdrnYb6CY
문제 : https://github.com/encrypted-def/basic-algo-lecture/blob/master/workbook.md
   
   
   
   
# 목차_문제풀이
```dataview
table  rows.file.link AS "이름",  rows.file.cday as "생성날짜"
from "Study/01.PS/Problems"
sort file.ctime asc
FLATTEN Algorithm_type as Type
GROUP BY Type
SORT default(((x) => {
"array":1,
"linked_list": 2,
"stack": 3,
"queue": 4,
"deque": 5,
"BFS": 6,
"DFS": 7,
"recursive": 8,
"backtracking": 9,
"simulation": 10,
"DP": 11,
"greedy": 12,
"brute force": 13,
"sort": 40
}[x])(Type), "99") ASC
```