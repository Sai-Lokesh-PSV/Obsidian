---
date: <% moment(tp.file.title, 'YYYY-[W]WW').startOf('week').add(0,'days').format('YYYY-MM-DD') %>
---
---
```calander-nav
```
---

# Week
<%*
let start_of_the_week = moment(tp.file.title, ‘YYYY-[W]WW’).startOf(‘week’);
let days_in_a_week = 7;
tR += ‘> [! picture]- Pictures\n’;
tR += Array(days_in_week).fill(null).map((x,i)=> ‘> ![[$moment(start_of_the_week).add(i,’d’).format(‘YYYY-MM-DD [] YYYY-MM-DD’)}]]’).join(“\n”) + ‘\n'’;
%>