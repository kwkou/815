<properties
    pageTitle="适用于 Azure SQL 数据仓库的 PowerShell cmdlet"
    description="了解 Azure SQL 数据仓库的最常用 PowerShell cmdlet，包括如何暂停和恢复数据库。"
    services="sql-data-warehouse"
    documentationcenter="NA"
    author="barbkess"
    manager="jhubbard"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="6f0d5772-f05f-4cc8-9749-4adb153dfd50"
    ms.service="sql-data-warehouse"
    ms.devlang="NA"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="data-services"
    ms.custom="reference"
    ms.date="10/31/2016"
    wacn.date="05/08/2017"
    ms.author="barbkess"
    ms.sourcegitcommit="2c4ee90387d280f15b2f2ed656f7d4862ad80901"
    ms.openlocfilehash="d58d9ea8ce13efa6d6bd36cb8b00483a9dd92894"
    ms.lasthandoff="04/28/2017" />

# <a name="powershell-cmdlets-and-rest-apis-for-sql-data-warehouse"></a>适用于 SQL 数据仓库的 PowerShell cmdlet 和 REST API
可以使用 Azure PowerShell cmdlet 或 REST API 来管理许多 SQL 数据仓库管理任务。  下面是如何使用 PowerShell 命令自动执行 SQL 数据仓库中的常见任务的一些示例。  如需一些好的 REST 示例，请参阅 [使用 REST 管理可伸缩性][Manage scalability with REST]一文。

> [AZURE.NOTE]
> 若要对 SQL 数据仓库使用 Azure PowerShell，需要安装 Azure PowerShell 1.0.3 或更高版本。  可以通过运行 **Get-Module -ListAvailable -Name Azure** 来检查版本。  可通过 [Microsoft Web 平台安装程序][Microsoft Web Platform Installer]安装最新版本。  有关安装最新版本的详细信息，请参阅[如何安装和配置 Azure PowerShell][How to install and configure Azure PowerShell]。
> 
> 

## <a name="get-started-with-azure-powershell-cmdlets"></a>Azure PowerShell cmdlet 入门
1. 打开 Windows PowerShell。
2. 在 PowerShell 提示符下，运行以下命令以登录到 Azure Resource Manager，然后选择你的订阅。

        Login-AzureRmAccount -EnvironmentName AzureChinaCloud
        Get-AzureRmSubscription
        Select-AzureRmSubscription -SubscriptionName "MySubscription"

## <a name="pause-sql-data-warehouse-example"></a>暂停 SQL 数据仓库示例
暂停名为“Server01”的服务器上托管的名为“Database02”的数据库。  该服务器位于名为“ResourceGroup1”的 Azure 资源组中。

    Suspend-AzureRmSqlDatabase -ResourceGroupName "ResourceGroup1" -ServerName "Server01" -DatabaseName "Database02"

作为一种变体，此示例可通过管道将检索到的对象传递给 [Suspend-AzureRmSqlDatabase][Suspend-AzureRmSqlDatabase]。  因此将会暂停该数据库。 最后一个命令显示结果。

    $database = Get-AzureRmSqlDatabase -ResourceGroupName "ResourceGroup1" -ServerName "Server01" -DatabaseName "Database02"
    $resultDatabase = $database | Suspend-AzureRmSqlDatabase
    $resultDatabase

## <a name="start-sql-data-warehouse-example"></a>启动 SQL 数据仓库示例

恢复“Server01”的服务器上托管的“Database02”数据库的运行。 该服务器包含在名为“ResourceGroup1”的资源组中。

    Resume-AzureRmSqlDatabase -ResourceGroupName "ResourceGroup1" -ServerName "Server01" -DatabaseName "Database02"

作为一种变体，此示例可从“ResourceGroup1”资源组包含的“Server01”服务器中检索“Database02”数据库。 它通过管道将检索到的对象传递给 [Resume-AzureRmSqlDatabase][Resume-AzureRmSqlDatabase]。

    $database = Get-AzureRmSqlDatabase -ResourceGroupName "ResourceGroup1" -ServerName "Server01" -DatabaseName "Database02"
    $resultDatabase = $database | Resume-AzureRmSqlDatabase

> [AZURE.NOTE]
> 注意，如果服务器是 foo.database.chinacloudapi.cn，请使用“foo”作为 PowerShell cmdlet 中的 -ServerName。
> 
> 

## <a name="other-supported-powershell-cmdlets"></a>其他支持的 PowerShell cmdlet
Azure SQL 数据仓库支持以下 PowerShell cmdlet。

* [Get-AzureRmSqlDatabase][Get-AzureRmSqlDatabase]
* [Get-AzureRmSqlDeletedDatabaseBackup][Get-AzureRmSqlDeletedDatabaseBackup]
* [Get-AzureRmSqlDatabaseRestorePoints][Get-AzureRmSqlDatabaseRestorePoints]
* [New-AzureRmSqlDatabase][New-AzureRmSqlDatabase]
* [Remove-AzureRmSqlDatabase][Remove-AzureRmSqlDatabase]
* [Restore-AzureRmSqlDatabase][Restore-AzureRmSqlDatabase]
* [Resume-AzureRmSqlDatabase][Resume-AzureRmSqlDatabase]
* [Select-AzureRmSubscription][Select-AzureRmSubscription]
* [Set-AzureRmSqlDatabase][Set-AzureRmSqlDatabase]
* [Suspend-AzureRmSqlDatabase][Suspend-AzureRmSqlDatabase]

## <a name="next-steps"></a>后续步骤
有关更多的 PowerShell 示例，请参阅：

* [Create a SQL Data Warehouse using PowerShell（使用 PowerShell 创建 SQL 数据仓库）][Create a SQL Data Warehouse using PowerShell]
* [数据库还原][Database restore]

有关可使用 PowerShell 自动执行的其他列表，请参阅 [Azure SQL 数据库 Cmdlet][Azure SQL Database Cmdlets]。 请注意，Azure SQL 数据仓库并非支持全部 Azure SQL 数据库 cmdlet。  有关可以使用 REST 自动执行的任务的列表，请参阅 [Azure SQL 数据库的操作][Operations for Azure SQL Databases]。

<!--Image references-->

<!--Article references-->
[How to install and configure Azure PowerShell]: https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs
[Create a SQL Data Warehouse using PowerShell]: /documentation/articles/sql-data-warehouse-get-started-provision-powershell/
[Database restore]: /documentation/articles/sql-data-warehouse-restore-database-powershell/
[Manage scalability with REST]: /documentation/articles/sql-data-warehouse-manage-compute-rest-api/

<!--MSDN references-->
[Azure SQL Database Cmdlets]: https://msdn.microsoft.com/zh-cn/library/mt574084.aspx
[Operations for Azure SQL Databases]: https://msdn.microsoft.com/zh-cn/library/azure/dn505719.aspx
[Get-AzureRmSqlDatabase]: https://msdn.microsoft.com/zh-cn/library/mt603648.aspx
[Get-AzureRmSqlDeletedDatabaseBackup]: https://msdn.microsoft.com/zh-cn/library/mt693387.aspx
[Get-AzureRmSqlDatabaseRestorePoints]: https://msdn.microsoft.com/zh-cn/library/mt603642.aspx
[New-AzureRmSqlDatabase]: https://msdn.microsoft.com/zh-cn/library/mt619339.aspx
[Remove-AzureRmSqlDatabase]: https://msdn.microsoft.com/zh-cn/library/mt619368.aspx
[Restore-AzureRmSqlDatabase]: https://msdn.microsoft.com/zh-cn/library/mt693390.aspx
[Resume-AzureRmSqlDatabase]: https://msdn.microsoft.com/zh-cn/library/mt619347.aspx
<!-- It appears that Select-AzureRmSubscription isn't documented, so this points to Select-AzureSubscription -->
[Select-AzureRmSubscription]: https://msdn.microsoft.com/zh-cn/library/dn722499.aspx
[Set-AzureRmSqlDatabase]: https://msdn.microsoft.com/zh-cn/library/mt619433.aspx
[Suspend-AzureRmSqlDatabase]: https://msdn.microsoft.com/zh-cn/library/mt619337.aspx

<!--Other Web references-->

[Microsoft Web Platform Installer]: https://aka.ms/webpi-azps

<!--Update_Description:update meta properties;wording update-->