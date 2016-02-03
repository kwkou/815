<properties 
   pageTitle="Azure SQL 数据库 Query Performance Insight" 
   description="查询性能监视可以识别 Azure SQL 数据库中 DTU 消耗最大的查询。" 
   services="sql-database" 
   documentationCenter="" 
   authors="stevestein" 
   manager="jeffreyg" 
   editor="monicar"/>

<tags
   ms.service="sql-database"
   ms.date="09/30/2015"
   wacn.date="01/05/2016"/>

# Azure SQL 数据库 Query Performance Insight


管理和优化关系数据库性能是一项颇具挑战性的操作，需要投入大量的专业知识和时间。Query Performance Insight 提供以下功能，使你能够花费更少的时间来排查数据库性能问题：

- 深入洞察数据库资源 (DTU) 的消耗。 
- 排名靠前的 DTU 消耗查询，经过优化后可能会改善性能。 
- 可以向下钻取查询的详细信息。

> [AZURE.NOTE]Query Performance Insight 目前以预览版提供，仅在 [Azure 经典门户](https://manage.windowsazure.cn)中提供。



## 先决条件

- Query Performance Insight 仅适用于 Azure SQL 数据库 V12。
- Query Performance Insight 要求[查询存储](https://msdn.microsoft.com/zh-cn/library/dn817826.aspx)正在运行，因此当你注册 Query Performance Insight 时已自动启用。
 
 
## 权限

需有以下[基于角色的访问控制](/documentation/articles/role-based-access-control-configure)权限才能使用 Query Performance Insight：

- 查看排名靠前的资源消耗查询和图表时，需有“读取者”、“所有者”、“参与者”、“SQL DB 参与者”或“SQL Server 参与者”权限。 
- 查看查询文本时，需有“所有者”、“参与者”、“SQL DB 参与者”或“SQL Server 参与者”权限。



## 使用 Query Performance Insight

Query Performance Insight 很容易使用：

- 查看排名靠前的资源消耗查询列表。 
- 选择单个查询以查看其详细信息。
- “编辑图表”：用于自定义 DTU 消耗数据的显示方式，或显示不同的时间段。



> [AZURE.NOTE]SQL 数据库的查询存储区需要捕获几个小时的数据，才能提供查询性能洞察数据。如果在某段时间内数据库没有任何活动，或查询存储不处于活动状态，则在显示该期间时图表是空的。如果查询存储未运行，你随时可以启用它。





## 查看排名靠前的 DTU 消耗查询

在[经典门户](https://manage.windowsazure.cn)中执行以下操作：

1. 浏览到 SQL 数据库并单击“Query Performance Insight”。 
 

    排名靠前的查询视图随即打开并列出排名靠前的 DTU 消耗查询。

1. 单击图表周围以获取详细信息。<br>最前面一行显示数据库的整体 DTU 百分比，条形显示所选查询消耗的 DTU 百分比。选择或清除图表要包含或排除的单个查询。

 

1. （可选）单击“编辑图表”以自定义 DTU 消耗数据的显示方式，或显示不同的时间段。

## 查看单个查询的详细信息

若要查看查询详细信息，请执行以下操作：

1. 单击排名靠前的查询列表中的任何查询。<br>详细信息视图随即打开，查询 DTU 消耗已按时间细分。 
3. 单击图表以获取详细信息。<br>最前面一行显示整体 DTU 百分比，条形是所选查询消耗的 DTU 百分比。
4. 检查数据以查看详细度量值，包括运行查询的每个间隔持续时间、运行次数、资源消耗百分比。
    
 

1. （可选）单击“查看脚本”以查看查询文本，并单击“编辑图表”以自定义 DTU 消耗数据的显示方式，或显示不同的时间段。




## 摘要

Query Performance Insight 可帮助你了解查询工作负荷的影响，以及它与数据库资源消耗的关系。使用此功能可以了解排名靠前的消耗查询，并在发生问题之前轻松找出问题。单击数据库边栏选项卡上的“Query Performance Insight”磁贴可以查看排名靠前的资源 (DTU) 消耗查询。




## 后续步骤

数据库工作负荷是动态的，并且不断地更改。监视查询并继续微调以优化性能。

查看[索引顾问](/documentation/articles/sql-database-index-advisor)，以获取改善 SQL 数据库性能的其他建议。

<!--Image references-->
[1]: ./media/sql-database-query-performance/tile.png
[2]: ./media/sql-database-query-performance/top-queries.png
[3]: ./media/sql-database-query-performance/query-details.png

<!---HONumber=Mooncake_1221_2015-->
