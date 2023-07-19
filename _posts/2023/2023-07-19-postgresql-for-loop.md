---
title: "在 PostgreSQL 中使用 FOR LOOP 插入多筆資料"
subtitle: ""
excerpt: "PostgreSQL FOR LOOP"
layout: post
author: "blueskyson"
header-style: text
tags:
  - sql
---


插入 Id 為 1 到 300 的資料：

```sql
do $$
BEGIN
	FOR i IN 1..300 LOOP
		INSERT INTO "TableName" ("Id", "ColumnName")
		VALUES (i, 'YourData');
	END LOOP;
END;
$$;
```