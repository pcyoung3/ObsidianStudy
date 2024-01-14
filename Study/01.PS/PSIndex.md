---
Index:
  - "[[StudyIndex]]"
tags:
  - Study
  - PS
---
   
   
# 목차_이론
```dataview
table file.cday as "생성날짜"
from "Study/01.PS/Theory"
sort file.cday asc
FLATTEN Algorithm_type as Type
SORT default(((x) => {
"Input_Output": 1,
"Data_Type": 2,
"time_complexity":3,
"array":4,
"linked_list": 5,
"stack": 6,
"queue": 7,
"deque": 8,
"BFS": 9,
"DFS": 10,
"recursive": 11,
"backtracking": 12,
"simulation": 13,
"sort": 14,
"DP": 15,
"greedy": 16,
"brute force": 17
}[x])(Type), "99") asc
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
