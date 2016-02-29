<properties
   pageTitle="SQL 数据仓库中的 cmdlet 入门 | Microsoft Azure"
   description="使用 PowerShell cmdlet 暂停和重新启动 SQL 数据仓库"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="sidneyh"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="01/11/2016"
   wacn.date="02/26/2016"/>

# Azure 数据仓库 cmdlet 和 REST API 入门

可以使用 Azure PowerShell cmdlet 或 REST API 来管理 SQL 数据仓库。

为 **Azure SQL 数据库**定义的命令也可用于 **SQL 数据仓库**。有关最新列表，请参阅 [Azure SQL Cmdlet](https://msdn.microsoft.com/zh-cn/library/mt574084.aspx)。**Suspend-AzureRmSqlDatabase** 和 **Resume-AzureRmSqlDatabase** 是为 SQL 数据仓库补充设计的 cmdlet（如下所述）。

同样，**SQL Azure 数据库**的 REST API 也可用于 **SQL 数据仓库**实例。有关最新列表，请参阅[对 Azure SQL 数据库的操作](https://msdn.microsoft.com/zh-cn/library/azure/dn505719.aspx)。

## 获取和运行 Azure PowerShell cmdlet

1. 若要下载 Azure PowerShell 模块，请运行 [Microsoft Web 平台安装程序](http://go.microsoft.com/fwlink/p/?linkid=320376&clcid=0x409)。 
2. 若要运行该模块，请在开始窗口中键入 **Microsoft Azure PowerShell**。
3. 如果尚未将你的帐户添加到计算机，请运行以下 cmdlet。有关详细信息，请参阅[如何安装和配置 Azure PowerShell]()：

```
Add-AzureAccount
```

3. 为要暂停或恢复的数据库选择订阅。此示例选择名为“MySubscription”的订阅。

```
Select-AzureRmSubscription -SubscriptionName "MySubscription"
```

## Suspend-AzureRmSqlDatabase

有关命令参考，请参阅 [Suspend-AzureRmSQLDatabase](https://msdn.microsoft.com/zh-cn/library/mt619337.aspx)。

### 示例 1：在服务器上按名称暂停数据库

此示例将暂停“Server01”服务器上托管的“Database02”数据库。 该服务器位于名为“ResourceGroup1”的 Azure 资源组中。

```
Suspend-AzureRmSqlDatabase –ResourceGroupName "ResourceGroup11" –ServerName "Server01" –DatabaseName "Database02"
```

### 示例 2：暂停数据库对象

此示例从“ResourceGroup1”资源组包含的“Server01”服务器中检索“Database02”数据库。 它通过管道将检索到的对象传递给 **Suspend-AzureRmSqlDatabase**。因此将会暂停该数据库。最后一个命令显示结果。

```
$database = Get-AzureRmSqlDatabase –ResourceGroupName "ResourceGroup11" –ServerName "Server01" –DatabaseName "Database02"
$resultDatabase = $database | Suspend-AzureRmSqlDatabase
$resultDatabase
```

## Resume-AzureSqlDatabase

有关命令参考，请参阅 [Resume-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/mt619347.aspx)

### 示例 1：在服务器上按名称恢复数据库

此示例将恢复“Server01”服务器上托管的“Database02”数据库的运行。 该服务器包含在名为“ResourceGroup1”的资源组中。

```
Resume-AzureRmSqlDatabase –ResourceGroupName "ResourceGroup11" –ServerName "Server01" -DatabaseName "Database02"
```

### 示例 2：恢复数据库对象

此示例从“ResourceGroup1”资源组包含的“Server01”服务器中检索“Database02”数据库。 通过管道将该对象传递给 **Resume-AzureRmSqlDatabase**。

```
$database = Get-AzureRmSqlDatabase –ResourceGroupName "ResourceGroup11" –ServerName "Server01" –DatabaseName "Database02"
$resultDatabase = $database | Resume-AzureRmSqlDatabase
```

## Get-AzureRmSqlDatabaseRestorePoints

此 cmdlet 列出 Azure SQL 数据仓库数据库的备份还原点。还原点可用于还原数据库。
所返回对象的属性如下所示。

属性|说明
---|---
RestorePointType|DISCRETE / CONTINUOUS。离散还原点描述了可将 Azure SQL 数据仓库数据库还原到的可能时间点。连续还原点描述了可还原 Azure SQL 数据库的最早可能时间点。数据库可还原到最早时间点之后的任一时间点。
EarliestRestoreDate|最早还原时间（在 restorePointType = CONTINUOUS 时填充）
RestorePointCreationDate |备份快照时间（在 restorePointType = DISCRETE 时填充）

### 示例 1：在服务器上按名称检索数据库的还原点
此示例从“ResourceGroup1”资源组包含的“Server01”服务器中检索“Database02”数据库的还原点。

```	
$restorePoints = Get-AzureRmSqlDatabaseRestorePoints –ResourceGroupName "ResourceGroup11" –ServerName "Server01" –DatabaseName "Database02"
$restorePoints
```


### 示例 2：恢复数据库对象

此示例从“ResourceGroup1”资源组包含的“Server01”服务器中检索“Database02”数据库。 通过管道将数据库对象传递给 **Get-AzureRmSqlDatabase**，结果为数据库的还原点。最后一个命令输出结果。

```
$database = Get-AzureRmSqlDatabase –ResourceGroupName "ResourceGroup11" –ServerName "Server01" –DatabaseName "Database02"
$restorePoints = $database | Get-AzureRmSqlDatabaseRestorePoints
$retorePoints
```


> [AZURE.NOTE]请注意，如果服务器是 foo.database.chinacloudapi.cn，请使用“foo”作为 Powershell cmdlet 中的 -ServerName。


## 后续步骤
有关更多参考信息，请参阅 [SQL 数据仓库参考概述][]。

<!--Image references-->

<!--Article references-->
[SQL 数据仓库参考概述]: /documentation/articles/sql-data-warehouse-overview-reference
[How to install and configure Azure PowerShell]: /documentation/articles/powershell-install-configure

<!--MSDN references-->


<!--Other Web references-->
[gog]: http://google.com/
[yah]: http://search.yahoo.com/
[msn]: http://search.msn.com/

<!---HONumber=Mooncake_0215_2016-->