<properties 
   pageTitle="Azure SQL 数据库性能见解 | Azure" 
   description="Azure SQL 数据库提供的性能工具有助于发现可以提高当前查询性能的方面。" 
   services="sql-database" 
   documentationCenter="" 
   authors="stevestein" 
   manager="jhubbard" 
   editor="monicar"/>

<tags
   ms.service="sql-database"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="data-management" 
   ms.date="07/19/2016" 
   wacn.date="09/26/2016"
   ms.author="sstein"/>

# SQL 数据库性能见解

Azure SQL 数据库提供的性能工具通过提供智能优化操作和建议，有助于识别并提高数据库的性能。

1. 浏览到 [Azure 门户预览](http://portal.azure.cn)中的数据库并单击“所有设置”>“性能”>“概述”以打开“性能”页面。


2. 单击“建议”，打开 [SQL 数据库顾问](#sql-database-advisor)，然后单击“查询”打开[Query Performance Insight](#query-performance-insight)。

    ![查看性能](./media/sql-database-performance/entries.png)



## 性能概述

单击“概述”磁贴或“性能”磁贴可转到数据库的性能仪表板。此视图提供了数据库性能摘要，有助于你进行性能优化和故障排除。

![性能](./media/sql-database-performance/performance.png)

- “建议”磁贴提供数据库的优化建议明细（如果建议超出 3 条，则将显示前 3 条）。单击此磁贴可转到“SQL 数据库顾问”。
- “优化活动”磁贴提供正在进行的和已完成的数据库优化操作摘要，可快速查看优化活动的历史记录。单击此磁贴可转到数据库的完整优化历史记录视图。
- “自动优化”磁贴显示数据库的自动优化配置（其中，将优化操作配置为自动应用于数据库）。单击此磁贴可打开自动化配置对话框。
- “数据库查询”磁贴显示数据库的查询性能摘要（DTU 整体使用情况和排名靠前的资源消耗查询）。单击此磁贴可转到“Query Performance Insight”。



##<a id="sql-database-advisor"></a> SQL 数据库顾问


[SQL 数据库顾问](/documentation/articles/sql-database-advisor/)可提供智能优化建议，有助于提高数据库的性能。

- 关于创建或删除哪些索引的建议（可以选择在没有用户交互的情况下自动应用索引建议和自动回滚对性能产生负面影响的建议）。
- 数据库中发现架构问题时提供的建议。
- 查询可从参数化查询中获益时提供的建议。




##<a id="query-performance-insight"></a> Query Performance Insight

[Query Performance Insight](/documentation/articles/sql-database-query-performance/) 通过提供以下功能使排查数据库性能问题花费的时间更少：

- 深入洞察数据库资源 (DTU) 的消耗。
- 排名靠前的 CPU 消耗查询，经过优化后可能会改善性能。
- 可以向下钻取查询的详细信息。


## 其他资源

- [Azure SQL 数据库的单一数据库性能指导](/documentation/articles/sql-database-performance-guidance/)
- [何时使用弹性数据库池？](/documentation/articles/sql-database-elastic-pool-guidance/)

<!---HONumber=Mooncake_0919_2016-->