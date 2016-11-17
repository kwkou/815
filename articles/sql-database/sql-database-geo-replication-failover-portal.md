<properties 
    pageTitle="使用 Azure 门户预览为 Azure SQL 数据库启动计划内或计划外故障转移 | Azure" 
    description="使用 Azure 门户预览为 Azure SQL 数据库启动计划内或计划外故障转移" 
    services="sql-database" 
    documentationCenter="" 
    authors="stevestein" 
    manager="jhubbard" 
    editor=""/>

<tags
    ms.service="sql-database"
    ms.devlang="NA"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="data-management" 
    ms.date="08/29/2016" 
    wacn.date="10/17/2016"
    ms.author="sstein"/>

# 使用 Azure 门户预览为 Azure SQL 数据库启动计划内或计划外故障转移


> [AZURE.SELECTOR]
- [Azure 门户预览](/documentation/articles/sql-database-geo-replication-failover-portal/)
- [PowerShell](/documentation/articles/sql-database-geo-replication-failover-powershell/)
- [T-SQL](/documentation/articles/sql-database-geo-replication-failover-transact-sql/)


本文介绍了如何使用 [Azure 门户预览](http://portal.azure.cn)为辅助 SQL 数据库启动故障转移。若要配置异地复制，请参阅[为 Azure SQL 数据库配置异地复制](/documentation/articles/sql-database-geo-replication-portal/)。


## 启动故障转移

辅助数据库可以通过切换变为主数据库。

1. 在 [Azure 门户预览](http://portal.azure.cn)中浏览到异地复制合作关系中的主数据库。
2. 在 SQL 数据库边栏选项卡中，选择“所有设置”>“异地复制”。
3. 在“辅助数据库”列表中，选择想要其成为新的主数据库的数据库并单击“故障转移”。

    ![故障转移][2]

4. 单击“是”开始故障转移。

该命令会立即将辅助数据库切换为主数据库角色。

切换角色时，有一小段时间无法使用这两个数据库（大约为 0 到 25 秒）。如果主数据库具有多个辅助数据库，则该命令将自动重新配置其他辅助数据库以连接到新的主数据库。在正常情况下，完成整个操作所需的时间应该少于一分钟。

>[AZURE.NOTE] 如果发出命令时主数据库处于在线状态且正在提交事务，则可能会丢失某些数据。


## 后续步骤   

- 故障转移后，请确保在新的主机上配置服务器和数据库的身份验证要求。有关详细信息，请参阅 [SQL Database security after disaster recovery](/documentation/articles/sql-database-geo-replication-security-config/)（灾难恢复后的 Azure SQL 数据库安全性）。
- 若要了解如何使用活动异地复制进行灾难恢复，包括恢复前后步骤和执行灾难恢复演练，请参阅[灾难恢复演练](/documentation/articles/sql-database-disaster-recovery/)
- 有关活动异地复制的 Sasha Nosov 博客文章，请参阅 [Spotlight on new Geo-Replication capabilities](https://azure.microsoft.com/blog/spotlight-on-new-capabilities-of-azure-sql-database-geo-replication/)（聚焦异地复制新功能）
- 若要了解如何设计云应用程序以使用活动异地复制，请参阅[设计云应用程序以使用异地复制实现业务连续性](/documentation/articles/sql-database-designing-cloud-solutions-for-disaster-recovery/)
- 若要了解如何通过弹性数据库池使用活动异地复制，请参阅[弹性池灾难恢复策略](/documentation/articles/sql-database-disaster-recovery-strategies-for-applications-with-elastic-pool/)。
- 有关业务连续性概述，请参阅[业务连续性概述](/documentation/articles/sql-database-business-continuity/)




<!--Image references-->
[1]: ./media/sql-database-geo-replication-failover-portal/failover.png
[2]: ./media/sql-database-geo-replication-failover-portal/secondaries.png

<!---HONumber=Mooncake_1010_2016-->