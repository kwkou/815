<properties 
    pageTitle="使用 Transact-SQL 为 Azure SQL 数据库启动计划内或计划外故障转移 | Azure" 
    description="使用 Transact-SQL 为 Azure SQL 数据库启动计划内或计划外故障转移" 
    services="sql-database" 
    documentationCenter="" 
    authors="carlrabeler" 
    manager="jhubbard" 
    editor=""/>

<tags
    ms.service="sql-database"
    ms.date="04/27/2016"
    wacn.date="06/14/2016"/>

# 使用 Transact-SQL 为 Azure SQL 数据库启动计划内或计划外故障转移


> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/sql-database-geo-replication-failover-powershell/)
- [Transact-SQL](/documentation/articles/sql-database-geo-replication-failover-transact-sql/)


本文介绍了如何使用 Transact-SQL 为辅助 SQL 数据库启动故障转移。若要配置异地复制，请参阅[为 Azure SQL 数据库配置异地复制](/documentation/articles/sql-database-geo-replication-transact-sql/)。



若要启动故障转移，需要提供以下各项：

- 主数据库上 DBManager 的登录信息，拥有你要异地复制的本地数据库的 db\_ownership，以及你要配置异地复制的伙伴服务器上的 DBManager。
- 最新版本的 SQL Server Management Studio - 若要获取最新版本的 SQL Server Management Studio (SSMS)，请转到[下载 SQL Server Management Studio](https://msdn.microsoft.com/zh-cn/library/mt238290.aspx)。有关使用 SQL Server Management Studio 管理 Azure SQL 数据库逻辑服务器和数据库的详细信息，请参阅[使用 SQL Server Management Studio 管理 Azure SQL 数据库](/documentation/articles/sql-database-manage-azure-ssms/)



## 启动计划的故障转移，将辅助数据库升级为新的主数据库

可以使用 **ALTER DATABASE** 语句来升级辅助数据库，以计划好的方式使它成为新的主数据库，并将现有的主数据库降级为辅助数据库。此语句将在要升级的异地复制辅助数据库所在的 Azure SQL 数据库逻辑服务器中的 master 数据库上运行。此功能适用于计划的故障转移（例如灾难恢复演练期间），它要求主数据库可用。

该命令将执行以下工作流：

1. 暂时将复制切换为同步模式，导致所有未完成的事务被排送到辅助数据库并阻止所有新事务；

2. 切换异地复制合作关系中两个数据库的角色。

此序列保证在角色切换前同步这两个数据库，因此不会发生数据丢失。切换角色时，有一小段时间无法使用这两个数据库（大约为 0 到 25 秒）。如果主数据库具有多个辅助数据库，则该命令将自动重新配置其他辅助数据库以连接到新的主数据库。在正常情况下，完成整个操作所需的时间应该少于一分钟。有关详细信息，请参阅 [ALTER DATABASE (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/mt574871.aspx) 和[服务层](/documentation/articles/sql-database-service-tiers/)。


使用以下步骤来启动计划的故障转移。

1. 在 Management Studio 中，连接到异地复制辅助数据库所在的 Azure SQL 数据库逻辑服务器。

2. 打开“数据库”文件夹、展开“系统数据库”、右键单击“master”，然后单击“新建查询”。

3. 使用以下 **ALTER DATABASE** 语句将辅助数据库切换到主数据库角色。

        ALTER DATABASE <MyDB> FAILOVER;

4. 单击“执行”运行查询。

>[AZURE.NOTE] 在少数情况下，操作无法完成并且可能会出现停滞。在此情况下，用户可以执行强制故障转移命令并接受数据丢失。


## 启动从主数据库到辅助数据库的非计划故障转移

可以使用 **ALTER DATABASE** 语句来升级辅助数据库，以非计划的方式使它成为新的主数据库，当主数据库不可用时，强制将现有主数据库降级为辅助数据库。此语句将在要升级的异地复制辅助数据库所在的 Azure SQL 数据库逻辑服务器中的 master 数据库上运行。

此功能适用于还原数据库的可用性至关重要并且可接受部分数据丢失的灾难恢复场合。调用强制故障转移时，指定的辅助数据库将立即成为主数据库，并开始接受写入事务。一旦原始主数据库能够与此新主数据库重新连接，将在原始主数据库上创建增量备份，旧的主数据库将变成新主数据库的辅助数据库；然后，它只是新主数据库的同步副本。

但是，由于辅助数据库上不支持时间点还原，发生强制故障转移之前，如果用户想要恢复已提交到旧主数据库但尚未复制到新主数据库的数据，则应该咨询支持人员来恢复这些丢失的数据。

如果主数据库具有多个辅助数据库，则该命令将自动重新配置其他辅助数据库以连接到新的主数据库。

使用以下步骤来启动计划外故障转移。

1. 在 Management Studio 中，连接到异地复制辅助数据库所在的 Azure SQL 数据库逻辑服务器。

2. 打开“数据库”文件夹、展开“系统数据库”、右键单击“master”，然后单击“新建查询”。

3. 使用以下 **ALTER DATABASE** 语句将辅助数据库切换到主数据库角色。

        ALTER DATABASE <MyDB>   FORCE_FAILOVER_ALLOW_DATA_LOSS;

4. 单击“执行”运行查询。

>[AZURE.NOTE] 如果在主数据库和辅助数据库在线时发出此命令，旧的主数据库将立即变为新的辅助数据库，但不会进行数据同步。如果发出命令时主数据库正在提交事务，则可能会丢失某些数据。



## 其他资源

- [新异地复制功能的亮点](https://azure.microsoft.com/blog/spotlight-on-new-capabilities-of-azure-sql-database-geo-replication)
- [将云应用程序设计为使用异地复制实现业务连续性](/documentation/articles/sql-database-designing-cloud-solutions-for-disaster-recovery/)
- [业务连续性概述](/documentation/articles/sql-database-business-continuity/)
- [SQL 数据库文档](/home/features/sql-database)
- [灾难恢复练习](/documentation/articles/sql-database-disaster-recovery-drills/)

<!---HONumber=Mooncake_0530_2016-->
