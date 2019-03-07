---
layout: post
title:  Excel过滤非数字字符
tags: Excel
categories: Excel
---
Excel过滤单元格中的非数字字符，复制公式，然后按 `Ctrl + Shift + Enter` 确定

```
=SUM(MID(0&A2,LARGE(INDEX(ISNUMBER(--MID(A2,ROW($1:$99),1))*ROW($1:$99),),ROW($1:$99))+1,1)*10^ROW($1:$99)/10) 
```

目标单元格为A2, 可自行修改

![效果](/images/excel-filter-example.png)
