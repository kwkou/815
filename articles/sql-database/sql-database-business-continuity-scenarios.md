<properties
	pageTitle="Azure SQL 数据库业务连续性场景 | Azure"
	description="Azure SQL 数据库业务连续性场景"
	services="sql-database"
	documentationCenter=""
	authors="carlrabeler"
	manager="jhubbard"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.date="06/16/2016"
	wacn.date="07/21/2016"/>



# Azure SQL 数据库业务连续性场景

> [AZURE.SELECTOR]
- [业务连续性](/documentation/articles/sql-database-business-continuity)
- [方案](/documentation/articles/sql-database-business-continuity-scenarios)
- [时间点还原](/documentation/articles/sql-database-point-in-time-retore)
- [还原已删除的数据库](/documentation/articles/sql-database-restore-deleted-database-powershell/)
- [异地还原](/documentation/articles/sql-database-geo-restore)
- [活动异地复制](/documentation/articles/sql-database-geo-replication)

在本文中，你可以了解一些 Azure SQL 数据库业务连续性场景。

## 在中断后恢复

[Restore an Azure SQL Database or failover to a secondary（还原 Azure SQL 数据库或故障转移到辅助数据库）](/documentation/articles/sql-database-disaster-recovery)这篇文章介绍了如何使用以下功能从故障中恢复：

- [活动异地复制](/documentation/articles/sql-database-geo-replication-overview)
- [异地还原](/documentation/articles/sql-database-geo-restore)

本文讨论了何时启动恢复、如何使用各个功能进行恢复、恢复后如何配置数据库，以及恢复后如何配置应用程序。

## 从用户错误中恢复

[Recover an Azure SQL Database from a user error（从用户错误中恢复 Azure SQL 数据库）](/documentation/articles/sql-database-user-error-recovery)这篇文章介绍了如何使用以下功能从用户错误或意外的数据更改中恢复：

- [时间点还原](/documentation/articles/sql-database-point-in-time-restore) 

## 执行灾难恢复演练

[Performing a disaster recovery drill（执行灾难恢复演练）](/documentation/articles/sql-database-disaster-recovery-drills)这篇文章介绍了如何使用以下功能进行灾难恢复演练：

- [活动异地复制](/documentation/articles/sql-database-geo-replication-overview)
- [异地还原](/documentation/articles/sql-database-geo-restore)

建议定期对恢复工作流执行应用程序就绪性验证。验证应用程序的行为以及数据丢失和/或涉及到故障转移的中断所造成的影响，是一种良好的工程实践。许多行业标准在涉及到业务连续性认证方面也会提出此要求。

执行灾难恢复演练的操作包括：

- 模拟数据层中断
- 恢复 
- 验证恢复后的应用程序完整性

## 管理灾难恢复后的安全性

[How to manage security after disaster recovery（如何管理灾难恢复后的安全性）](/documentation/articles/sql-database-geo-replication-security-config)这篇文章介绍了配置和控制[活动异地复制](/documentation/articles/sql-database-geo-replication-overview)时的身份验证要求，以及设置用户对辅助数据库的访问权限所要执行的步骤。这篇文章还介绍了使用[异地还原](/documentation/articles/sql-database-geo-restore)后如何启用对已恢复的数据库的访问。

## 使用活动异地复制管理云应用程序的滚动升级

[Managing rolling upgrades of cloud applications using SQL Database Active Geo-Replication（使用 SQL 数据库活动异地复制管理云应用程序的滚动升级）](/documentation/articles/sql-database-manage-application-rolling-upgrade)这篇文章介绍了如何使用 SQL 数据库中的[异地复制](/documentation/articles/sql-database-geo-replication-overview)功能启用云应用程序的滚动升级。由于升级是中断性操作，所以它应成为业务连续性规划和设计的一部分。本文介绍了编排升级过程的两种不同方法，并讨论了每种方法的优点和缺点。

## 使用活动异地复制功能为云灾难恢复设计应用程序

[Design an application for cloud disaster recovery using Active Geo-Replication in SQL Database（使用 SQL 数据库中的活动异地复制功能为云灾难恢复设计应用程序）](/documentation/articles/sql-database-designing-cloud-solutions-for-disaster-recovery)这篇文章介绍了如何使用 SQL 数据库中的[活动异地复制](/documentation/articles/sql-database-geo-replication-overview)功能设计用于应对区域故障和灾难性停机的数据库应用程序。本文探讨了应用程序部署拓扑、你要面对的服务级别协议、通信延迟和成本，然后介绍了常见的应用程序模式，以及每个模式的优点和缺点。

## 使用弹性数据库池的应用程序的灾难恢复策略

[Disaster recovery strategies for applications using SQL Database Elastic Pool（使用 SQL 数据库弹性池的应用程序的灾难恢复策略）](/documentation/articles/sql-database-disaster-recovery-strategies-for-applications-with-elastic-pool)这篇文章介绍了使用[弹性数据库池](/documentation/articles/sql-database-elastic-pool)的灾难恢复方案。

## 后续步骤

- 有关使用和配置灾难恢复的活动异地复制功能的信息，请参阅 [Active Geo-Replication（活动异地复制）](/documentation/articles/sql-database-geo-replication-overview)
- 有关使用异地还原进行灾难恢复的信息，请参阅 [Geo-Restore（异地还原）](/documentation/articles/sql-database-geo-restore)

## 其他资源

- [SQL 数据库业务连续性和灾难恢复](/documentation/articles/sql-database-business-continuity)
- [时间点还原](/documentation/articles/sql-database-point-in-time-restore)
- [异地还原](/documentation/articles/sql-database-geo-restore)
- [活动异地复制](/documentation/articles/sql-database-geo-replication-overview)
- [设计用于云灾难恢复的应用程序](/documentation/articles/sql-database-designing-cloud-solutions-for-disaster-recovery)
- [确认已恢复的 Azure SQL 数据库](/documentation/articles/sql-database-recovered-finalize)
- [异地复制的安全性配置](/documentation/articles/sql-database-geo-replication-security-config)

<!---HONumber=Mooncake_0704_2016-->