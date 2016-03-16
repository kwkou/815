<properties
	pageTitle="暂停和恢复数据迁移（SQL Server Stretch Database）| Azure"
	description="了解如何暂停或继续将数据迁移到 Azure。"
	services="sql-server-stretch-database"
	documentationCenter=""
	authors="douglasl"
	manager="jhubbard"
	editor="monicar"/>

<tags
	ms.service="sql-server-stretch-database"
	ms.date="02/26/2016"
	wacn.date="03/10/2016"/>

# 暂停和恢复数据迁移（SQL Server Stretch Database）

若要暂停或继续将数据迁移到 Azure，请在 SQL Server Management Studio 中选择某个表对应的“延伸”，然后选择“暂停”以暂停数据迁移，或选择“恢复”以恢复数据迁移。也可以使用 Transact-SQL 来暂停或恢复数据迁移。

当你想要排查本地服务器上的问题或者最大程度地提供可用网络带宽时，可以暂停单个表的数据迁移。

## 暂停数据迁移

### 使用 SQL Server Management Studio 暂停数据迁移

1.  在 SQL Server Management Studio 的对象资源管理器中，选择你要暂停其数据迁移的、已启用延伸的表。

2.  单击右键并选择“延伸”，然后选择“暂停”。

### 使用 Transact-SQL 暂停数据迁移
运行以下命令。

```tsql
ALTER TABLE <table name>
    SET ( REMOTE_DATA_ARCHIVE ( MIGRATION_STATE = PAUSED ) ) ;
GO;
```

## 恢复数据迁移

### 使用 SQL Server Management Studio 恢复数据迁移

1.  在 SQL Server Management Studio 的对象资源管理器中，选择你要恢复其数据迁移的、已启用延伸的表。

2.  单击右键并选择“延伸”，然后选择“恢复”。

### 使用 Transact-SQL 恢复数据迁移
运行以下命令。

```tsql
ALTER TABLE <table name>
    SET ( REMOTE_DATA_ARCHIVE ( MIGRATION_STATE = OUTBOUND ) ) ;
```

## 另请参阅
[ALTER TABLE (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/ms190273.aspx)

<!---HONumber=Mooncake_0307_2016-->