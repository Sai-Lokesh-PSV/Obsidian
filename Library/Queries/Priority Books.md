---
cssClasses: cards
---
```dataview
table Authors, ("![|100](" + CoverSmallUrl + ")") as "Cover", TotalPage, Rating
from "Books"
where Priority
sort file.name asc
```
