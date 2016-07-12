<properties 
    pageTitle="使用 T-SQL 创建 Azure SQL 数据库或将其移入弹性池 | Azure" 
    description="使用 T-SQL 在弹性池中创建 Azure SQL 数据库。或使用 T-SQL 将数据库移入和移出池。" 
	services="sql-database" 
    documentationCenter="" 
    authors="sidneyh" 
    manager="jhubbard" 
    editor=""/>

<tags
    ms.service="sql-database"
    ms.date="04/11/2016"
    wacn.date="06/14/2016"/>

# 使用 Transact-SQL 监视和管理弹性数据库池  

> [AZURE.SELECTOR]
- [C#](/documentation/articles/sql-database-elastic-pool-manage-csharp/)
- [PowerShell](/documentation/articles/sql-database-elastic-pool-manage-powershell/)
- [T-SQL](/documentation/articles/sql-database-elastic-pool-manage-tsql/)

使用 [Create Database（Azure SQL 数据库）](https://msdn.microsoft.com/zh-cn/library/dn268335.aspx)和 [Alter Database（Azure SQL 数据库）](https://msdn.microsoft.com/zh-cn/library/mt574871.aspx)命令可以创建数据库并将它移入和移出弹性池。必须先有弹性池才可以使用这些命令。这些命令只影响数据库。无法使用 T-SQL 命令更改新池的创建和池属性（例如最小和最大 eDTU）的设置。


> [AZURE.NOTE] 弹性数据库池目前为预览版，仅适用于 SQL 数据库 V12 服务器。如果你有一个 SQL 数据库 V11 服务器，可以通过一个步骤使用 PowerShell 升级到 V12 并创建池。


## 在弹性池中创建新数据库
结合 SERVICE\_OBJECTIVE 选项使用 CREATE DATABASE 命令。

	CREATE DATABASE db1 ( SERVICE_OBJECTIVE = ELASTIC_POOL (name = [S3M100] ));
	-- Create a database named db1 in a pool named S3M100.

弹性池中的所有数据库都都继承弹性池的服务层（基本、标准、高级）。


## 在弹性池之间移动数据库
结合 MODIFY 使用 ALTER DATABASE 命令，并将 SERVICE\_OBJECTIVE 选项设置为 ELASTIC\_POOL；将名称设置为目标池的名称。

	ALTER DATABASE db1 MODIFY ( SERVICE_OBJECTIVE = ELASTIC_POOL (name = [PM125] ));
	-- Move the database named db1 to a pool named P1M125  


## 将数据库移入弹性池 
结合 MODIFY 使用 ALTER DATABASE 命令，并将 SERVICE\_OBJECTIVE 选项设置为 ELASTIC\_POOL；将名称设置为目标池的名称。

	ALTER DATABASE db1 MODIFY ( SERVICE_OBJECTIVE = ELASTIC_POOL (name = [S3100] ));
	-- Move the database named db1 to a pool named S3100.

## 将数据库移出弹性池
使用 ALTER DATABASE 命令并将 SERVICE\_OBJECTIVE 设置为某个性能级别（S0、S1 等）。

	ALTER DATABASE db1 MODIFY ( SERVICE_OBJECTIVE = 'S1');
	-- Changes the database into a stand-alone database with the service objective S1.

## 列出弹性池中的数据库
使用 [sys.database\_service\_objectives 视图](https://msdn.microsoft.com/zh-cn/library/mt712619)列出弹性池中的所有数据库。登录到 master 数据库可查询该视图。

>[AZURE.NOTE] 目前弹性池中数据库的 service\_objective\_column 将返回服务目标字符串的内部标记。此字符串将由字符串“ElasticPool”替换。
>

	SELECT d.name, slo.*  
	FROM sys.databases d 
	JOIN sys.database_service_objectives slo  
	ON d.database_id = slo.database_id
	WHERE elastic_pool_name = 'MyElasticPool'; 

## 监视弹性池的资源使用情况

使用 [sys.elastic\_pool\_resource\_stats 视图](https://msdn.microsoft.com/zh-cn/library/mt280062.aspx)来检查逻辑服务器上弹性池的资源使用情况统计信息。登录到 master 数据库可查询该视图。

	SELECT * FROM sys.elastic_pool_resource_stats 
	WHERE elastic_pool_name = 'MyElasticPool'
	ORDER BY end_time DESC;

## 监视弹性池中数据库的资源使用情况
使用 [sys.dm\_resource\_stats 视图](https://msdn.microsoft.com/zh-cn/library/dn800981.aspx)或 [sys.resource\_stats 视图](https://msdn.microsoft.com/zh-cn/library/dn269979.aspx)来检查弹性池中数据库的资源使用情况统计信息。此过程类似于查询任何单一数据库的资源使用情况。

## 弹性池操作延迟

- 更改单个数据库的保障 eDTU 数或单个数据库的最大 eDTU 数通常在 5 分钟或更少的时间内完成。
- 如何更改池的池存储限制取决于池中所有数据库使用的空间总容量。更改平均起来每 100 GB 需要 90 分钟或更短的时间。例如，如果池中所有数据库使用的总空间为 200 GB，则更改池的 eDTU/存储限制时，预计延迟为 3 小时或更短的时间。

## 后续步骤

创建弹性数据库池后，你可以通过创建弹性作业来管理池中的弹性数据库。弹性作业可以用来根据池中数据库的数目来运行 T-SQL 脚本。有关详细信息，请参阅[弹性数据库作业概述](/documentation/articles/sql-database-elastic-jobs-overview/)。

<!---HONumber=Mooncake_0606_2016-->
