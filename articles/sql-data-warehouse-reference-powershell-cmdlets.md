<properties
   pageTitle="适用于 Azure SQL 数据仓库的 PowerShell cmdlet"
   description="了解 Azure SQL 数据仓库的最常用 PowerShell cmdlet，包括如何暂停和恢复数据库。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="sonyama"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="04/02/2016"
   wacn.date="05/05/2016"/>

# 适用于 SQL 数据仓库的 PowerShell cmdlet 和 REST API

可以使用 Azure PowerShell cmdlet 或 REST API 来管理 SQL 数据仓库。

为 **Azure SQL 数据库**定义的大多数命令也可用于 **SQL 数据仓库**。有关最新列表，请参阅 [Azure SQL Cmdlet](https://msdn.microsoft.com/zh-cn/library/mt574084.aspx)。[Suspend-AzureRmSqlDatabase][] 和 [Resume-AzureRmSqlDatabase][] 是为 SQL 数据仓库补充设计的 cmdlet。

同样，**SQL Azure 数据库**的 REST API 也可用于 **SQL 数据仓库**实例。有关最新列表，请参阅[对 Azure SQL 数据库的操作](https://msdn.microsoft.com/zh-cn/library/azure/dn505719.aspx)。

## Azure PowerShell cmdlet 入门

1. 若要下载 Azure PowerShell 模块，请运行 [Microsoft Web 平台安装程序](http://aka.ms/webpi-azps)。有关此安装程序的详细信息，请参阅 [How to install and configure Azure PowerShell（如何安装和配置 Azure PowerShell）][]。
2. 若要启动 PowerShell，请单击“启动”，然后键入 **Windows PowerShell**。
3. 在 PowerShell 提示符下，运行以下命令以登录到 Azure Resource Manager，然后选择你的订阅。

    ```
    Login-AzureRmAccount -e AzureChinaCloud
    Get-AzureRmSubscription
    Select-AzureRmSubscription -SubscriptionName "MySubscription"
    ```


## 常用的 PowerShell cmdlet

以下 PowerShell cmdlet 经常与 Azure SQL 数据仓库配合使用：


- [Get-AzureRmSqlDatabase][]
- [Get-AzureRmSqlDeletedDatabaseBackup][]
- [Get-AzureRmSqlDatabaseRestorePoints][]
- [New-AzureRmSqlDatabase][]
- [Remove-AzureRmSqlDatabase][]
- [Restore-AzureRmSqlDatabase][] 
- [Resume-AzureRmSqlDatabase][]
- [Select-AzureRmSubscription][]
- [Set-AzureRmSqlDatabase][]
- [Suspend-AzureRmSqlDatabase][]


## SQL 数据仓库示例

这些示例针对只适用于 SQL 数据仓库的功能。

### [Suspend-AzureRmSqlDatabase][]

暂停名为“Server01”的服务器上托管的名为“Database02”的数据库。 该服务器位于名为“ResourceGroup1”的 Azure 资源组中。

```
Suspend-AzureRmSqlDatabase –ResourceGroupName "ResourceGroup1" –ServerName "Server01" –DatabaseName "Database02"
```
作为一种变体，此示例可通过管道将检索到的对象传递给 [Suspend-AzureRmSqlDatabase][]。因此将会暂停该数据库。最后一个命令显示结果。

```
$database = Get-AzureRmSqlDatabase –ResourceGroupName "ResourceGroup1" –ServerName "Server01" –DatabaseName "Database02"
$resultDatabase = $database | Suspend-AzureRmSqlDatabase
$resultDatabase
```

### [Resume-AzureRmSqlDatabase][]

恢复“Server01”的服务器上托管的“Database02”数据库的运行。 该服务器包含在名为“ResourceGroup1”的资源组中。

```
Resume-AzureRmSqlDatabase –ResourceGroupName "ResourceGroup1" –ServerName "Server01" -DatabaseName "Database02"
```

作为一种变体，此示例可从“ResourceGroup1”资源组包含的“Server01”服务器中检索“Database02”数据库。 它通过管道将检索到的对象传递给 [Resume-AzureRmSqlDatabase][]。

```
$database = Get-AzureRmSqlDatabase –ResourceGroupName "ResourceGroup1" –ServerName "Server01" –DatabaseName "Database02"
$resultDatabase = $database | Resume-AzureRmSqlDatabase
```

> [AZURE.NOTE] 注意，如果服务器是 foo.database.windows.net，请使用“foo”作为 Powershell cmdlet 中的 -ServerName。


## 后续步骤
有关更多参考信息，请参阅 [SQL 数据仓库参考概述][]。
有关更多的 PowerShell 示例，请参阅：
- [使用 PowerShell 创建 SQL 数据仓库](/documentation/articles/sql-data-warehouse-get-started-provision-powershell)
- [从快照还原](/documentation/articles/sql-data-warehouse-backup-and-restore-from-snapshot)
- [从快照异地还原](/documentation/articles/sql-data-warehouse-backup-and-restore-from-geo-restore-snapshot)

<!--Image references-->

<!--Article references-->
[SQL 数据仓库参考概述]: /documentation/articles/sql-data-warehouse-overview-reference
[如何安装和配置 Azure PowerShell]: /documentation/articles/powershell-install-configure

<!--MSDN references-->
[Get-AzureRmSqlDatabase]: https://msdn.microsoft.com/zh-cn/library/mt603648.aspx
[Get-AzureRmSqlDeletedDatabaseBackup]: https://msdn.microsoft.com/zh-cn/library/mt693387.aspx
[Get-AzureRmSqlDatabaseRestorePoints]: https://msdn.microsoft.com/zh-cn/library/mt603642.aspx
[New-AzureRmSqlDatabase]: https://msdn.microsoft.com/zh-cn/library/mt619339.aspx
[Remove-AzureRmSqlDatabase]: https://msdn.microsoft.com/zh-cn/library/mt619368.aspx
[Restore-AzureRmSqlDatabase]: https://msdn.microsoft.com/zh-cn/library/mt693390.aspx
[Resume-AzureRmSqlDatabase]: http://msdn.microsoft.com/zh-cn/library/mt619347.aspx
<!-- It appears that Select-AzureRmSubscription isn't documented, so this points to Select-AzureRmSubscription -->
[Select-AzureRmSubscription]: https://msdn.microsoft.com/zh-cn/library/dn722499.aspx
[Set-AzureRmSqlDatabase]: https://msdn.microsoft.com/zh-cn/library/mt619433.aspx
[Suspend-AzureRmSqlDatabase]: http://msdn.microsoft.com/zh-cn/library/mt619337.aspx



<!--Other Web references-->

<!---HONumber=Mooncake_0425_2016-->