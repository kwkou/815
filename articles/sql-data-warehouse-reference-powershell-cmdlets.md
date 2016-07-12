<properties
   pageTitle="适用于 Azure SQL 数据仓库的 PowerShell cmdlet"
   description="了解 Azure SQL 数据仓库的最常用 PowerShell cmdlet，包括如何暂停和恢复数据库。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="sonyam"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="05/14/2016"
   wacn.date="06/27/2016"/>

# 适用于 SQL 数据仓库的 PowerShell cmdlet 和 REST API

可以使用 Azure PowerShell cmdlet 或 REST API 来管理许多 SQL 数据仓库管理任务。下面是如何使用 PowerShell 命令自动执行 SQL 数据仓库中的常见任务的一些示例。或者，如需可自动执行这些相同任务的 REST API 列表，请参阅 [Operations for Azure SQL Databases（对 Azure SQL 数据库的操作）][]。

> [AZURE.NOTE]  若要对 SQL 数据仓库使用 Azure PowerShell，需要安装 Azure PowerShell 1.0.3 或更高版本。可以通过运行 **Get-Module -ListAvailable -Name Azure** 来检查版本。可以通过 [Microsoft Web 平台安装程序][]安装最新版本。有关安装最新版本的详细信息，请参阅 [How to install and configure Azure PowerShell（如何安装和配置 Azure PowerShell）][]。

## Azure PowerShell cmdlet 入门

1. 打开 Windows PowerShell。 
2. 在 PowerShell 提示符下，运行以下命令以登录到 Azure Resource Manager，然后选择你的订阅。

        Login-AzureRmAccount -EnvironmentName AzureChinaCloud
        Get-AzureRmSubscription
        Select-AzureRmSubscription -SubscriptionName "MySubscription"

## 暂停 SQL 数据仓库示例

暂停名为“Server01”的服务器上托管的名为“Database02”的数据库。 该服务器位于名为“ResourceGroup1”的 Azure 资源组中。

    Suspend-AzureRmSqlDatabase –ResourceGroupName "ResourceGroup1" –ServerName "Server01" –DatabaseName "Database02"

作为一种变体，此示例可通过管道将检索到的对象传递给 [Suspend-AzureRmSqlDatabase][]。因此将会暂停该数据库。最后一个命令显示结果。

    $database = Get-AzureRmSqlDatabase –ResourceGroupName "ResourceGroup1" –ServerName "Server01" –DatabaseName "Database02"
    $resultDatabase = $database | Suspend-AzureRmSqlDatabase
    $resultDatabase

## 启动 SQL 数据仓库示例

恢复“Server01”的服务器上托管的“Database02”数据库的运行。 该服务器包含在名为“ResourceGroup1”的资源组中。

    Resume-AzureRmSqlDatabase –ResourceGroupName "ResourceGroup1" –ServerName "Server01" -DatabaseName "Database02"

作为一种变体，此示例可从“ResourceGroup1”资源组包含的“Server01”服务器中检索“Database02”数据库。 它通过管道将检索到的对象传递给 [Resume-AzureRmSqlDatabase][]。

	$database = Get-AzureRmSqlDatabase –ResourceGroupName "ResourceGroup1" –ServerName "Server01" –DatabaseName "Database02"
	$resultDatabase = $database | Resume-AzureRmSqlDatabase


> [AZURE.NOTE] 注意，如果服务器是 foo.database.windows.net，请使用“foo”作为 Powershell cmdlet 中的 -ServerName。

## 经常使用的 PowerShell cmdlet

以下 PowerShell cmdlet 经常与 Azure SQL 数据仓库配合使用。

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

## 后续步骤
有关更多的 PowerShell 示例，请参阅：

- [Create a SQL Data Warehouse using PowerShell（使用 PowerShell 创建 SQL 数据仓库）][]
- [Restore from snapshot（从快照还原）][]
- [Geo-restore from snapshot（从快照异地还原）][]

如需可以使用 PowerShell 自动执行的所有任务的列表，请参阅 [Azure SQL 数据库 Cmdlet][]。

<!--Image references-->

<!--Article references-->
[How to install and configure Azure PowerShell（如何安装和配置 Azure PowerShell）]: /documentation/articles/powershell-install-configure/
[Create a SQL Data Warehouse using PowerShell（使用 PowerShell 创建 SQL 数据仓库）]: /documentation/articles/sql-data-warehouse-get-started-provision-powershell/
[Restore from snapshot（从快照还原）]: /documentation/articles/sql-data-warehouse-backup-and-restore-from-snapshot/
[Geo-restore from snapshot（从快照异地还原）]: /documentation/articles/sql-data-warehouse-backup-and-restore-from-geo-restore-snapshot/

<!--MSDN references-->
[Azure SQL 数据库 Cmdlet]: https://msdn.microsoft.com/zh-cn/library/mt574084.aspx
[Operations for Azure SQL Databases（对 Azure SQL 数据库的操作）]: https://msdn.microsoft.com/zh-cn/library/azure/dn505719.aspx
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
[Microsoft Web 平台安装程序]: https://aka.ms/webpi-azps

<!---HONumber=Mooncake_0620_2016-->