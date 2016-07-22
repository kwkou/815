<properties 
   pageTitle="SQL 数据库灾难恢复演练" 
   description="了解有关使用 Azure SQL 数据库执行灾难恢复演练，帮助任务关键型业务应用程序弹性应对故障和中断的指导和最佳实践。" 
   services="sql-database" 
   documentationCenter="" 
   authors="mihaelablendea" 
   manager="jhubbard" 
   editor="monicar"/>

<tags
   ms.service="sql-database"
   ms.date="06/16/2016"
   wacn.date="07/21/2016"/>

#执行灾难恢复演练

建议定期对恢复工作流执行应用程序就绪性验证。验证应用程序的行为以及数据丢失和/或涉及到故障转移的中断所造成的影响，是一种良好的工程实践。许多行业标准在涉及到业务连续性认证方面也会提出此要求。

执行灾难恢复演练的操作包括：

- 模拟数据层中断
- 恢复 
- 验证恢复后的应用程序完整性

根据[针对业务连续性设计应用程序](/documentation/articles/sql-database-business-continuity/)的方式，用于执行演练的工作流会有所不同。下面将会讨论在 Azure SQL 数据库上下文中执行灾难恢复演练的最佳实践。

##异地还原

若要防止执行灾难恢复演练时发生潜在的数据丢失，我们建议通过创建生产环境的副本在测试环境中执行演练，并使用测试环境来验证应用程序的故障转移工作流。
 
####中断模拟

若要模拟中断，可以删除或重命名源数据库。这会导致应用程序连接失败。

####恢复

- 根据[此处](/documentation/articles/sql-database-disaster-recovery/)所述，在另一台服务器中执行数据库异地还原。 
- 更改应用程序配置以连接到已恢复的数据库，并根据[在恢复后配置数据库](/documentation/articles/sql-database-disaster-recovery/)指南完成恢复。

####验证

- 通过验证恢复后的应用程序完整性（例如，连接字符串、登录名、基本功能测试，或标准应用程序验收过程的其他验证部分）来完成演练。

##异地复制

对于使用异地复制保护的数据库，演练过程包括按计划故障转移到辅助数据库。计划的故障转移可确保在切换角色后主数据库和辅助数据库保持同步。与非计划的故障转移不同，此操作不会导致数据丢失，因此可以在生产环境中执行演练。

####中断模拟

若要模拟中断，可以禁用已连接到数据库的 Web 应用程序或虚拟机。这会导致 Web 客户端连接失败。

####恢复

- 确保 DR 区域中的应用程序配置指向以前的辅助数据库，故障转移后，该数据库将成为完全可访问的新主数据库。 
- 执行[计划的故障转移](/documentation/articles/sql-database-geo-replication-powershell/#initiate-a-planned-failover)，使辅助数据库成为新的主数据库
- 根据[在恢复后配置数据库](/documentation/articles/sql-database-disaster-recovery/)指南完成恢复。

####验证

- 通过验证恢复后的应用程序完整性（例如，连接字符串、登录名、基本功能测试，或标准应用程序验收过程的其他验证部分）来完成演练。


## 后续步骤

- 有关使用和配置灾难恢复的活动异地复制功能的信息，请参阅 [Active Geo-Replication（活动异地复制）](/documentation/articles/sql-database-geo-replication-overview/)
- 有关使用异地还原进行灾难恢复的信息，请参阅 [Geo-Restore（异地还原）](/documentation/articles/sql-database-geo-restore/)

## 其他资源

- [SQL 数据库业务连续性和灾难恢复](/documentation/articles/sql-database-business-continuity/)
- [时间点还原](/documentation/articles/sql-database-point-in-time-restore/)
- [异地还原](/documentation/articles/sql-database-geo-restore/)
- [活动异地复制](/documentation/articles/sql-database-geo-replication-overview/)
- [设计用于云灾难恢复的应用程序](/documentation/articles/sql-database-designing-cloud-solutions-for-disaster-recovery/)
- [确认已恢复的 Azure SQL 数据库](/documentation/articles/sql-database-recovered-finalize/)
- [异地复制的安全性配置](/documentation/articles/sql-database-geo-replication-security-config/)
- [SQL 数据库 BCDR 常见问题](/documentation/articles/sql-database-business-continuity/)

<!---HONumber=Mooncake_0704_2016-->