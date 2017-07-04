<properties
    pageTitle="Azure PowerShell 脚本 - 设置异地复制 - 单个 SQL 数据库 | Azure"
    description="Azure PowerShell 脚本示例 - 使用 PowerShell 为单个 Azure SQL 数据库设置活动异地复制"
    services="sql-database"
    documentationcenter="sql-database"
    author="janeng"
    manager="jstrauss"
    editor="carlrab"
    tags="azure-service-management"
    translationtype="Human Translation" />
<tags
    ms.assetid=""
    ms.service="sql-database"
    ms.custom="sample"
    ms.devlang="PowerShell"
    ms.topic="article"
    ms.tgt_pltfrm="sql-database"
    ms.workload="database"
    ms.date="03/07/2017"
    wacn.date="04/17/2017"
    ms.author="janeng"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="2110f5ee5cbf224535ac734f12c4d1f185a4486c"
    ms.lasthandoff="04/07/2017" />

# <a name="configure-active-geo-replication-for-a-single-azure-sql-database-using-powershell"></a>使用 PowerShell 为单个 Azure SQL 数据库配置活动异地复制

此示例 PowerShell 脚本为单个数据库配置活动异地复制，并将其故障转移到辅助副本。

在运行此脚本前，请确保已使用 `Add-AzureRmAccount` cmdlet 创建与 Azure 的连接。

## <a name="sample-scripts"></a>示例脚本

    # Set an admin login and password for your database
    $adminlogin = "ServerAdmin"
    $password = "ChangeYourAdminPassword1"
    # The logical server names have to be unique in the system
    $primaryservername = "primary-server-$($(Get-AzureRMContext).Subscription.SubscriptionId)"
    $sercondaryservername = "secondary-server-$($(Get-AzureRMContext).Subscription.SubscriptionId)"

    # Create two new resource groups
    New-AzureRmResourceGroup -Name "myPrimaryResourceGroup" -Location "China East"
    New-AzureRmResourceGroup -Name "mySecondaryResourceGroup" -Location "China North"

    # Create two new logical servers with a system wide unique server name
    New-AzureRmSqlServer -ResourceGroupName "myPrimaryResourceGroup" `
        -ServerName $primaryservername `
        -Location "China East" `
        -SqlAdministratorCredentials $(New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $adminlogin, $(ConvertTo-SecureString -String $password -AsPlainText -Force))
    New-AzureRmSqlServer -ResourceGroupName "mySecondaryResourceGroup" `
        -ServerName $sercondaryservername `
        -Location "China North" `
        -SqlAdministratorCredentials $(New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $adminlogin, $(ConvertTo-SecureString -String $password -AsPlainText -Force))

    # Create a blank database with S0 performance level on the primary server
    New-AzureRmSqlDatabase  -ResourceGroupName "myPrimaryResourceGroup" `
        -ServerName $primaryservername `
        -DatabaseName "MySampleDatabase" -RequestedServiceObjectiveName "S0"

    # Establish Active Geo-Replication
    $myDatabase = Get-AzureRmSqlDatabase -DatabaseName "MySampleDatabase" -ResourceGroupName "myPrimaryResourceGroup" -ServerName $primaryservername
    $myDatabase | New-AzureRmSqlDatabaseSecondary -PartnerResourceGroupName "mySecondaryResourceGroup" -PartnerServerName $sercondaryservername -AllowConnections "All"

    # Initiate a planned failover
    $myDatabase = Get-AzureRmSqlDatabase -DatabaseName "MySampleDatabase" -ResourceGroupName "mySecondaryResourceGroup" -ServerName $sercondaryservername
    $myDatabase | Set-AzureRmSqlDatabaseSecondary -PartnerResourceGroupName "myPrimaryResourceGroup" -Failover

    # Monitor Geo-Replication config and health after failover
    $myDatabase = Get-AzureRmSqlDatabase -DatabaseName "MySampleDatabase" -ResourceGroupName "mySecondaryResourceGroup" -ServerName $sercondaryservername
    $myDatabase | Get-AzureRmSqlDatabaseReplicationLink -PartnerResourceGroupName "myPrimaryResourceGroup" -PartnerServerName $primaryservername

    # Remove the replication link after the failover
    $myDatabase = Get-AzureRmSqlDatabase -DatabaseName "MySampleDatabase" -ResourceGroupName "mySecondaryResourceGroup" -ServerName $sercondaryservername
    $secondaryLink = $myDatabase | Get-AzureRmSqlDatabaseReplicationLink -PartnerResourceGroupName "myPrimaryResourceGroup" -PartnerServerName $primaryservername
    $secondaryLink | Remove-AzureRmSqlDatabaseSecondary

## <a name="clean-up-deployment"></a>清理部署

运行脚本示例后，可以使用以下命令删除资源组以及与其关联的所有资源。

    Remove-AzureRmResourceGroup -ResourceGroupName "myResourceGroup"

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [New-AzureRmResourceGroup](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.resources/v3.5.0/new-azurermresourcegroup) | 创建用于存储所有资源的资源组。 |
| [New-AzureRmSqlServer](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.sql/v2.5.0/new-azurermsqlserver) | 创建用于托管数据库或弹性池的逻辑服务器。 |
| [New-AzureRmSqlElasticPool](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.sql/v2.5.0/new-azurermsqlelasticpool) | 在逻辑服务器中创建弹性池。 |
| [Set-AzureRmSqlDatabase](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.sql/v2.5.0/set-azurermsqldatabase) | 更新数据库属性，或者将数据库移入、移出弹性池或在弹性池之间移动。 |
| [New-AzureRmSqlDatabaseSecondary](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.sql/v2.5.0/new-azurermsqldatabasesecondary)| 为现有数据库创建辅助数据库，并开始数据复制。 |
| [Get-AzureRmSqlDatabase](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.sql/v2.5.0/get-azurermsqldatabase)| 获取一个或多个数据库。 |
| [Set-AzureRmSqlDatabaseSecondary](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.sql/v2.5.0/set-azurermsqldatabasesecondary)| 将辅助数据库切换为主数据库，启动故障转移。|
| [Get-AzureRmSqlDatabaseReplicationLink](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.sql/v2.5.0/get-azurermsqldatabasereplicationlink) | 获取 Azure SQL 数据库和资源组或 SQL Server 之间的异地复制链路。 |
| [Remove-AzureRmSqlDatabaseSecondary](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.sql/v2.5.0/remove-azurermsqldatabasesecondary) | 终止 SQL 数据库和指定的辅助数据库之间的数据复制。 |
| [Remove-AzureRmResourceGroup](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.resources/v3.5.0/remove-azurermresourcegroup) | 删除资源组，包括所有嵌套的资源。 |
|||

## <a name="next-steps"></a>后续步骤

有关 Azure PowerShell 的详细信息，请参阅 [Azure PowerShell 文档](https://docs.microsoft.com/zh-cn/powershell/)。

可以在 [Azure SQL 数据库 PowerShell 脚本](/documentation/articles/sql-database-powershell-samples/)中找到更多 SQL 数据库 PowerShell 脚本示例。