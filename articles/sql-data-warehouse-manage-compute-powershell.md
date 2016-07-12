<properties
   pageTitle="管理 Azure SQL 数据仓库中的计算能力 (PowerShell) | Azure"
   description="用于管理计算能力的 PowerShell 任务。通过调整 DWU 缩放计算资源。或者，暂停和恢复计算资源来节省成本。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="barbkess"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="05/06/2016"
   wacn.date="06/13/2016"/>

# 管理 Azure SQL 数据仓库中的计算能力 (PowerShell)

> [AZURE.SELECTOR]
- [概述](/documentation/articles/sql-data-warehouse-overview-manage-compute/)
- [PowerShell](/documentation/articles/sql-data-warehouse-manage-compute-powershell/)
- [REST](/documentation/articles/sql-data-warehouse-manage-compute-rest-api/)
- [TSQL](/documentation/articles/sql-data-warehouse-manage-compute-tsql/)


通过扩大计算资源和内存来提升性能，从而满足工作负荷不断变化的需求。通过在非高峰时段缩减资源或同时暂停计算来节省成本。

此任务集合使用 Powershell 实现：

- 缩放计算
- 暂停计算
- 恢复计算

若要了解相关信息，请参阅 [管理计算能力概述][]。


## 开始之前

### 安装最新版本的 Azure PowerShell

> [AZURE.NOTE]  若要对 SQL 数据仓库使用 Azure PowerShell，需要安装 Azure PowerShell 1.0.3 或更高版本。若要验证当前版本，请运行命令 **Get-Module -ListAvailable -Name Azure**。可以从 [Microsoft Web 平台安装程序][] 安装最新的版本。有关详细信息，请参阅 [如何安装和配置 Azure PowerShell][]。

### Azure PowerShell cmdlet 入门

开始操作：

1. 打开 Azure PowerShell。 
2. 在 PowerShell 提示符下，运行以下命令以登录到 Azure Resource Manager，然后选择你的订阅。

        Login-AzureRmAccount -EnvironmentName AzureChinaCloud
        Get-AzureRmSubscription
        Select-AzureRmSubscription -SubscriptionName "MySubscription"

<a name="scale-performance-bk"></a>
<a name="scale-compute-bk"></a>

## 缩放计算能力

[AZURE.INCLUDE [SQL Data Warehouse scale DWUs description（SQL 数据仓库缩放 DWU 说明）](../includes/sql-data-warehouse-scale-dwus-description)]

若要更改 DWU，请使用 [Set-AzureRmSqlDatabase][] PowerShell cmdlet。以下示例将托管在服务器 MyServer 上的数据库 MySQLDW 的服务级别目标设置为 DW1000。

    Set-AzureRmSqlDatabase -DatabaseName "MySQLDW" -ServerName "MyServer" -RequestedServiceObjectiveName "DW1000"

<a name="pause-compute-bk"></a>

## 暂停计算

[AZURE.INCLUDE [SQL Data Warehouse pause description（SQL 数据仓库暂停说明）](../includes/sql-data-warehouse-pause-description)]

若要暂停数据库，请使用 [Suspend-AzureRmSqlDatabase][] cmdlet。以下示例将暂停 Server01 服务器上托管的 Database02 数据库。该服务器位于名为 ResourceGroup1 的 Azure 资源组中。

> [AZURE.NOTE] 注意，如果服务器是 foo.database.chinacloudapi.cn，请使用“foo”作为 Powershell cmdlet 中的 -ServerName。

    Suspend-AzureRmSqlDatabase –ResourceGroupName "ResourceGroup1" `
    –ServerName "Server01" –DatabaseName "Database02"

一种变异，下一个示例将数据库检索到 $database 对象中。然后，它通过管道将该对象传递给 [Suspend-AzureRmSqlDatabase][]。结果存储在对象 resultDatabase 中。最后一个命令显示结果。

    $database = Get-AzureRmSqlDatabase –ResourceGroupName "ResourceGroup1" `
    –ServerName "Server01" –DatabaseName "Database02"
    $resultDatabase = $database | Suspend-AzureRmSqlDatabase
    $resultDatabase

<a name="resume-compute-bk"></a>

## 恢复计算

[AZURE.INCLUDE [SQL Data Warehouse resume description（SQL 数据仓库恢复说明）](../includes/sql-data-warehouse-resume-description)]

若要启动数据库，请使用 [Resume-AzureRmSqlDatabase][] cmdlet。以下示例将启动 Server01 服务器上托管的 Database02 数据库。该服务器位于名为 ResourceGroup1 的 Azure 资源组中。

    Resume-AzureRmSqlDatabase –ResourceGroupName "ResourceGroup1" `
    –ServerName "Server01" -DatabaseName "Database02"

一种变异，下一个示例将数据库检索到 $database 对象中。然后，它通过管道将对象传递给 [Resume-AzureRmSqlDatabase][]，并将结果存储在 $resultDatabase 中。最后一个命令显示结果。

    $database = Get-AzureRmSqlDatabase –ResourceGroupName "ResourceGroup1" `
    –ServerName "Server01" –DatabaseName "Database02"
    $resultDatabase = $database | Resume-AzureRmSqlDatabase
    $resultDatabase

<a name="next-steps-bk"></a>

## 后续步骤

有关其他管理任务，请参阅[管理概述][]。

<!--Image references-->

<!--Article references-->
[Service capacity limits]: /documentation/articles/sql-data-warehouse-service-capacity-limits/
[管理概述]: /documentation/articles/sql-data-warehouse-overview-manage/

<!--MSDN references-->
[Resume-AzureRmSqlDatabase]: https://msdn.microsoft.com/zh-cn/library/mt619347.aspx
[Suspend-AzureRmSqlDatabase]: https://msdn.microsoft.com/zh-cn/library/mt619337.aspx
[Set-AzureRmSqlDatabase]: https://msdn.microsoft.com/zh-cn/library/mt619433.aspx


<!--Other Web references-->

[Azure portal]: http://manage.windowsazure.cn/

<!---HONumber=Mooncake_0606_2016-->