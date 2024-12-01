---
cssClasses: cards
---
```dataview
table Authors, ("![|100](" + CoverSmallUrl + ")") as "Cover", TotalPage, Rating
from "Books"
where !Complete
sort file.name asc
```