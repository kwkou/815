<properties
    pageTitle="SQL 数据库性能优化提示 | Azure"
    description="有关通过评估和改进来调整 Azure SQL 数据库性能的提示。"
    services="sql-database"
    documentationcenter=""
    author="v-shysun"
    manager="felixwu"
    editor=""
    keywords="sql 性能优化, 数据库性能优化, sql 性能优化提示, sql 数据库性能优化" />
<tags
    ms.assetid="eb7b3f66-3b33-4e1b-84fb-424a928a6672"
    ms.service="sql-database"
    ms.custom="monitor and tune"
    ms.workload="data-management"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/07/2017"
    wacn.date="03/24/2017"
    ms.author="v-shysun" />

# SQL 数据库性能调整提示
可随时更改独立数据库的[服务层](/documentation/articles/sql-database-service-tiers/)或增加弹性池的 eDTU 以提高性能，但可能需要首先确定提升和优化查询性能的机会。缺少索引与查询优化不足是数据库性能不佳的常见原因。本文提供在 SQL 数据库中执行性能优化的指南。


## 评估和优化数据库性能的步骤
1.	在 [Azure 门户预览](https://portal.azure.cn)中，单击“SQL 数据库”，选择该数据库，然后使用“监视”图表查找接近其上限的资源。默认将显示 DTU 消耗量。单击“编辑”更改显示的时间范围和值。
2.	使用 [Query Performance Insight](/documentation/articles/sql-database-query-performance/) 评估使用 DTU 的查询，然后使用 [SQL 数据库顾问](/documentation/articles/sql-database-advisor/)查看针对创建和删除索引、参数化查询以及解决架构问题的建议。
3.	可以使用动态管理视图 (DMV)、扩展事件 (Xevent) 和 SSMS 中的查询存储实时获取性能参数。有关详细的监视和优化提示，请参阅[性能指南主题](/documentation/articles/sql-database-performance-guidance/)。


    > [AZURE.IMPORTANT] 建议始终使用最新版本的 Management Studio 以保持与 Azure 和 SQL 数据库的更新同步。[更新 SQL Server Management Studio](https://msdn.microsoft.com/zh-cn/library/mt238290.aspx)。


## 使用更多资源提高数据库性能的步骤
1.	对于独立数据库，可根据需要[更改服务层](/documentation/articles/sql-database-service-tiers/)以提高数据库性能。
2.	对于多个数据库，请考虑使用[弹性池](/documentation/articles/sql-database-elastic-pool-guidance/)自动调整资源规模。

<!---HONumber=Mooncake_0320_2017-->