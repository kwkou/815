<properties
    pageTitle="在 Azure SQL 数据仓库中使用 T-SQL 进行暂停、恢复和缩放 | Azure"
    description="可通过调整 DWU 来提高性能的 Transact-SQL (T-SQL) 任务。通过在非高峰期缩减性能来节省成本。"
    services="sql-data-warehouse"
    documentationcenter="NA"
    author="barbkess"
    manager="jhubbard"
    editor="" />
<tags
    ms.assetid="a970d939-2adf-4856-8a78-d4fe8ab2cceb"
    ms.service="sql-data-warehouse"
    ms.devlang="NA"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="data-services"
    ms.date="10/31/2016"
    wacn.date="03/20/2017"
    ms.author="barbkess" />  


# 管理 Azure SQL 数据仓库中的计算能力 (T-SQL)

> [AZURE.SELECTOR]
- [概述](/documentation/articles/sql-data-warehouse-manage-compute-overview/)
- [门户](/documentation/articles/sql-data-warehouse-manage-compute-portal/)
- [PowerShell](/documentation/articles/sql-data-warehouse-manage-compute-powershell/)
- [REST](/documentation/articles/sql-data-warehouse-manage-compute-rest-api/)
- [TSQL](/documentation/articles/sql-data-warehouse-manage-compute-tsql/)

## <a name="current-dwu-bk"></a>查看当前的 DWU 设置
若要查看数据库的当前 DWU 设置，请执行以下操作：

1. 在 Visual Studio 2015 中打开“SQL Server 对象资源管理器”。
2. 连接到与逻辑 SQL 数据库服务器关联的 master 数据库。
3. 从 sys.database\_service\_objectives 动态管理视图中选择。下面是一个示例：

        SELECT
    	 db.name [Database],
    	 ds.edition [Edition],
    	 ds.service_objective [Service Objective]
        FROM
    	 sys.database_service_objectives ds
    	 JOIN sys.databases db ON ds.database_id = db.database_id



## <a name="scale-compute-bk"></a> <a name="scale-dwu-bk"> 缩放计算
[AZURE.INCLUDE [SQL Data Warehouse scale DWUs description（SQL 数据仓库缩放 DWU 说明）](../../includes/sql-data-warehouse-scale-dwus-description.md)]

若要更改 DWU，请执行以下操作：

1. 连接到与逻辑 SQL 数据库服务器关联的 master 数据库。
2. 使用 [ALTER DATABASE][ALTER DATABASE] TSQL 语句。以下示例将数据库 MySQLDW 的服务级别目标设置为 DW1000。


        ALTER DATABASE MySQLDW
        MODIFY (SERVICE_OBJECTIVE = 'DW1000')
        ;

## <a name="next-steps-bk"></a>后续步骤
有关其他管理任务，请参阅[管理概述][Management overview]。

<!--Image references-->

<!--Article references-->
[Service capacity limits]: /documentation/articles/sql-data-warehouse-service-capacity-limits/
[Management overview]: /documentation/articles/sql-data-warehouse-overview-manage/
[Manage compute power overview]: /documentation/articles/sql-data-warehouse-manage-compute-overview/

<!--MSDN references-->

[ALTER DATABASE]: https://msdn.microsoft.com/zh-cn/library/mt204042.aspx


<!--Other Web references-->

[Azure portal]: http://portal.azure.cn/

<!---HONumber=Mooncake_0313_2017-->
<!--Update_Description:update meata properties;wording update-->