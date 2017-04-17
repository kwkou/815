<properties
    pageTitle="Azure PowerShell 脚本 - 监视和缩放单个 SQL 数据库 | Azure"
    description="Azure PowerShell 脚本示例 - 使用 PowerShell 监视和缩放单个 SQL 数据库"
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
    ms.openlocfilehash="23ec7b49b923dcb3fa9bc33ae21ea5fee99db11f"
    ms.lasthandoff="04/07/2017" />

# <a name="monitor-and-scale-a-single-sql-database-using-powershell"></a>使用 PowerShell 监视和缩放单个 SQL 数据库

此示例 PowerShell 脚本监视一个数据库的性能指标，将其缩放为更高的性能级别，并基于性能指标之一创建警报规则。 

在运行此脚本前，请确保已使用 `Add-AzureRmAccount` cmdlet 创建了到 Azure 的连接。

## <a name="sample-script"></a>示例脚本

    # Set an admin login and password for your database
    $adminlogin = "ServerAdmin"
    $password = "ChangeYourAdminPassword1"
    # The logical server name has to be unique in the system
    $servername = "server-$($(Get-AzureRMContext).Subscription.SubscriptionId)"

    # Create a new resource group
    New-AzureRmResourceGroup -Name "myResourceGroup" -Location "China East"

    # Create a new server with a system wide unique server name
    New-AzureRmSqlServer -ResourceGroupName "myResourceGroup" `
        -ServerName $servername `
        -Location "China East" `
        -SqlAdministratorCredentials $(New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $adminlogin, $(ConvertTo-SecureString -String $password -AsPlainText -Force))

    # Create a blank database with S0 performance level
    New-AzureRmSqlDatabase  -ResourceGroupName "myResourceGroup" `
        -ServerName $servername `
        -DatabaseName "MySampleDatabase" -RequestedServiceObjectiveName "S0"

    # Monitor the DTU consumption on the imported database in 5 minute intervals
    $MonitorParameters = @{
      ResourceId = "/subscriptions/$($(Get-AzureRMContext).Subscription.SubscriptionId)/resourceGroups/myResourceGroup/providers/Microsoft.Sql/servers/server-$($(Get-AzureRMContext).Subscription.SubscriptionId)/databases/MySampleDatabase"
      TimeGrain = [TimeSpan]::Parse("00:05:00")
      MetricNames = "dtu_consumption_percent"
    }
    (Get-AzureRmMetric @MonitorParameters -DetailedOutput).MetricValues

    # Scale the database performance to Standard S2
    Set-AzureRmSqlDatabase -ResourceGroupName "myResourceGroup" `
        -ServerName $servername `
        -DatabaseName "MySampleDatabase" `
        -Edition "Standard" `
        -RequestedServiceObjectiveName "S1"

    # Set an alert rule to automatically monitor DTU in the future
    Add-AzureRMMetricAlertRule -ResourceGroup "myResourceGroup" `
        -Name "MySampleAlertRule" `
        -Location "China East" `
        -TargetResourceId "/subscriptions/$($(Get-AzureRMContext).Subscription.SubscriptionId)/resourceGroups/myResourceGroup/providers/Microsoft.Sql/servers/server-$($(Get-AzureRMContext).Subscription.SubscriptionId)/databases/MySampleDatabase" `
        -MetricName "dtu_consumption_percent" `
        -Operator "GreaterThan" `
        -Threshold 90 `
        -WindowSize $([TimeSpan]::Parse("00:05:00")) `
        -TimeAggregationOperator "Average" `
        -Actions $(New-AzureRmAlertRuleEmail -SendToServiceOwners)

## <a name="clean-up-deployment"></a>清理部署

运行脚本示例后，可以使用以下命令删除资源组以及与其关联的所有资源。

    Remove-AzureRmResourceGroup -ResourceGroupName "myResourceGroup"

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
 [New-AzureRmResourceGroup](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.resources/v3.5.0/new-azurermresourcegroup) | 创建用于存储所有资源的资源组。 |
| [New-AzureRmSqlServer](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.sql/v2.5.0/new-azurermsqlserver) | 创建用于托管数据库或弹性池的逻辑服务器。 |
| [Get-AzureRmMetric](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.insights/v2.5.0/get-azurermmetric) | 显示数据库的大小使用情况信息。|
| [Set-AzureRmSqlDatabase](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.sql/v2.5.0/set-azurermsqldatabase) | 更新数据库属性，或者将数据库移入、移出弹性池或在弹性池之间移动。 |
| [Add-AzureRMMetricAlertRule](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.insights/v2.5.0/add-azurermmetricalertrule) | 设置警报规则，以便在将来自动监视 DTU。 |
| [Remove-AzureRmResourceGroup](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.resources/v3.5.0/remove-azurermresourcegroup) | 删除资源组，包括所有嵌套的资源。 |
|||

## <a name="next-steps"></a>后续步骤

有关 Azure PowerShell 的详细信息，请参阅 [Azure PowerShell 文档](https://docs.microsoft.com/zh-cn/powershell/)。

可以在 [Azure SQL 数据库 PowerShell 脚本](/documentation/articles/sql-database-powershell-samples/)中找到更多 SQL 数据库 PowerShell 脚本示例。