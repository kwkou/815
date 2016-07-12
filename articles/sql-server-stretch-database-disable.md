<properties
	pageTitle="禁用SQL Server Stretch Database和移回远程数据 | Azure"
	description="了解如何为表禁用SQL Server Stretch Database并选择性地移回远程数据。"
	services="sql-server-stretch-database"
	documentationCenter=""
	authors="douglasl"
	manager="jhubbard"
	editor="monicar"/>

<tags
	ms.service="sql-server-stretch-database"
	ms.date="02/26/2016"
	wacn.date="03/10/2016"/>

# 禁用SQL Server Stretch Database和移回远程数据

若要为某个表禁用SQL Server Stretch Database，请在 SQL Server Management Studio 中选择该表对应的“延伸”。然后选择以下选项之一。

-   **禁用 | 从 Azure 移回数据**。将表的远程数据从 Azure 复制回到 SQL Server，然后为该表禁用SQL Server Stretch Database。此操作会产生数据传输费用，并且不可取消。

-   **禁用 | 将数据保留在 Azure 中**。为表禁用SQL Server Stretch Database。放弃表的远程数据，使其保留在 Azure 中。

为表禁用SQL Server Stretch Database之后，数据迁移将会停止，查询结果将不再包括来自远程表的结果。

也可以使用 Transact-SQL 为表或数据库禁用SQL Server Stretch Database。

如果你只是想要暂停数据迁移，请参阅[暂停和恢复SQL Server Stretch Database](/documentation/articles/sql-server-stretch-database-pause/)。

## 为表禁用SQL Server Stretch Database

### 使用 SQL Server Management Studio 为表禁用SQL Server Stretch Database

1.  在 SQL Server Management Studio 的对象资源管理器中，选择你要为其禁用SQL Server Stretch Database的表。

2.  单击右键并选择“延伸”，然后选择以下选项之一。

    -   **禁用 | 从 Azure 移回数据**。将表的远程数据从 Azure 复制回到 SQL Server，然后为该表禁用SQL Server Stretch Database。此命令不可取消。

        此操作会产生数据传输费用。有关详细信息，请参阅[数据传输定价详细信息](/pricing/details/data-transfer/)。

        将所有远程数据从 Azure 复制回到 SQL Server 之后，将为表禁用延伸。

    -   **禁用 | 将数据保留在 Azure 中**。为表禁用SQL Server Stretch Database。放弃表的远程数据，使其保留在 Azure 中。

        放弃远程数据和禁用延伸不会删除远程数据。如果你想要删除远程数据，必须使用 Azure 管理门户删除远程表。

### 使用 Transact-SQL 为表禁用SQL Server Stretch Database

-   若要为某个表禁用延伸并将该表的远程数据从 Azure SQL 数据库复制回到 SQL Server，请运行以下命令。此命令不可取消。

    ```tsql
    ALTER TABLE <table name>
       SET ( REMOTE_DATA_ARCHIVE ( MIGRATION_STATE = INBOUND ) ) ;
    ```
    此操作会产生数据传输费用。有关详细信息，请参阅[数据传输定价详细信息](/pricing/details/data-transfer/)。

    将所有远程数据从 Azure 复制回到 SQL Server 之后，将为表禁用延伸。

-   若要为某个表禁用延伸并放弃远程数据，请运行以下命令。

    ```tsql
    ALTER TABLE <table_name>
       SET ( REMOTE_DATA_ARCHIVE = OFF_WITHOUT_DATA_RECOVERY ( MIGRATION_STATE = PAUSED ) ) ;
    ```
    放弃远程数据和禁用延伸不会删除远程数据。如果你想要删除远程数据，必须使用 Azure 管理门户删除远程表。

## 为数据库禁用SQL Server Stretch Database
在为某个数据库禁用SQL Server Stretch Database之前，必须对该数据库中已启用延伸的每个表禁用SQL Server Stretch Database。

### 使用 SQL Server Management Studio 为数据库禁用SQL Server Stretch Database

1.  在 SQL Server Management Studio 的对象资源管理器中，选择你要为其禁用SQL Server Stretch Database的数据库。

2.  单击右键，然后依次选择“任务”、“延伸”、“禁用”。

### 使用 Transact-SQL 为数据库禁用SQL Server Stretch Database
运行以下命令。

```tsql
ALTER DATABASE <database name>
    SET REMOTE_DATA_ARCHIVE = OFF ;
```

## 删除已启用延伸的数据库
删除已启用SQL Server Stretch Database的数据库会删除本地数据库，但不会删除远程数据。如果你想要删除远程数据，必须使用 Azure 管理门户删除远程数据库。

## 另请参阅
[ALTER DATABASE SET 选项 (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/bb522682.aspx)
[暂停和恢复SQL Server Stretch Database](/documentation/articles/sql-server-stretch-database-pause/)

<!---HONumber=Mooncake_0307_2016-->