<properties 
    pageTitle="使用 PowerShell 更改 Azure SQL 数据库的服务层和性能级别" 
    description="“更改 Azure SQL 数据库的服务层和性能级别”介绍如何使用 PowerShell 扩展和缩减 SQL 数据库。使用 PowerShell 更改 Azure SQL 数据库定价层。" 
	services="sql-database"
	documentationCenter=""
	authors="stevestein"
	manager="jeffreyg"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.date="10/08/2015"
	wacn.date="01/11/2016"/>


# 使用 PowerShell 更改 SQL 数据库的服务层和性能级别（定价层）

**单一数据库**

> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/sql-database-scale-up-powershell)


本文介绍如何使用 PowerShell 更改 SQL 数据库的服务层和性能级别。

使用[将 SQL 数据库 Web/企业数据库升级到新服务层](/documentation/articles/sql-database-upgrade-new-service-tiers)和 [Azure SQL 数据库服务层和性能级别](/documentation/articles/sql-database-service-tiers)中的信息为 Azure SQL 数据库确定适当的服务层和性能级别。

> [AZURE.IMPORTANT]更改 SQL 数据库的服务层和性能级别是一项联机操作。这意味着在整个操作期间数据库将保持联机并可用而没有停机时间。

- 若要对数据库进行降级，数据库应小于目标服务层允许的最大大小。 
- 在启用了[标准异地复制](/documentation/articles/sql-database-business-continuity-design)或[活动异地复制](/documentation/articles/sql-database-geo-replication-overview/)的情况下升级数据库时，必须先将次要数据库升级到所需的性能层，然后再升级主数据库。
- 从高级服务层降级时，必须先终止所有异地复制关系。你可以按照[终止连续复制关系](/documentation/articles/sql-database-disaster-recovery/)主题中所述的步骤停止主数据库与活动次要数据库之间的复制过程。
- 各服务层提供的还原服务是不同的。如果进行降级，你可能无法再还原到某个时间点，或者备份保留期变短。有关详细信息，请参阅 [Azure SQL 数据库备份和还原](/documentation/articles/sql-database-business-continuity/)。
- 你可以在 24 小时内进行最多四项单独的数据库更改（服务层或性能级别）。
- 所做的更改完成之前不会应用数据库的新属性。



**若要完成本文，你需要以下各项：**

- Azure 订阅。如果你需要 Azure 订阅，只需单击本页顶部的“免费试用”，然后再回来完成本文的相关操作即可。
- Azure SQL 数据库。如果你没有 SQL 数据库，请按照[创建你的第一个 Azure SQL 数据库](/documentation/articles/sql-database-get-started)文章中的步骤创建一个。
- Azure PowerShell。

> [AZURE.IMPORTANT]从 Azure PowerShell 1.0 预览版开始，Switch-AzureMode cmdlet 不再可用，并且 Azure ResourceManger 模块中的 cmdlet 已重命名。本文中的示例使用新的 PowerShell 1.0 预览版命名约定。有关详细信息，请参阅[弃用 Azure PowerShell 中的 Switch-AzureMode](https://github.com/Azure/azure-powershell/wiki/Deprecation-of-Switch-AzureMode-in-Azure-PowerShell)。


若要运行 PowerShell cmdlet，需要安装并运行 Azure PowerShell，而且由于 Switch-AzureMode 已删除，你应该通过运行 [Microsoft Web 平台安装程序](http://go.microsoft.com/fwlink/p/?linkid=320376&clcid=0x409)来下载并安装最新的 Azure PowerShell。有关详细信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure)。



## 配置你的凭据，然后选择你的订阅

首先必须与 Azure 帐户建立访问连接，因此请启动 PowerShell，然后运行以下 cmdlet。在登录屏幕中，输入登录 Azure 门户时所用的相同电子邮件和密码。

	Add-AzureAccount -Environment AzureChinaCloud

成功登录后，你会在屏幕上看到一些信息，其中包括你登录时使用的 ID，以及你有权访问的 Azure 订阅。


### 选择 Azure 订阅

若要选择订阅，你需要提供订阅 ID 或订阅名称 (**-SubscriptionName**)。你可以从前面的步骤中显示的信息中复制订阅 ID，或者，如果你有多个订阅且需要更多详细信息，可以运行 **Get-AzureSubscription** cmdlet，然后从结果集中复制所需的订阅信息。获得订阅以后，你可以运行以下 cmdlet：

	$SubscriptionId = "4cac86b0-1e56-bbbb-aaaa-000000000000"
    Select-AzureSubscription -SubscriptionId $SubscriptionId

成功运行 **Select-AzureSubscription** 后，将返回到 PowerShell 提示符处。如果你有多个订阅，可以运行 **Get-AzureSubscription** 并验证要使用的订阅显示 **IsCurrent: True**。


 


## 更改 SQL 数据库的服务层和性能级别

运行 **Set-AzureRMSqlDatabase** cmdlet 并将 **-RequestedServiceObjectiveName** 设置为所需定价层的性能级别；例如 *S0*、*S1*、*S2*、*S3*、*P1*、*P2*...

    $ResourceGroupName = "resourceGroupName"
    
    $ServerName = "serverName"
    $DatabaseName = "databaseName"

    $NewEdition = "Standard"
    $NewPricingTier = "S2"

    $ScaleRequest = Set-AzureSqlDatabase -DatabaseName $DatabaseName -ServerName $ServerName -ResourceGroupName $ResourceGroupName -Edition $NewEdition -RequestedServiceObjectiveName $NewPricingTier


  

   


## 用于更改 SQL 数据库的服务层和性能级别的示例 PowerShell 脚本

    

    
    $SubscriptionId = "4cac86b0-1e56-bbbb-aaaa-000000000000"
    
    $ResourceGroupName = "resourceGroupName"
    $Location = "Japan West"
    
    $ServerName = "serverName"
    $DatabaseName = "databaseName"
    
    $NewEdition = "Standard"
    $NewPricingTier = "S2"
    
    Add-AzureAccount -Environment AzureChinaCloud
    Select-AzureSubscription -SubscriptionId $SubscriptionId
    
    $ScaleRequest = Set-AzureSqlDatabase -DatabaseName $DatabaseName -ServerName $ServerName -ResourceGroupName $ResourceGroupName -Edition $NewEdition -RequestedServiceObjectiveName $NewPricingTier
    
    $ScaleRequest
    
        


## 后续步骤

- [扩大和缩小](/documentation/articles/sql-database-elastic-scale-get-started)
- [使用 SSMS 连接和查询 SQL 数据库](/documentation/articles/sql-database-connect-query-ssms)
- [导出 Azure SQL 数据库](/documentation/articles/sql-database-export-powershell)

## 其他资源

- [业务连续性概述](/documentation/articles/sql-database-business-continuity)
- [SQL 数据库文档](/services/sql-databases/)
- [Azure SQL 数据库 Cmdlet](https://msdn.microsoft.com/zh-cn/library/azure/mt163521.aspx)

<!---HONumber=82-->