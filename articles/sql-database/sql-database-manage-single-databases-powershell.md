<properties
    pageTitle="PowerShell：创建和管理单一 Azure SQL 数据库 | Azure"
    description="快速参考：如何使用 Azure 门户预览创建和管理单一 Azure SQL 数据库"
    services="sql-database"
    documentationcenter=""
    author="CarlRabeler"
    manager="jhubbard"
    editor="" />
<tags
    ms.service="sql-database"
    ms.custom="single databases"
    ms.devlang="NA"
    ms.workload="data-management"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.date="02/06/2017"
    wacn.date="03/24/2017"
    ms.author="carlrab" />  


# 使用 PowerShell 创建和管理单一 Azure SQL 数据库

可以使用 [Azure 门户预览](https://portal.azure.cn/)、PowerShell、Transact-SQL、REST API 或 C# 创建和管理单一 Azure SQL 数据库。本主题说明如何使用 PowerShell。对于 Azure 门户预览，请参阅[使用 Azure 门户预览创建和管理单一数据库](/documentation/articles/sql-database-manage-single-databases-powershell/)。有关 Transact-SQL，请参阅[使用 Transact-SQL 创建和管理单一数据库](/documentation/articles/sql-database-manage-single-databases-tsql/)。

[AZURE.INCLUDE [启动 PowerShell 会话](../../includes/sql-database-powershell.md)]

## 使用 PowerShell 创建 Azure SQL 数据库

若要创建 SQL 数据库，请使用 [New-AzureRmSqlDatabase](https://docs.microsoft.com/powershell/resourcemanager/azurerm.sql/v2.3.0/new-azurermsqldatabase) cmdlet。资源组和服务器必须已存在于订阅中。


	$resourceGroupName = "resourcegroup1"
	$sqlServerName = "server1"

	$databaseName = "database1"
	$databaseEdition = "Standard"
	$databaseServiceLevel = "S0"

	$currentDatabase = New-AzureRmSqlDatabase -ResourceGroupName $resourceGroupName `
	 -ServerName $sqlServerName -DatabaseName $databaseName `
	 -Edition $databaseEdition -RequestedServiceObjectiveName $databaseServiceLevel


> [AZURE.TIP]
>有关示例脚本，请参阅[创建 SQL 数据库 PowerShell 脚本](/documentation/articles/sql-database-get-started-powershell/)。
>

##<a name="change-the-service-tier-and-performance-level-of-a-single-database"></a> 更改单一数据库的服务层和性能级别

运行 [Set-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt619433(v=azure.300).aspx) cmdlet 并将 **-RequestedServiceObjectiveName** 设置为所需定价层的性能级别；例如 *S0*、*S1*、*S2*、*S3*、*P1*、*P2*...

将 `{variables}` 替换为自己的值（不含大括号）。


	$SubscriptionId = "{4cac86b0-1e56-bbbb-aaaa-000000000000}"

	$ResourceGroupName = "{resourceGroup}"
	$Location = "{AzureRegion}"

	$ServerName = "{server}"
	$DatabaseName = "{database}"

	$NewEdition = "{Standard}"
	$NewPricingTier = "{S2}"

	Add-AzureRmAccount -EnvironmentName AzureChinaCloud
	Set-AzureRmContext -SubscriptionId $SubscriptionId

	Set-AzureRmSqlDatabase -DatabaseName $DatabaseName -ServerName $ServerName -ResourceGroupName $ResourceGroupName -Edition $NewEdition -RequestedServiceObjectiveName $NewPricingTier


## 后续步骤
* 有关管理工具的概述，请参阅[管理工具概述](/documentation/articles/sql-database-manage-overview/)。
* 若要了解如何使用 Azure 门户预览执行管理任务，请参阅[使用 Azure 门户预览管理 Azure SQL 数据库](/documentation/articles/sql-database-manage-portal/)。
* 若要了解如何使用 PowerShell 执行管理任务，请参阅[使用 PowerShell 管理 Azure SQL 数据库](/documentation/articles/sql-database-manage-powershell/)。
* 若要了解如何使用 SQL Server Management Studio 执行管理任务，请参阅 [SQL Server Management Studio](/documentation/articles/sql-database-manage-azure-ssms/)。
* 有关 SQL 数据库服务的信息，请参阅[什么是 SQL 数据库](/documentation/articles/sql-database-technical-overview/)。
* 有关 Azure 数据库服务器和数据库功能的信息，请参阅[功能](/documentation/articles/sql-database-features/)。

<!---HONumber=Mooncake_0320_2017-->