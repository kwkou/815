<properties 
    pageTitle="使用 PowerShell 为 Azure SQL 数据库启动计划内或计划外故障转移 | Azure" 
    description="使用 PowerShell 为 Azure SQL 数据库启动计划内或计划外故障转移" 
    services="sql-database" 
    documentationCenter="" 
    authors="stevestein" 
    manager="jhubbard" 
    editor=""/>

<tags
    ms.service="sql-database"
    ms.date="04/27/2016"
    wacn.date="06/14/2016"/>

# 使用 PowerShell 为 Azure SQL 数据库启动计划内或计划外故障转移



> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/sql-database-geo-replication-failover-powershell/)
- [Transact-SQL](/documentation/articles/sql-database-geo-replication-failover-transact-sql/)


本文介绍了如何使用 PowerShell 为 SQL 数据库启动计划内或计划外故障转移。若要配置异地复制，请参阅[为 Azure SQL 数据库配置异地复制](/documentation/articles/sql-database-geo-replication-powershell/)。



## 启动计划的故障转移

使用 **Set-AzureRmSqlDatabaseSecondary** cmdlet 并结合 **-Failover** 参数来升级辅助数据库，使它成为新的主数据库，并将现有主数据库降级为辅助数据库。此功能适用于计划的故障转移（例如灾难恢复演练期间），它要求主数据库可用。

该命令将执行以下工作流：

1. 暂时将复制切换到同步模式。这会导致将所有未处理的事务排送到辅助数据库。

2. 切换异地复制合作关系中两个数据库的角色。

此序列保证在角色切换前同步这两个数据库，因此不会发生数据丢失。切换角色时，有一小段时间无法使用这两个数据库（大约为 0 到 25 秒）。在正常情况下，完成整个操作所需的时间应该少于一分钟。有关详细信息，请参阅 [Set-AzureRmSqlDatabaseSecondary](https://msdn.microsoft.com/zh-cn/library/mt619393.aspx)。




将辅助数据库切换为主数据库完成时，此 cmdlet 将返回。

以下命令将资源组“rg2”下服务器“srv2”上名为“mydb”的数据库角色切换为主数据库。"db2”所连接的原始主数据库在两个数据库完全同步之后切换为辅助数据库。

    $database = Get-AzureRmSqlDatabase –DatabaseName "mydb" –ResourceGroupName "rg2” –ServerName "srv2”
    $database | Set-AzureRmSqlDatabaseSecondary -Failover


> [AZURE.NOTE] 在很少见的情况下，操作可能无法完成并且可能会出现停止响应。在此情况下，用户可以调用强制故障转移命令（非计划的故障转移）并接受数据丢失。


## 启动从主数据库到辅助数据库的非计划故障转移


可以使用 **Set-AzureRmSqlDatabaseSecondary** cmdlet 并结合 **-Failover** 和 **-AllowDataLoss** 参数来升级辅助数据库，以非计划的方式使它成为新的主数据库，当主数据库不可用时，强制将现有主数据库降级为辅助数据库。

此功能适用于还原数据库的可用性至关重要并且可接受部分数据丢失的灾难恢复场合。调用强制故障转移时，指定的辅助数据库将立即成为主数据库，并开始接受写入事务。执行强制故障转移操作后，一旦原始主数据库能够与此新主数据库重新连接，将在原始主数据库上创建增量备份，旧的主数据库将变成新主数据库的辅助数据库；然后，它只是新主数据库的副本。

但是，由于辅助数据库上不支持时间点还原，如果你想要恢复已提交到旧主数据库但尚未复制到新主数据库的数据，则应该咨询 CSS 将数据库还原到已知的日志备份。

> [AZURE.NOTE] 如果在主数据库和辅助数据库在线时发出此命令，旧的主数据库将立即变为新的辅助数据库，但不会进行数据同步。如果发出命令时主数据库正在提交事务，则可能会丢失某些数据。


如果主数据库中有多个辅助数据库，命令只会部分成功。执行命令的辅助数据库将变为主数据库。但是，旧的主数据库将仍为主数据库，即，两个主数据库最终处于不一致状态并通过挂起的复制链接进行连接。用户必须在主数据库上使用“删除辅助数据库”API 手动修复此配置。


以下命令在主数据库无法使用时将名为“mydb”的数据库角色切换为主数据库。“mydb”连接的原始主数据库将在重新联机后切换为辅助数据库。在该时间点，同步可能导致数据丢失。

    $database = Get-AzureRmSqlDatabase –DatabaseName "mydb" –ResourceGroupName "rg2” –ServerName "srv2”
    $database | Set-AzureRmSqlDatabaseSecondary –Failover -AllowDataLoss




## 其他资源

- [新异地复制功能的亮点](https://azure.microsoft.com/blog/spotlight-on-new-capabilities-of-azure-sql-database-geo-replication)
- [将云应用程序设计为使用异地复制实现业务连续性](/documentation/articles/sql-database-designing-cloud-solutions-for-disaster-recovery/)
- [业务连续性概述](/documentation/articles/sql-database-business-continuity/)
- [SQL 数据库文档](/documentation/services/sql-databases)
- [灾难恢复练习](/documentation/articles/sql-database-disaster-recovery-drills/)

<!---HONumber=Mooncake_0530_2016-->
