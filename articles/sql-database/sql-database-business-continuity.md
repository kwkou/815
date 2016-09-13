<properties
   pageTitle="云业务连续性 - 数据库恢复 - SQL 数据库 | Azure"
   description="了解 Azure SQL 数据库如何支持云业务连续性和数据库恢复以及如何帮助保持运行任务关键型云应用程序。"
   keywords="业务连续性, 云业务连续性, 数据库灾难恢复, 数据库恢复"
   services="sql-database"
   documentationCenter=""
   authors="elfisher"
   manager="jhubbard"
   editor="monicar"/>

<tags
   ms.service="sql-database"
   ms.date="06/09/2016"
   wacn.date="07/21/2016"/>

# 概述：云业务连续性与使用 Azure SQL 数据库进行数据库灾难恢复

> [AZURE.SELECTOR]
- [时间点还原](/documentation/articles/sql-database-point-in-time-restore)
- [还原已删除的数据库](/documentation/articles/sql-database-restore-deleted-database-powershell/)
- [异地还原](/documentation/articles/sql-database-geo-restore)
- [活动异地复制](/documentation/articles/sql-database-geo-replication-overview)
- [业务连续性方案](/documentation/articles/sql-database-business-continuity-scenarios)

Azure SQL 数据库可提供大量业务连续性解决方案。业务连续性是指以弹性方式设计、部署和运行应用程序，以应对计划或非计划的中断事件，避免应用程序永久性或暂时性地无法执行其业务功能。非计划事件覆盖了从人为失误、到永久或暂时性的中断、再到区域性的灾难（这可能会导致特定 Azure 区域中出现大规模的机能损失）的整个范围。计划事件包括向不同的区域重新部署应用程序以及应用程序升级。业务连续性的目标是发生这些事件期间使应用程序能够继续正常运行，而只会对业务功能造成极小的影响。

若要探讨 SQL 数据库云业务连续性解决方案，你需要熟悉几个概念：这些位置包括：

* **灾难恢复 (DR)：**还原应用程序正常业务功能的过程

* **预计恢复时间 (ERT)：**在发出还原或故障转移请求后，数据库完全可用之前预计持续的时间。

* **恢复时间目标 (RTO)**：在发生中断性事件后，应用程序完全恢复之前的最长可接受时间。RTO 用于度量故障期间的最大可用性损失。

* **恢复点目标 (RPO)**：在发生中断性事件后，应用程序在完全恢复时可以丢失的最大最近更新数量（时间间隔）。RPO 用于度量故障期间的最大数据丢失。


## SQL 数据库云业务连续性场景

以下是在规划业务连续性和数据库恢复时要考虑的主要场景：

###设计用于保持业务连续性的应用程序

我要构建的应用程序对我的业务而言至关重要。我希望设计和配置的应用程序在服务发生区域性的灾难故障时可以生存。我知道应用程序的 RPO 和 RTO 要求，现在想要选择满足这些要求的配置。

###在人为失误后恢复

我对应用程序的生产版本拥有管理访问权限。在日常维护过程中，我犯了一个错误，删除了生产中使用的一些重要数据。我想要快速还原这些数据，以减轻该错误造成的影响。

###在中断后恢复

我在生产环境中运行应用程序时收到了一条警报，其中指出，该应用程序所部署到的区域中发生了严重的服务中断。我想要启动恢复过程，以便在另一个区域中重新运行该应用程序，以减轻对业务造成的影响。

###执行灾难恢复演练

由于在中断后进行恢复需要将应用程序的数据层重新定位到其他区域，我想要定期测试恢复过程，并评估它对应用程序的影响，从而使应用程序随时保持工作状态。

###在不停机的情况下升级应用程序

我正在释放应用程序的一项重要升级。这涉及到数据库架构更改、部署其他存储过程，等等。此过程需要阻止用户访问数据库。同时，我想要确保升级过程不会导致业务运营出现重大中断。

## SQL 数据库业务连续性功能

下表列出了 SQL 数据库业务连续性功能，并显示了这些功能在各个[服务层](/documentation/articles/sql-database-service-tiers)中的差异：

| 功能 | 基本层 | 标准层 |高级层
| --- |--- | --- | ---
| 时间点还原 | 7 天内的任何还原点 | 14 天内的任何还原点 | 35 天内的任何还原点
| 异地还原 | ERT < 12 小时，RPO < 1 小时 | ERT < 12 小时，RPO < 1 小时 | ERT < 12 小时，RPO < 1 小时
| 活动异地复制 | ERT < 30 秒，RPO < 5 秒 | ERT < 30 秒，RPO < 5 秒 | ERT < 30 秒，RPO < 5 秒

提供这些功能是为了解决前面列出的方案。有关如何选择特定功能的指导，请参阅[业务连续性设计](/documentation/articles/sql-database-business-continuity-scenarios/)部分。

> [AZURE.NOTE] ERT 和 RPO 值是工程目标，并仅提供指导。它们不属于 [SQL 数据库的 SLA](/support/legal/sla)


###时间点还原

[时间点还原](/documentation/articles/sql-database-point-in-time-restore/)旨在将数据库还原到以前的某个时间点。它使用服务自动为每个用户数据库维护的数据库备份、增量备份和事务日志备份。此功能适用于所有服务层。基本、标准和高级数据库的还原期限分别为 7 天、14 天和 35 天。

###异地还原

[异地还原](/documentation/articles/sql-database-geo-restore/)也适用于基本、标准和高级数据库。当数据库由于它所在的区域发生事故而不可用时，异地还原会提供默认的恢复选项。与时间点还原一样，异地还原依赖于异地冗余的 Azure 存储空间中的数据库备份。它会从异地复制的备份副本中还原，因此可以灵活应对主要区域中的存储中断。

###活动异地复制

[活动异地复制](/documentation/articles/sql-database-geo-replication-overview)适用于所有数据库层。它专为恢复要求超出了异地还原的能力的应用程序而设计。使用活动异地复制，最多可以在不同区域中的服务器上创建四个可读辅助数据库。可以启动到任何辅助数据库的故障转移。此外，活动异地复制可用于支持[应用程序升级或重定位情况](/documentation/articles/sql-database-manage-application-rolling-upgrade)，以及只读工作负荷的负载平衡。

## 在业务连续性功能中选择

针对业务连续性设计应用程序需要你回答以下问题：

1. 哪种业务连续性功能适合用于防止我的应用程序发生中断？
2. 我要使用哪个级别的冗余和复制拓扑？

有关使用弹性池时的详细的恢复策略，请参阅 [Disaster recovery strategies for applications using SQL Database Elastic Pool（使用 SQL 数据库弹性池的应用程序的灾难恢复策略）](/documentation/articles/sql-database-disaster-recovery-strategies-for-applications-with-elastic-pool)。

### 何时使用异地还原

当你的数据库因其所在的区域发生事故而不可用时，[异地还原](/documentation/articles/sql-database-geo-restore)会提供默认的恢复选项。默认情况下，SQL 数据库为每个数据库提供内置的基本保护。具体的保护方式是执行数据库备份并将[数据库备份](/documentation/articles/sql-database-automated-backups)存储在异地冗余的 Azure 存储空间 (GRS) 中。如果你选择此方法，则不需要进行特殊的配置或其他资源分配。你可以将数据库恢复到任何区域，方法是从这些自动异地冗余备份中还原到新的数据库。

如果你的应用程序符合以下条件，则你应该使用内置保护功能：

1. 它不是任务关键型应用程序。它没有约束性的 SLA，也就是说，停机 24 小时或更长时间不会导致财务责任。
2. 数据更改频率（例如，每小时事务数）较低。1 小时的 RPO 不会导致丢失大量数据。
3. 应用程序成本敏感，并且无法判断异地复制的额外成本 

> [AZURE.NOTE] 在任何特定的区域，中断期间异地还原不会预先分配计算容量从备份中还原活动的数据库。服务管理与异地还原请求关联的工作负荷的方式是，最大程度地降低对该区域中现有数据库的影响，而这些数据库的容量需求具有优先级。因此，数据库恢复时间将取决于同一区域中同一时间要恢复的其他数据库数量，以及数据库的大小、事务日志的数量和网络带宽等等。

### 何时使用活动异地复制

[活动异地复制](/documentation/articles/sql-database-geo-replication-overview)支持从主要位置创建和维护位于不同区域的可读（辅助）数据库，并使用异步复制机制使数据库保持最新。它可以保证你的数据库在恢复后，具有必要的数据和计算资源来支持应用程序的工作负荷。

如果你的应用程序符合以下条件，则应使用活动异地复制：

1. 它是任务关键型应用程序。它具有严格规定了 RPO 和 RTO 的约束性 SLA。丢失数据和可用性会导致财务责任。 
2. 数据更改频率（例如每分钟或每秒事务数）较高。使用默认保护时发生 1 小时的 RPO 很可能会导致不可接受的数据丢失。
3. 使用异地复制所涉及的成本明显低于潜在财务责任和相关业务损失所付出的代价。


## 后续步骤

- 若要了解有关 Azure SQL 数据库自动备份的信息，请参阅 [SQL Database automated backups（SQL 数据库自动备份）](/documentation/articles/sql-database-automated-backups)
- 若要了解如何使用 Azure SQL 数据库自动备份进行数据库的时间点还原，请参阅 [Point-in-time restore（时间点还原）](/documentation/articles/sql-database-point-in-time-restore)
- 若要了解如何使用 Azure SQL 数据库自动备份进行数据库的异地还原，请参阅 [Geo-Restore（异地还原）](/documentation/articles/sql-database-geo-restore)
- 若要了解如何使用配置和使用活动异地复制来实现业务连续性，请参阅 [Active Geo-Replication（活动异地复制）](/documentation/articles/sql-database-geo-replication-overview)

## 其他资源

- [在中断后恢复](/documentation/articles/sql-database-disaster-recovery)
- [从用户错误中恢复](/documentation/articles/sql-database-user-error-recovery)
- [执行灾难恢复演练](/documentation/articles/sql-database-disaster-recovery-drills)
- [管理恢复后的安全性](/documentation/articles/sql-database-geo-replication-security-config)
- [使用 SQL 数据库活动异地复制管理云应用程序的滚动升级](/documentation/articles/sql-database-manage-application-rolling-upgrade)
- [设计针对灾难恢复的云解决方案](/documentation/articles/sql-database-designing-cloud-solutions-for-disaster-recovery)
- [使用 SQL 数据库弹性池的应用程序的灾难恢复策略](/documentation/articles/sql-database-disaster-recovery-strategies-for-applications-with-elastic-pool)
- [使用异地复制设计灾难恢复的云解决方案](/documentation/articles/sql-database-designing-cloud-solutions-for-disaster-recovery)

<!---HONumber=Mooncake_0704_2016-->