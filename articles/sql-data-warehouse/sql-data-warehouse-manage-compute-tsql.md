<properties
    pageTitle="在 Azure SQL 数据仓库中使用 T-SQL 进行暂停、恢复和缩放 | Azure"
    description="可通过调整 DWU 来提高性能的 Transact-SQL (T-SQL) 任务。 通过在非高峰期缩减性能来节省成本。"
    services="sql-data-warehouse"
    documentationcenter="NA"
    author="barbkess"
    manager="jhubbard"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="a970d939-2adf-4856-8a78-d4fe8ab2cceb"
    ms.service="sql-data-warehouse"
    ms.devlang="NA"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="data-services"
    ms.date="03/30/2017"
    wacn.date="05/08/2017"
    ms.author="barbkess"
    ms.sourcegitcommit="2c4ee90387d280f15b2f2ed656f7d4862ad80901"
    ms.openlocfilehash="d005755a0c4834ffd1986a212be4c46029cfb857"
    ms.lasthandoff="04/28/2017" />

# <a name="manage-compute-power-in-azure-sql-data-warehouse-t-sql"></a>管理 Azure SQL 数据仓库中的计算能力 (T-SQL)
> [AZURE.SELECTOR]
- [概述](/documentation/articles/sql-data-warehouse-manage-compute-overview/)
- [门户](/documentation/articles/sql-data-warehouse-manage-compute-portal/)
- [PowerShell](/documentation/articles/sql-data-warehouse-manage-compute-powershell/)
- [REST](/documentation/articles/sql-data-warehouse-manage-compute-rest-api/)
- [TSQL](/documentation/articles/sql-data-warehouse-manage-compute-tsql/)

## <a name="current-dwu-bk"></a> 查看当前的 DWU 设置
若要查看数据库的当前 DWU 设置，请执行以下操作：

1. 在 Visual Studio 中打开“SQL Server 对象资源管理器”。
2. 连接到与逻辑 SQL 数据库服务器关联的 master 数据库。
3. 从 sys.database_service_objectives 动态管理视图中选择。 下面是一个示例： 

        SELECT
            db.name [Database]
        ,    ds.edition [Edition]
        ,    ds.service_objective [Service Objective]
        FROM
             sys.database_service_objectives ds
        JOIN
            sys.databases db ON ds.database_id = db.database_id

## <a name="scale-dwu-bk"></a> <a name="scale-compute-bk"></a>缩放计算
[AZURE.INCLUDE [SQL Data Warehouse scale DWUs description](../../includes/sql-data-warehouse-scale-dwus-description.md)]

若要更改 DWU，请执行以下操作：

1. 连接到与逻辑 SQL 数据库服务器关联的 master 数据库。
2. 使用 [ALTER DATABASE][ALTER DATABASE] TSQL 语句。 以下示例将数据库 MySQLDW 的服务级别目标设置为 DW1000。 

        ALTER DATABASE MySQLDW
        MODIFY (SERVICE_OBJECTIVE = 'DW1000')
        ;

## <a name="check-database-state-bk"></a><a name="check-database-state-and-operation-progress"></a>查看数据库状态和操作进度

1. 连接到与逻辑 SQL 数据库服务器关联的 master 数据库。
2. 提交查询以查看数据库状态

        SELECT *
        FROM
        sys.databases

3. 提交查询以查看操作状态

        SELECT *
        FROM
            sys.dm_operation_status
        WHERE
            resource_type_desc = 'Database'
        AND 
            major_resource_id = 'MySQLDW'

此 DMV 将返回有关对 SQL 数据仓库的各种管理操作的信息，例如操作和操作状态（IN_PROGRESS 或 COMPLETED）。

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

<!--Update_Description:update meata properties;wording update; add content how to check database state-->