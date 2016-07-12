<properties 
    pageTitle="使用 PowerShell 为 Azure SQL 数据库配置异地复制 | Azure" 
    description="使用 PowerShell 为 Azure SQL 数据库配置异地复制" 
    services="sql-database" 
    documentationCenter="" 
    authors="stevestein" 
    manager="jeffreyg" 
    editor=""/>

<tags
    ms.service="sql-database"
    ms.date="02/23/2016"
    wacn.date="06/14/2016"/>

# 使用 PowerShell 为 Azure SQL 数据库配置异地复制



> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/sql-database-geo-replication-powershell/)
- [Transact-SQL](/documentation/articles/sql-database-geo-replication-transact-sql/)


本文说明如何使用 PowerShell 为 Azure SQL 数据库配置异地复制。

异地复制可让你在不同的数据中心位置（区域）最多创建 4 个副本（辅助）数据库。在数据中心发生服务中断或无法连接到主数据库时，可以使用辅助数据库。

异地复制仅适用于标准和高级数据库。

标准数据库可以有一个不可读取的辅助副本，并且必须使用建议的区域。高级数据库最多可以有四个位于任何可用区域并且可读取的辅助副本。

若要配置异地复制，需要提供以下各项：

- Azure 订阅。如果你需要 Azure 订阅，只需单击本页顶部的“试用”，然后再回来完成本文的相关操作即可。
- 一个 Azure SQL 数据库 - 你要复制到不同地理区域的主数据库。
- Azure PowerShell 1.0 或更高版本。根据遵循[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/) 来下载并安装 Azure PowerShell 模块。




## 配置你的凭据，然后选择你的订阅

首先必须与 Azure 帐户建立访问连接，因此请启动 PowerShell，然后运行以下 cmdlet。在登录屏幕中，输入登录 Azure 门户时所用的相同电子邮件和密码。


	Login-AzureRmAccount -EnvironmentName AzureChinaCloud

成功登录后，你会在屏幕上看到一些信息，其中包括你登录时使用的 ID，以及你有权访问的 Azure 订阅。


### 选择 Azure 订阅

若要选择订阅，你需要提供订阅 ID。你可以从前面的步骤中显示的信息中复制订阅 ID，或者，如果你有多个订阅且需要更多详细信息，可以运行 **Get-AzureRmSubscription** cmdlet，然后从结果集中复制所需的订阅信息。以下 cmdlet 使用订阅 ID 来设置当前的订阅：

	Select-AzureRmSubscription -SubscriptionId 4cac86b0-1e56-bbbb-aaaa-000000000000

成功运行 **Select-AzureRmSubscription** 后，将返回到 PowerShell 提示符处。



## 添加辅助数据库


以下步骤在异地复制合作关系中创建新的辅助数据库。
  
若要启用辅助数据库，你必须是订阅所有者或共同所有者。

可以使用 **New-AzureRmSqlDatabaseSecondary** cmdlet，将伙伴服务器上的辅助数据库添加到所连接的服务器上的本地数据库（主数据库）。

此 cmdlet 将 **Start-AzureSqlDatabaseCopy** 替换为 **-IsContinuous** 参数。它将输出可供其他 cmdlet 用于明确识别特定复制链接的 **AzureRmSqlDatabaseSecondary** 对象。创建辅助数据库并完全设定种子后，此 cmdlet 将返回。根据数据库的大小，这可能需要花费数分钟到数小时的时间。

辅助服务器上的复制数据库具备与主要服务器上的数据库相同的名称，并且默认具有相同的服务级别。辅助数据库可以是可读或不可读，并且可以是单一数据库或弹性数据库。有关详细信息，请参阅 [New-AzureRMSqlDatabaseSecondary](https://msdn.microsoft.com/zh-cn/library/mt603689.aspx) 和[服务层](/documentation/articles/sql-database-service-tiers/)。
创建辅助数据库并设定种子之后，开始将数据从主数据库复制到新的辅助数据库。以下步骤说明如何使用 PowerShell 完成这项任务，以使用单一数据库或弹性数据库来创建不可读和可读的辅助数据库。

如果伙伴数据库已存在（例如，由于终止前面的异地复制关系），命令将会失败。



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




## 启动计划的故障转移

使用 **Set-AzureRmSqlDatabaseSecondary** cmdlet 并结合 **-Failover** 参数来升级辅助数据库，使它成为新的主数据库，并将现有主数据库降级为辅助数据库。此功能适用于计划的故障转移（例如灾难恢复演练期间），它要求主数据库可用。

该命令将执行以下工作流：

1. 暂时将复制切换到同步模式。这会导致将所有未处理的事务排送到辅助数据库。

2. 切换异地复制合作关系中两个数据库的角色。

此顺序可确保不发生任何数据丢失。切换角色时，有一小段时间无法使用这两个数据库（大约为 0 到 25 秒）。在正常情况下，完成整个操作所需的时间应该少于一分钟。有关详细信息，请参阅 [Set-AzureRmSqlDatabaseSecondary](https://msdn.microsoft.com/zh-cn/library/mt619393.aspx)。


> [AZURE.NOTE] 发出该命令时，如果主数据库不可用，则命令将会失败并出现一条错误消息，指出没有可用的主服务器。在少数情况下，操作无法完成并且可能会出现停滞。在此情况下，用户可以调用强制故障转移命令（非计划的故障转移）并接受数据丢失。



将辅助数据库切换为主数据库完成时，此 cmdlet 将返回。

以下命令将资源组“rg2”下服务器“srv2”上名为“mydb”的数据库角色切换为主数据库。"db2”所连接的原始主数据库在两个数据库完全同步之后切换为辅助数据库。

    $database = Get-AzureRmSqlDatabase –DatabaseName "mydb" –ResourceGroupName "rg2” –ServerName "srv2”
    $database | Set-AzureRmSqlDatabaseSecondary -Failover



## 启动从主数据库到辅助数据库的非计划故障转移


可以使用 **Set-AzureRmSqlDatabaseSecondary** cmdlet 并结合 **-Failover** 和 **-AllowDataLoss** 参数来升级辅助数据库，以非计划的方式使它成为新的主数据库，当主数据库不可用时，强制将现有主数据库降级为辅助数据库。

此功能适用于还原数据库的可用性至关重要并且可接受部分数据丢失的灾难恢复场合。调用强制故障转移时，指定的辅助数据库将立即成为主数据库，并开始接受写入事务。执行强制故障转移操作后，一旦原始主数据库能够与此新主数据库重新连接，将在原始主数据库上创建增量备份，旧的主数据库将变成新主数据库的辅助数据库；然后，它只是新主数据库的副本。

但是，由于辅助数据库上不支持时间点还原，如果你想要恢复已提交到旧主数据库但尚未复制到新主数据库的数据，则应该咨询 CSS 将数据库还原到已知的日志备份。

> [AZURE.NOTE] 如果在主数据库和辅助数据库联机时发出此命令，旧的主数据库将变为新的辅助数据库，但不会尝试数据同步，因此可能会丢失一些数据。


如果主数据库中有多个辅助数据库，命令只会部分成功。执行命令的辅助数据库将变为主数据库。但是，旧的主数据库将仍为主数据库，即，两个主数据库最终处于不一致状态并通过挂起的复制链接进行连接。用户必须在主数据库上使用“删除辅助数据库”API 手动修复此配置。


以下命令在主数据库无法使用时将名为“mydb”的数据库角色切换为主数据库。“mydb”连接的原始主数据库将在重新联机后切换为辅助数据库。在该时间点，同步可能导致数据丢失。

    $database = Get-AzureRmSqlDatabase –DatabaseName "mydb" –ResourceGroupName "rg2” –ServerName "srv2”
    $database | Set-AzureRmSqlDatabaseSecondary –Failover -AllowDataLoss



## 监视异地复制配置和运行状况

监视任务包括监视异地复制配置和监视数据复制运行状况。

使用 [Get-AzureRmSqlDatabaseReplicationLink](https://msdn.microsoft.com/zh-cn/library/mt619330.aspx) 可以检索 sys.geo\_replication\_links 目录视图中显示的正向复制链接的相关信息。

以下命令将检索主数据库“mydb”与资源组“rg2”中服务器“srv2”上的辅助数据库之间的复制链接状态。

    $database = Get-AzureRmSqlDatabase –DatabaseName "mydb" -ResourceGroupName "rg1" -ServerName "srv1"
    $secondaryLink = $database | Get-AzureRmSqlDatabaseReplicationLink –PartnerResourceGroup "rg2” –PartnerServerName "srv2”



   

## 后续步骤

- [灾难恢复练习](/documentation/articles/sql-database-disaster-recovery-drills/)




## 其他资源

- [新异地复制功能的亮点](https://azure.microsoft.com/blog/spotlight-on-new-capabilities-of-azure-sql-database-geo-replication)
- [将云应用程序设计为使用异地复制实现业务连续性](/documentation/articles/sql-database-designing-cloud-solutions-for-disaster-recovery/)
- [业务连续性概述](/documentation/articles/sql-database-business-continuity/)
- [SQL 数据库文档](/documentation/services/sql-databases)

<!---HONumber=Mooncake_0606_2016-->
