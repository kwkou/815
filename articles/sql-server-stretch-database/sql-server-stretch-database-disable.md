<properties
	pageTitle="禁用 Stretch Database 和移回远程数据 | Azure"
	description="了解如何为表禁用延伸数据库并选择性地移回远程数据。"
	services="sql-server-stretch-database"
	documentationCenter=""
	authors="douglaslMS"
	manager=""
	editor=""/>

<tags
	ms.service="sql-server-stretch-database"
	ms.workload="data-management"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/05/2016"
	wacn.date="09/05/2016"
	ms.author="douglasl"/>

# 禁用延伸数据库和移回远程数据

若要为某个表禁用延伸数据库，请在 SQL Server Management Studio 中选择该表对应的“延伸”。然后选择以下选项之一。

-   **禁用| Bring data back from Azure**. 将表的远程数据从 Azure 复制回到 SQL Server，然后为该表禁用延伸数据库。 此操作会产生数据传输费用，并且不可取消。

-   **禁用| Leave data in Azure**. 为表禁用延伸数据库。  放弃表的远程数据，使其保留在 Azure 中。

也可以使用 Transact-SQL 为表或数据库禁用延伸数据库。

为表禁用延伸数据库之后，数据迁移将会停止，查询结果将不再包括来自远程表的结果。

如果你只是想要暂停数据迁移，请参阅[暂停和恢复 Stretch Database](/documentation/articles/sql-server-stretch-database-pause/)。

>   [AZURE.NOTE] 针对表或数据库禁用 Stretch Database 不会删除远程对象。若要删除远程表或远程数据库，必须使用 Azure 管理门户。远程对象在删除之前，会持续产生 Azure 费用。有关详细信息，请参阅 [SQL Server Stretch Database 定价](/pricing/details/sql-server-stretch-database/)。

## 为表禁用延伸数据库

### 使用 SQL Server Management Studio 为表禁用延伸数据库

1.  在 SQL Server Management Studio 的对象资源管理器中，选择你要为其禁用延伸数据库的表。

2.  单击右键并选择“Stretch”，然后选择以下选项之一。

    -   **禁用| Bring data back from Azure**. 将表的远程数据从 Azure 复制回到 SQL Server，然后为该表禁用延伸数据库。 此命令不可取消。

    -   **禁用| Leave data in Azure**. 为表禁用延伸数据库。  放弃表的远程数据，使其保留在 Azure 中。

    >   [AZURE.NOTE] 针对表禁用 Stretch Database 不会删除远程数据或远程表。若要删除远程表，必须使用 Azure 管理门户。远程表在删除之前，会持续产生 Azure 存储费用。有关详细信息，请参阅 [SQL Server Stretch Database 定价](/pricing/details/sql-server-stretch-database/)。

### 使用 Transact-SQL 为表禁用延伸数据库

-   若要针对某个表禁用 Stretch 并将该表的远程数据从 Azure 复制回 SQL Server，请运行以下命令。将所有远程数据从 Azure 复制回到 SQL Server 之后，将为表禁用延伸。

    此命令不可取消。


	USE <Stretch 启用的数据库名>;
    GO
    ALTER TABLE <Stretch 启用的表名>  
       SET ( REMOTE\_DATA\_ARCHIVE ( MIGRATION\_STATE = INBOUND ) ) ;
    GO


-   若要为某个表禁用延伸并放弃远程数据，请运行以下命令。

        ALTER TABLE <table_name>
           SET ( REMOTE_DATA_ARCHIVE = OFF_WITHOUT_DATA_RECOVERY ( MIGRATION_STATE = PAUSED ) ) ;

>   [AZURE.NOTE] 针对表禁用 Stretch Database 不会删除远程数据或远程表。若要删除远程表，必须使用 Azure 管理门户。远程表在删除之前，会持续产生 Azure 费用。有关详细信息，请参阅 [SQL Server Stretch Database 定价](/pricing/details/sql-server-stretch-database/)。

## 为数据库禁用延伸数据库
在为某个数据库禁用延伸数据库之前，必须对该数据库中已启用延伸的每个表禁用延伸数据库。

### 使用 SQL Server Management Studio 为数据库禁用延伸数据库

1.  在 SQL Server Management Studio 的对象资源管理器中，选择你要为其禁用延伸数据库的数据库。

2.  单击右键，然后依次选择“任务”、“Stretch”、“禁用”。

>   [AZURE.NOTE] 针对数据库禁用 Stretch Database 不会删除远程数据库。若要删除远程数据库，必须使用 Azure 管理门户。远程数据库在删除之前，会持续产生 Azure 费用。有关详细信息，请参阅 [SQL Server Stretch Database 定价](/pricing/details/sql-server-stretch-database/)。

### 使用 Transact-SQL 为数据库禁用延伸数据库
运行以下命令。

    ALTER DATABASE <database name>
        SET REMOTE_DATA_ARCHIVE = OFF ;

>   [AZURE.NOTE] 针对数据库禁用 Stretch Database 不会删除远程数据库。若要删除远程数据库，必须使用 Azure 管理门户。远程数据库在删除之前，会持续产生 Azure 费用。有关详细信息，请参阅 [SQL Server Stretch Database 定价](/pricing/details/sql-server-stretch-database/)。

## 另请参阅

[ALTER DATABASE SET 选项 (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/bb522682.aspx)

[暂停和恢复延伸数据库](/documentation/articles/sql-server-stretch-database-pause/)

<!---HONumber=Mooncake_0829_2016-->