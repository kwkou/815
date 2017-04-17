<properties
    pageTitle="Azure PowerShell 脚本 - 监视和缩放 SQL 弹性池 | Azure"
    description="Azure PowerShell 脚本示例 - 使用 PowerShell 监视和缩放 SQL 数据库弹性池"
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
    ms.openlocfilehash="a3988f437aa531e7c82ff77121e4339f07541363"
    ms.lasthandoff="04/07/2017" />

# <a name="monitor-and-scale-a-sql-database-elastic-pool-using-powershell"></a>使用 PowerShell 监视和缩放 SQL 数据库弹性池

此示例 PowerShell 脚本监视弹性池的性能指标，将其缩放为更高的性能级别，并基于性能指标之一创建警报规则。 

在运行此脚本前，请确保已使用 `Add-AzureRmAccount` cmdlet 创建与 Azure 的连接。

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

    # Create two elastic database pools
    New-AzureRmSqlElasticPool -ResourceGroupName "myResourceGroup" `
        -ServerName $servername `
        -ElasticPoolName "SamplePool" `
        -Edition "Standard" `
        -Dtu 50 `
        -DatabaseDtuMin 10 `
        -DatabaseDtuMax 50

    # Create two blank database in the pool
    New-AzureRmSqlDatabase  -ResourceGroupName "myResourceGroup" `
        -ServerName $servername `
        -DatabaseName "MyFirstSampleDatabase" `
        -ElasticPoolName "SamplePool"
    New-AzureRmSqlDatabase  -ResourceGroupName "myResourceGroup" `
        -ServerName $servername `
        -DatabaseName "MySecondSampleDatabase" `
        -ElasticPoolName "SamplePool"

    # Monitor the pool
    $MonitorParameters = @{
      ResourceId = "/subscriptions/$($(Get-AzureRMContext).Subscription.SubscriptionId)/resourceGroups/myResourceGroup/providers/Microsoft.Sql/servers/server-$($(Get-AzureRMContext).Subscription.SubscriptionId)/elasticPools/SamplePool"
      TimeGrain = [TimeSpan]::Parse("00:05:00")
      MetricNames = "dtu_consumption_percent"
    }
    (Get-AzureRmMetric @MonitorParameters -DetailedOutput).MetricValues

    # Scale the pool
    Set-AzureRmSqlElasticPool -ResourceGroupName "myResourceGroup" `
        -ServerName $servername `
        -ElasticPoolName "SamplePool" `
        -Edition "Standard" `
        -Dtu 100 `
        -DatabaseDtuMin 20 `
        -DatabaseDtuMax 100

    # Add an alert that fires when the pool utilization reaches 90%
    Add-AzureRMMetricAlertRule -ResourceGroup "myResourceGroup" `
        -Name "MySampleAlertRule" `
        -Location "China East" `
        -TargetResourceId "/subscriptions/$($(Get-AzureRMContext).Subscription.SubscriptionId)/resourceGroups/myResourceGroup/providers/Microsoft.Sql/servers/server-$($(Get-AzureRMContext).Subscription.SubscriptionId)/elasticPools/SamplePool" `
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
| [New-AzureRmSqlElasticPool](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.sql/v2.5.0/new-azurermsqlelasticpool) | 在逻辑服务器中创建弹性池。 |
| [New-AzureRmSqlDatabase](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.sql/v2.5.0/new-azurermsqldatabase) | 在逻辑服务器中创建数据库作为单一数据库或入池数据库。 |
| [Get-AzureRmMetric](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.insights/v2.5.0/get-azurermmetric) | 显示数据库的大小使用情况信息。|
| [Add-AzureRMMetricAlertRule](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.insights/v2.5.0/add-azurermmetricalertrule) | 添加或更新基于指标的警报规则。 |
| [Set-AzureRmSqlElasticPool](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.sql/v2.5.0/set-azurermsqlelasticpool) | 更新弹性池属性 |
| [Add-AzureRMMetricAlertRule](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.insights/v2.5.0/add-azurermmetricalertrule) | 设置警报规则，以便在将来自动监视 DTU。 |
| [Remove-AzureRmResourceGroup](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.resources/v3.5.0/remove-azurermresourcegroup) | 删除资源组，包括所有嵌套的资源。 |
|||

## <a name="next-steps"></a>后续步骤

有关 Azure PowerShell 的详细信息，请参阅 [Azure PowerShell 文档](https://docs.microsoft.com/zh-cn/powershell/)。

可以在 [Azure SQL 数据库 PowerShell 脚本](/documentation/articles/sql-database-powershell-samples/)中找到更多 SQL 数据库 PowerShell 脚本示例。