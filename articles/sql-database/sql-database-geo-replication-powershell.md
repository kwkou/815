<properties 
    pageTitle="使用 PowerShell 为 Azure SQL 数据库配置活动异地复制 | Azure" 
    description="使用 PowerShell 为 Azure SQL 数据库配置活动异地复制" 
    services="sql-database" 
    documentationCenter="" 
    authors="stevestein" 
    manager="jhubbard" 
    editor=""/>

<tags
    ms.service="sql-database"
    ms.devlang="NA"
    ms.topic="article"
    ms.tgt_pltfrm="powershell"
   ms.workload="NA"
    ms.date="07/14/2016"
    wacn.date="09/19/2016"
    ms.author="sstein"/>

# 使用 PowerShell 为 Azure SQL 数据库配置异地复制

> [AZURE.SELECTOR]
- [概述](/documentation/articles/sql-database-geo-replication-overview/)
- [PowerShell](/documentation/articles/sql-database-geo-replication-powershell/)
- [T-SQL](/documentation/articles/sql-database-geo-replication-transact-sql/)

本文说明如何使用 PowerShell 为 SQL 数据库配置活动异地复制。

若要使用 PowerShell 启动故障转移，请参阅[使用 PowerShell 为 Azure SQL 数据库启动计划内或计划外故障转移](/documentation/articles/sql-database-geo-replication-failover-powershell/)。

>[AZURE.NOTE] 活动异地复制（可读辅助数据库）现在可供所有服务层中的所有数据库使用。非可读辅助类型将在 2017 年 4 月停用，现有的非可读数据库将自动升级到可读辅助数据库。



若要使用 PowerShell 配置活动异地复制，需提供：

- Azure 订阅。
- Azure SQL 数据库 - 要复制的主数据库。
- Azure PowerShell 1.0 或更高版本。根据遵循[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/) 来下载并安装 Azure PowerShell 模块。


## 配置你的凭据，然后选择你的订阅

首先必须与 Azure 帐户建立访问连接，因此请启动 PowerShell，然后运行以下 cmdlet。在登录屏幕中，输入登录 Azure 门户时所用的相同电子邮件和密码。


	Login-AzureRmAccount -EnvironmentName AzureChinaCloud

成功登录后，屏幕上将显示一些信息，包括登录时所用的 ID 和有权访问的 Azure 订阅。


### 选择 Azure 订阅

需要订阅 ID 才可选择订阅。可复制上一步所示信息中的订阅 ID；如果具有多个订阅且需要更多详细信息，可运行 **Get-AzureRmSubscription** cmdlet 并复制结果集中的所需订阅信息。以下 cmdlet 使用订阅 ID 来设置当前订阅：

	Select-AzureRmSubscription -SubscriptionId 4cac86b0-1e56-bbbb-aaaa-000000000000

成功运行 **Select-AzureRmSubscription** 后，将返回到 PowerShell 提示符处。


## 添加辅助数据库


以下步骤在异地复制合作关系中创建新的辅助数据库。
  
只有订阅所有者或共同所有者才可启用辅助库。

可以使用 **New-AzureRmSqlDatabaseSecondary** cmdlet，将伙伴服务器上的辅助数据库添加到所连接的服务器上的本地数据库（主数据库）。

此 cmdlet 将 **Start-AzureSqlDatabaseCopy** 替换为 **-IsContinuous** 参数。它将输出可供其他 cmdlet 用于明确识别特定复制链接的 **AzureRmSqlDatabaseSecondary** 对象。创建辅助数据库并完全设定种子后，此 cmdlet 将返回。根据数据库的大小，这可能需要花费数分钟到数小时的时间。

辅助服务器上的复制数据库具备与主要服务器上的数据库相同的名称，并且默认具有相同的服务级别。辅助数据库可以是可读或不可读，并且可以是单一数据库或弹性数据库。有关详细信息，请参阅 [New-AzureRMSqlDatabaseSecondary](https://msdn.microsoft.com/zh-cn/library/mt603689.aspx) 和[服务层](/documentation/articles/sql-database-service-tiers/)。
创建辅助数据库并设定种子之后，开始将数据从主数据库复制到新的辅助数据库。以下步骤说明如何使用 PowerShell 完成这项任务，以使用单一数据库或弹性数据库来创建不可读和可读的辅助数据库。

如果伙伴数据库已存在（例如，在终止旧的异地复制关系的情况下），命令将会失败。



### 添加不可读的辅助数据库（单一数据库）

以下命令在资源组“rg2”中的服务器“srv2”上创建数据库“mydb”的不可读辅助数据库：

    $database = Get-AzureRmSqlDatabase –DatabaseName "mydb" -ResourceGroupName "rg1" -ServerName "srv1"
    $secondaryLink = $database | New-AzureRmSqlDatabaseSecondary –PartnerResourceGroupName "rg2" –PartnerServerName "srv2" -AllowConnections "No"



### 添加可读的辅助数据库（单一数据库）

以下命令在资源组“rg2”中的服务器“srv2”上创建数据库“mydb”的可读辅助数据库：

    $database = Get-AzureRmSqlDatabase –DatabaseName "mydb" -ResourceGroupName "rg1" -ServerName "srv1"
    $secondaryLink = $database | New-AzureRmSqlDatabaseSecondary –PartnerResourceGroupName "rg2" –PartnerServerName "srv2" -AllowConnections "All"




### 添加不可读的辅助数据库（弹性数据库）

以下命令在资源组“rg2”服务器“srv2”中名为“ElasticPool1”的弹性数据库池内创建数据库“mydb”的不可读辅助数据库：

    $database = Get-AzureRmSqlDatabase –DatabaseName "mydb" -ResourceGroupName "rg1" -ServerName "srv1"
    $secondaryLink = $database | New-AzureRmSqlDatabaseSecondary –PartnerResourceGroupName "rg2" –PartnerServerName "srv2" –SecondaryElasticPoolName "ElasticPool1" -AllowConnections "No"


### 添加可读的辅助数据库（弹性数据库）

以下命令在资源组“rg2”服务器“srv2”中名为“ElasticPool1”的弹性数据库池内创建数据库“mydb”的可读辅助数据库：

    $database = Get-AzureRmSqlDatabase –DatabaseName "mydb" -ResourceGroupName "rg1" -ServerName "srv1"
    $secondaryLink = $database | New-AzureRmSqlDatabaseSecondary –PartnerResourceGroupName "rg2" –PartnerServerName "srv2" –SecondaryElasticPoolName "ElasticPool1" -AllowConnections "All"





## 删除辅助数据库

使用 **Remove-AzureRmSqlDatabaseSecondary** cmdlet 永久终止辅助数据库与其主数据库之间的复制合作关系。终止关系后，辅助数据库将成为读写数据库。如果与辅助数据库的连接断开，命令将会成功，但辅助数据库必须等到连接恢复后才变为可读写。有关详细信息，请参阅 [Remove-AzureRmSqlDatabaseSecondary](https://msdn.microsoft.com/zh-cn/library/mt603457.aspx) 和[服务层](/documentation/articles/sql-database-service-tiers/)。

此 cmdlet 取代了用于复制的 Stop-AzureSqlDatabaseCopy。

这项删除操作相当于强制终止，删除复制链接并将前面的辅助数据库保留作为终止之前未完全复制的独立数据库。将清除前面的主要和前面的辅助数据库上链接的所有数据（如果有）。删除复制链接时此 cmdlet 将返回。


为了删除辅助数据库，根据 RBAC，用户应该拥有主数据库和辅助数据库的写入权限。有关详细信息，请参阅“基于角色的访问控制”。

以下代码将删除资源组“rg2”的服务器“srv2”名为“mydb”的数据库复制链接。

    $database = Get-AzureRmSqlDatabase –DatabaseName "mydb" -ResourceGroupName "rg1" -ServerName "srv1"
    $secondaryLink = $database | Get-AzureRmSqlDatabaseReplicationLink –SecondaryResourceGroup "rg2" –PartnerServerName "srv2"
    $secondaryLink | Remove-AzureRmSqlDatabaseSecondary 


## 监视异地复制配置和运行状况

监视任务包括监视异地复制配置和监视数据复制运行状况。

使用 [Get-AzureRmSqlDatabaseReplicationLink](https://msdn.microsoft.com/zh-cn/library/mt619330.aspx) 可以检索 sys.geo\_replication\_links 目录视图中显示的正向复制链接的相关信息。

以下命令将检索主数据库“mydb”与资源组“rg2”中服务器“srv2”上的辅助数据库之间的复制链接状态。

    $database = Get-AzureRmSqlDatabase –DatabaseName "mydb" -ResourceGroupName "rg1" -ServerName "srv1"
    $secondaryLink = $database | Get-AzureRmSqlDatabaseReplicationLink –PartnerResourceGroup "rg2” –PartnerServerName "srv2”


## 后续步骤

- 若要深入了解活动异地复制，请参阅[活动异地复制](/documentation/articles/sql-database-geo-replication-overview/)
- 有关业务连续性概述和应用场景，请参阅[业务连续性概述](/documentation/articles/sql-database-business-continuity/)

<!---HONumber=Mooncake_0912_2016-->