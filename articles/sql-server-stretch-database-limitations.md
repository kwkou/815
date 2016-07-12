<properties
	pageTitle="SQL Server Stretch Database的外围应用限制与阻碍性问题 | Azure"
	description="了解启用SQL Server Stretch Database之前必须解决的阻碍性问题。"
	services="sql-server-stretch-database"
	documentationCenter=""
	authors="douglasl"
	manager="jhubbard"
	editor="monicar"/>

<tags
	ms.service="sql-server-stretch-database"
	ms.date="02/26/2016"
	wacn.date="03/10/2016"/>

# SQL Server Stretch Database的外围应用限制与阻碍性问题

了解启用SQL Server Stretch Database之前必须解决的阻碍性问题。

## <a name="Limitations"></a>阻碍性问题
在当前的 SQL Server 2016 预览版中，以下各项会使表不符合延伸的条件。

**表属性**
-   超过 1,023 列

-   超过 998 个索引

-   包含 FILESTREAM 数据的表

-   FileTables

-   复制的表

-   正在实际使用更改跟踪或更改数据捕获的表

-   内存优化表

**数据类型和列属性**
-   timestamp

-   sql\_variant

-   XML

-   geometry

-   geography

-   hierarchyid

-   CLR 用户定义的类型 (UDT)

**列类型**
-   COLUMN\_SET

-   计算列

**约束**
-   检查约束

-   默认约束

-   引用表的外键约束

**索引**
-   全文索引

-   XML 索引

-   空间索引

-   引用表的索引视图

## <a name="Caveats"></a>已启用延伸的表的限制和注意事项
在当前的 SQL Server 2016 预览版中，已启用延伸的表存在以下限制或注意事项。

-   没有对已启用延伸的表中的 UNIQUE 约束和 PRIMARY KEY 约束强制实施唯一性。

-   无法对已启用延伸的表运行 UPDATE 或 DELETE 操作。

-   无法使用 INSERT 在远程 Azure SQL 数据库表中插入数据。

-   无法为包含已启用延伸的表的视图创建索引。

-   无法在包含已启用延伸的表的视图中更新或删除数据。但是，可以在包含已启用延伸的表的视图中插入数据。

-   对索引创建的筛选器不会传播到远程表。

## 另请参阅
[通过运行SQL Server Stretch Database顾问识别SQL Server Stretch Database的数据库和表](/documentation/articles/sql-server-stretch-database-identify-databases/)
[为数据库启用SQL Server Stretch Database](/documentation/articles/sql-server-stretch-database-enable-database/)
[为表启用SQL Server Stretch Database](/documentation/articles/sql-server-stretch-database-enable-table/)

<!---HONumber=Mooncake_0307_2016-->