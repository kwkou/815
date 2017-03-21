<properties
    pageTitle="管理 Azure SQL 数据仓库中的计算能力 (PowerShell) | Azure"
    description="用于管理计算能力的 PowerShell 任务。通过调整 DWU 缩放计算资源。或者，暂停和恢复计算资源来节省成本。"
    services="sql-data-warehouse"
    documentationcenter="NA"
    author="barbkess"
    manager="jhubbard"
    editor="" />
<tags
    ms.assetid="8354a3c1-4e04-4809-933f-db414a8c74dc"
    ms.service="sql-data-warehouse"
    ms.devlang="NA"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="data-services"
    ms.date="10/31/2016"
    wacn.date="03/20/2017"
    ms.author="barbkess" />  

# 管理 Azure SQL 数据仓库中的计算能力 (PowerShell)

> [AZURE.SELECTOR]
- [概述](/documentation/articles/sql-data-warehouse-manage-compute-overview/)
- [门户](/documentation/articles/sql-data-warehouse-manage-compute-portal/)
- [PowerShell](/documentation/articles/sql-data-warehouse-manage-compute-powershell/)
- [REST](/documentation/articles/sql-data-warehouse-manage-compute-rest-api/)
- [TSQL](/documentation/articles/sql-data-warehouse-manage-compute-tsql/)

## 开始之前

### 安装最新版本的 Azure PowerShell
> [AZURE.NOTE]
> 若要对 SQL 数据仓库使用 Azure PowerShell，需要安装 Azure PowerShell 1.0.3 或更高版本。若要验证当前版本，请运行命令 **Get-Module -ListAvailable -Name Azure**。可以从 [Microsoft Web 平台安装程序][Microsoft Web Platform Installer]安装最新的版本。有关详细信息，请参阅[如何安装和配置 Azure PowerShell][How to install and configure Azure PowerShell]。
> 
> 

### Azure PowerShell cmdlet 入门

开始操作：

1. 打开 Azure PowerShell。
2. 在 PowerShell 提示符下，运行以下命令以登录到 Azure Resource Manager，然后选择你的订阅。

        Login-AzureRmAccount -EnvironmentName AzureChinaCloud
        Get-AzureRmSubscription
        Select-AzureRmSubscription -SubscriptionName "MySubscription"

## <a name="scale-performance-bk"></a><a name="scale-compute-bk"></a> 缩放计算能力

[AZURE.INCLUDE [SQL Data Warehouse scale DWUs description（SQL 数据仓库缩放 DWU 说明）](../../includes/sql-data-warehouse-scale-dwus-description.md)]

若要更改 DWU，请使用 [Set-AzureRmSqlDatabase][Set-AzureRmSqlDatabase] PowerShell cmdlet。以下示例将托管在服务器 MyServer 上的数据库 MySQLDW 的服务级别目标设置为 DW1000。

    Set-AzureRmSqlDatabase -DatabaseName "MySQLDW" -ServerName "MyServer" -RequestedServiceObjectiveName "DW1000"

## <a name="pause-compute-bk"></a> 暂停计算

[AZURE.INCLUDE [SQL Data Warehouse pause description（SQL 数据仓库暂停说明）](../../includes/sql-data-warehouse-pause-description.md)]

若要暂停数据库，请使用 [Suspend-AzureRmSqlDatabase][] cmdlet。以下示例将暂停 Server01 服务器上托管的 Database02 数据库。该服务器位于名为 ResourceGroup1 的 Azure 资源组中。

> [AZURE.NOTE]
> 注意，如果服务器是 foo.database.chinacloudapi.cn，请使用“foo”作为 PowerShell cmdlet 中的 -ServerName。
> 
> 

    Suspend-AzureRmSqlDatabase -ResourceGroupName "ResourceGroup1" `
    -ServerName "Server01" -DatabaseName "Database02"

一种变异，下一个示例将数据库检索到 $database 对象中。然后，它通过管道将该对象传递给 [Suspend-AzureRmSqlDatabase][Suspend-AzureRmSqlDatabase]。结果存储在对象 resultDatabase 中。最后一个命令显示结果。

    $database = Get-AzureRmSqlDatabase -ResourceGroupName "ResourceGroup1" `
    -ServerName "Server01" -DatabaseName "Database02"
    $resultDatabase = $database | Suspend-AzureRmSqlDatabase
    $resultDatabase

## <a name="resume-compute-bk"></a> 恢复计算

[AZURE.INCLUDE [SQL Data Warehouse resume description（SQL 数据仓库恢复说明）](../../includes/sql-data-warehouse-resume-description.md)]

若要启动数据库，请使用 [Resume-AzureRmSqlDatabase][Resume-AzureRmSqlDatabase] cmdlet。以下示例将启动 Server01 服务器上托管的 Database02 数据库。该服务器位于名为 ResourceGroup1 的 Azure 资源组中。

    Resume-AzureRmSqlDatabase -ResourceGroupName "ResourceGroup1" `
    -ServerName "Server01" -DatabaseName "Database02"

一种变异，下一个示例将数据库检索到 $database 对象中。然后，它通过管道将对象传递给 [Resume-AzureRmSqlDatabase][Resume-AzureRmSqlDatabase]，并将结果存储在 $resultDatabase 中。最后一个命令显示结果。

    $database = Get-AzureRmSqlDatabase -ResourceGroupName "ResourceGroup1" `
    -ServerName "Server01" -DatabaseName "Database02"
    $resultDatabase = $database | Resume-AzureRmSqlDatabase
    $resultDatabase

## <a name="next-steps-bk"></a> 后续步骤

有关其他管理任务，请参阅[管理概述][]。

<!--Image references-->

<!--Article references-->
[Service capacity limits]: /documentation/articles/sql-data-warehouse-service-capacity-limits/
[管理概述]: /documentation/articles/sql-data-warehouse-overview-manage/
[How to install and configure Azure PowerShell]: https://docs.microsoft.com/powershell/azureps-cmdlets-docs/
[Manage compute overview]: /documentation/articles/sql-data-warehouse-manage-compute-overview/

<!--MSDN references-->
[Resume-AzureRmSqlDatabase]: https://msdn.microsoft.com/zh-cn/library/mt619347.aspx
[Suspend-AzureRmSqlDatabase]: https://msdn.microsoft.com/zh-cn/library/mt619337.aspx
[Set-AzureRmSqlDatabase]: https://msdn.microsoft.com/zh-cn/library/mt619433.aspx

<!--Other Web references-->

[Microsoft Web Platform Installer]: https://www.microsoft.com/web/downloads/platform.aspx
[Azure portal]: http://portal.azure.cn/

<!---HONumber=Mooncake_0313_2017-->
<!--Update_Description:update meta properties;wording update-->