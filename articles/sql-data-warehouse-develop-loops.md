<properties
   pageTitle="SQL 数据仓库中的循环 | Azure"
   description="有关在开发解决方案时使用 Azure SQL 数据仓库中的 Transact-SQL 循环和替换游标的技巧。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="lodipalm"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="03/03/2016"
   wacn.date="04/11/2016"/>

# SQL 数据仓库中的循环
SQL 数据仓库支持对重复执行的语句块使用 [WHILE][] 循环。只要指定的条件都成立，或者在代码专门使用 `BREAK` 关键字终止循环之前，这些语句将不断继续。循环特别适合用于替换 SQL 代码中定义的游标。幸运的是，几乎所有以 SQL 代码编写的游标都是快进的只读变体。因此，如果你发现自己必须替换一个游标，[WHILE] 循环是绝佳的替代方案。

## 利用循环和替换 SQL 数据仓库中的游标
但是，在深入学习之前，你应该先自问以下问题：“此游标是否可重写以使用基于集的操作？”。在许多情况下，答案是肯定的，通常这也是最佳方法。基于集的操作的执行速度通常比迭代性的逐行方法要快得多。

可以轻松使用循环构造来替换快进只读游标。下面是一个简单的示例。此代码示例将更新数据库中每个表的统计信息。通过迭代循环中的表，我们就能够依次执行每个命令。

首先，创建一个临时表，其中包含用于标识各个语句的唯一行号：
  
```
CREATE TABLE #tbl 
WITH 
(   LOCATION     = USER_DB
,   DISTRIBUTION = ROUND_ROBIN
)
AS
SELECT  ROW_NUMBER() OVER(ORDER BY (SELECT NULL)) ) AS Sequence
,       [name]
,       'UPDATE STATISTICS '+QUOTENAME([name]) AS sql_code
FROM    sys.tables
;
```

其次，初始化执行循环所需的变量：

```
DECLARE @nbr_statements INT = (SELECT COUNT(*) FROM #tbl)
,       @i INT = 1
;
```

现在，每次对一个语句执行一次循环：

```
WHILE   @i <= @nbr_statements
BEGIN
    DECLARE @sql_code NVARCHAR(4000) = (SELECT sql_code FROM #tbl WHERE Sequence = @i);
    EXEC    sp_executesql @sql_code;
    SET     @i +=1;
END
```

最后，将第一个步骤创建的临时表删除

```
DROP TABLE #tbl;
```


<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->

## 后续步骤
有关更多开发技巧，请参阅[开发概述][]。

<!--Image references-->

<!--Article references-->
[开发概述]: /documentation/articles/sql-data-warehouse-overview-develop/

<!--MSDN references-->
[WHILE]: https://msdn.microsoft.com/zh-cn/library/ms178642.aspx


<!--Other Web references-->



<!---HONumber=Mooncake_0321_2016-->