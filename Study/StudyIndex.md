---
Index: "[[Index]]"
tags:
  - index
  - Study
---

```dataview
table file.cday as "생성날짜"
from "Study"
where contains(file.name, "Index")
where !contains(file.name, "StudyIndex")
sort file.name asc
```
   
   
# TODO
---
- [x] 코딩테스트 준비
- [x] Udemy Unreal 인강

