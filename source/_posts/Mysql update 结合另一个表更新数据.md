---
title: Mysql update 结合另一个表更新数据
date: 2019-10-30 14:50:32
tags: Mysql
---


有时 update 更新语句会需要根据另一个表进行更新，举例如下：

```mysql
-- 方式一：
update tableA a, tableB b set a.Name=b.Name, a.Age=b.Age where a.IDCard=b.IDCard;

-- 方式二:
update tableA a inner join tableB b on a.IDCard=b.IDCard set a.Name=b.Name, a.Age=b.Age;
```

