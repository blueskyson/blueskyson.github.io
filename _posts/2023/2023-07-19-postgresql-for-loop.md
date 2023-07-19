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

- `do $$` 和 `$$;`：表示 PL/pgSQL 塊的開始和結束，位於這些定界符內的程式碼將作為一個單一的程式碼塊來執行。
- `BEGIN` 和 `END`：這表示 PL/pgSQL 塊的開始。塊內的所有語句將按照順序依次執行。
- `FOR i IN 1..300 LOOP` 和 `END LOOP;`：這是一個 FOR 迴圈，它從 1 循環到 300。在每次迭代中，迴圈變數 i 會取值從 1 到 300。
