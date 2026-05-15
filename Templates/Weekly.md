---
创建时间: <% tp.date.now("YYYY-MM-DD HH:MM") %>
tags:
  - 周记
  - 复盘
---
<%*
const fileName = tp.date.now("YYYY 年第 ww 周");
const monday = tp.date.weekday("YYYY-MM-DD_ddd", 0);
const sunday = tp.date.weekday("YYYY-MM-DD_ddd", 6);
const mondayDate = tp.date.weekday("YYYY-MM-DD", 0);
const sundayDate = tp.date.weekday("YYYY-MM-DD", 6);
await tp.file.rename(fileName);
-%>
### <% fileName %>
(<% monday %>) 至 (<% sunday %>)

### 本周新建笔记
```dataview
table without id
file.link as "笔记",
file.folder as "目录",
status as "状态",
dateformat(file.ctime, "yyyy-MM-dd") as "创建日期"
from "项目" or "学习"
where file.ctime >= date(<% mondayDate %>)
and file.ctime <= date(<% sundayDate %>)
sort file.ctime desc
```


### 已完成任务
```dataview
task
from "记录/日记"
where file.name >= "<% monday %>"
and file.name <= "<% sunday %>"
and completed
```
### 未完成任务
```dataview
task
from "记录/日记"
where file.name >= "<% monday %>"
and file.name <= "<% sunday %>"
and !completed
```

### 学习收获


### 总结