<properties
	pageTitle="Stretch Database 的限制 | Azure"
	description="了解 Stretch Database 的限制。"
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
	ms.date="06/14/2016"
	wacn.date="07/25/2016"
	ms.author="douglasl"/>

# Stretch Database 的限制

了解已启用延伸的表的限制，以及当前会阻止你为表启用延伸的限制。

##  <a name="Caveats"></a>已启用延伸的表的限制

已启用延伸的表存在以下限制。

### 约束

-   没有对包含迁移数据的 Azure 表中的 UNIQUE 约束和 PRIMARY KEY 约束强制实施唯一性。

### DML 操作

-   无法在已启用延伸的表中或者在包含已启用延伸的表的视图中更新或删除行。

-   无法将行插入链接服务器上的已启用延伸的表。

### 索引

-   无法为包含已启用延伸的表的视图创建索引。

-   对 SQL Server 索引创建的筛选器不会传播到远程表。

##  <a name="Limitations"></a>当前会阻止你为表启用延伸的限制

以下各项当前会阻止你为表启用延伸。

### 表属性

-   列超过 1,023 个或索引超过 998 个的表

-   包含 FILESTREAM 数据的 FileTables 或表

-   复制的或者正在实际使用更改跟踪或更改数据捕获的表

-   内存优化表

### 数据类型

-   text、ntext 和 image

-   timestamp

-   sql\_variant

-   XML

-   CLR 数据类型，包括 geometry、geography、hierarchyid 和 CLR 用户定义的类型

### 列类型

-   COLUMN\_SET

-   计算列

### 约束

-   默认约束和检查约束

-   引用表的外键约束。在父-子关系中（例如，Order 和 Order\_Detail），可以为子表 (Order\_Detail) 启用延伸，但不能为父表 (Order) 启用延伸。

### 索引

-   全文索引

-   XML 索引

-   空间索引

-   引用表的索引视图

## 另请参阅

[通过运行延伸数据库顾问来识别符合延伸数据库条件的数据库和表](/documentation/articles/sql-server-stretch-database-identify-databases/)

[为数据库启用延伸数据库](/documentation/articles/sql-server-stretch-database-enable-database/)

[为表启用延伸数据库](/documentation/articles/sql-server-stretch-database-enable-table/)

<!---HONumber=Mooncake_0307_2016-->