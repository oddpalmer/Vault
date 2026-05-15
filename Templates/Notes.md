---
title: <% tp.file.title %>
type:
status: 进行中
tags:
ctime: <% tp.date.now("YYYY-MM-DD HH:MM") %>
---
<%*
// 获取时间戳
const timestamp = tp.date.now("YY-MM-DD日HH时: ");
// 弹出输入框获取描述
const title = await tp.system.prompt("请输入标题");
// 在当前光标位置添加新文件链接
tp.file.cursor_append(timestamp + "[[" + title + "]]")
// 执行重命名操作
await tp.file.rename(title); 
await tp.file.move('/学习/' + title)
-%>

 ## 介绍



