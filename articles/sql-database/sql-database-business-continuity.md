<properties
   pageTitle="云业务连续性 - 数据库恢复 - SQL 数据库 | Azure"
   description="了解 Azure SQL 数据库如何支持云业务连续性和数据库恢复以及如何帮助保持运行任务关键型云应用程序。"
   keywords="业务连续性, 云业务连续性, 数据库灾难恢复, 数据库恢复"
   services="sql-database"
   documentationCenter=""
   authors="CarlRabeler"
   manager="jhubbard"
   editor=""/>  


<tags
   ms.service="sql-database"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="10/13/2016"
   wacn.date="10/31/2016"
   ms.author="carlrab"/>  


# 使用 Azure SQL 数据库确保业务连续性的相关概述

本概述介绍 Azure SQL 数据库针对业务连续性和灾难恢复所提供的功能。它提供用于从干扰性事件恢复的选项、建议和教程，这些事件可能导致数据丢失或者数据库和应用程序无法使用。本概述讨论对一些情况的处理方式，包括用户或应用程序错误影响数据完整性、Azure 区域服务中断，或者应用程序需要维护。

## 有利于业务连续性的 SQL 数据库功能

SQL 数据库提供若干业务连续性功能，包括自动备份和可选的数据库复制。对于最近发生的事务，每种功能在估计恢复时间 (ERT) 和可能丢失的数据方面都有不同的特性。了解这些选项后，便可从中进行选择 — 在大多数方案中，可以针对不同方案将其搭配使用。制定业务连续性计划时，需了解应用程序在干扰性事件之后完全恢复的最大可接受时间，即恢复时间目标 (RTO)。此外，还需要了解从干扰性事件恢复时，应用程序可忍受丢失的最近数据更新（时间间隔）最大数量，即恢复点目标 (RPO)。

下表比较了三种最常见方案的 ERT 和 RPO。

| 功能 |	基本层 | 标准层 | 高级层 |
|---|---|---|---|
| 从备份执行时间点还原 | 7 天内的任何还原点 | 35 天内的任何还原点 | 35 天内的任何还原点 |
从异地复制的备份执行异域还原 | ERT < 12 小时，RPO < 1 小时 | ERT < 12 小时，RPO < 1 小时 | ERT < 12 小时，RPO < 1 小时 |
|活动异地复制 | ERT < 30 秒，RPO < 5 秒 | ERT < 30 秒，RPO < 5 秒 |	ERT < 30 秒，RPO < 5 秒 |


### 使用数据库备份恢复数据库

SQL 数据库每周自动执行完整数据库备份，每小时自动执行差异数据库备份，每 5 分钟自动执行事务日志备份，防止企业丢失数据。对于标准和高级服务层中的数据库，这些备份会在本地冗余存储中存储 35 天；对于基本服务层中的数据库，则存储 7 天 — 有关服务层的更多详细信息，请参阅[服务层](/documentation/articles/sql-database-service-tiers/)。如果服务层的保留期不符合企业需求，可以[更改服务层](/documentation/articles/sql-database-scale-up-powershell/)来延长保留期。完整和差异数据库备份也会复制到配对的数据中心(中国东部和中国北部)，以防数据中心中断 — 有关更多详细信息，请参阅[自动数据库备份](/documentation/articles/sql-database-automated-backups/)。

通过这些自动数据库备份，可以在自己的数据中心以及其他数据中心内，从各种干扰性事件中恢复数据库。使用自动数据库备份时，估计恢复时间取决于若干因素，包括在相同区域同时进行恢复的数据库总数、数据库大小、事务日志大小以及网络带宽。大多数情况下，恢复时间少于 12 小时。恢复到另一个数据区域时，因为每小时会针对异地冗余存储进行差异数据库备份，因此最多只可能丢失 1 小时的数据。

> [AZURE.IMPORTANT] 若要使用自动备份进行恢复，必须是 SQL Server 参与者角色的成员或订阅所有者 — 请参阅 [RBAC：内置角色](/documentation/articles/role-based-access-built-in-roles/)。可以使用 Azure 经典管理门户、PowerShell 或 REST API 进行恢复。但不能使用 Transact-SQL。

如果应用程序符合以下条件，则可以将自动备份作为业务连续性和恢复机制：

- 不是任务关键型应用程序。
- 没有约束性的 SLA，因此，停机 24 小时或更长时间不会导致财务责任。
- 数据更改率低（每小时事务数少），并且最多可接受丢失一小时的数据更改。
- 成本有限。

如需更快速的恢复，请使用[活动异地复制](/documentation/articles/sql-database-geo-replication-overview/)（接下来将讨论）。如果需要恢复 35 天前的数据，请考虑定期将数据库存档到 BACPAC 文件（一种包含数据库架构和关联数据的压缩文件），并存储在 Azure Blob 存储或所选的其他位置中。有关如何创建事务一致数据库存档的详细信息，请参阅[创建数据库副本](/documentation/articles/sql-database-copy/)和[导出数据库副本](/documentation/articles/sql-database-export-powershell/)。

### 使用活动异地复制缩短恢复时间并限制恢复导致的数据丢失

除了在发生业务中断时使用数据库备份来恢复数据库之外，还可以使用[活动异地复制](/documentation/articles/sql-database-geo-replication-overview/)来配置数据库，在所选区域中最多可拥有 4 个可读辅助数据库。这些辅助数据库使用异步复制机制与主数据库保持同步。在数据中心中断或应用程序升级期间，此功能可以防止出现业务中断。活动异地复制还可以为地理位置分散的用户提高只读查询的查询性能。

如果主数据库意外脱机，或者需要脱机维护，可以将辅助数据库快速提升为主数据库（也称为故障转移），并配置应用程序，将它们连接到新提升的主数据库。进行计划内故障转移时，不会丢失任何数据。进行计划外故障转移时，由于异步复制的性质使然，最近的事务可能会丢失少量数据。故障转移之后，可以根据计划或在数据中心重新联机时回复故障。无论在什么情况下，用户都会经历短暂的停机，并需要重新连接。

> [AZURE.IMPORTANT] 若要使用活动异地复制，必须是订阅所有者或在 SQL Server 中拥有管理权限。可通过订阅的权限使用 Azure 经典管理门户、PowerShell 或 REST API 进行配置和故障转移，也可以通过 SQL Server 中的权限使用 Transact-SQL 进行配置和故障转移。

如果应用程序符合以下任意条件，就使用活动异地复制：

- 是任务关键型应用程序。
- 具有服务级别协议 (SLA)，不允许停机 24 小时或更长时间。
- 停机将导致财务责任。
- 具有很高的数据更改率，而且不接受丢失一小时的数据。
- 活动异地复制的额外成本低于潜在财务责任和相关业务损失所付出的代价。

## 在用户或应用程序错误之后恢复数据库

*人无完人。 用户可能会不小心删除某些数据、无意中删除重要的表，甚至删除整个数据库。或者，应用程序可能因为自身缺陷，意外以错误数据覆盖正确数据。

在此方案中，可使用以下恢复选项。

### 执行时间点还原

可使用自动备份，将数据库副本恢复到数据库保留期内的已知时间点。数据库还原之后，可以将原始数据库替换为还原的数据库，或从还原的数据将所需数据复制到原始数据库。如果数据库使用活动异地复制，建议从还原的副本将所需数据复制到原始数据库。如果将原始数据库替换为还原的数据库，需要重新配置并重新同步活动异地复制（大型数据库可能要花费很长时间）。

有关使用 Azure 门户或 PowerShell 将数据库还原至某个时间点的详细信息及步骤，请参阅[时间点还原](/documentation/articles/sql-database-recovery-using-backups/#point-in-time-restore)。不能使用 Transact-SQL 进行恢复。

### 还原已删除的数据库

如果已删除数据库，但逻辑服务器尚未删除，可以将已删除的数据库还原到它被删除的时间点。这会将数据库备份还原到之前删除数据库的同一个逻辑 SQL 服务器。可以使用原始名称还原，也可以为还原的数据库提供新名称。

有关使用 Azure 门户或 PowerShell 还原已删除数据库的详细信息及步骤，请参阅[还原已删除的数据库](/documentation/articles/sql-database-recovery-using-backups/#deleted-database-restore)。不能使用 Transact-SQL 进行还原。

> [AZURE.IMPORTANT] 如果逻辑服务器已删除，则无法恢复已删除的数据库。

### 从数据库存档导入

如果在当前自动备份保留期外发生数据丢失，但用户一直进行数据库存档，可以在新数据库中[导入存档的 BACPAC 文件](/documentation/articles/sql-database-import-powershell/)。此时，可以将原始数据库替换为导入的数据库，或从导入的数据将所需数据复制到原始数据库。

## Azure 区域数据中心中断时将数据库恢复到另一个区域

<!-- Explain this scenario -->

Azure 数据中心会罕见地发生中断。发生中断时，业务可能仅中断几分钟，也可能持续数小时。

- 用户可以选择等待数据中心中断结束，数据库重新联机。这适用于可以容忍数据库脱机的应用程序。例如无需一直处理的开发项目或免费试用版。数据中心中断时，你不知道中断会持续多久，因此该选项仅适用于暂时不需要数据库的情况。
- 另一个选项是故障转移到另一个数据区域（如果使用活动异地复制）或使用异地冗余数据库备份进行恢复（异地还原）。故障转移只需几秒钟，而从备份恢复需要数小时。

执行操作时，恢复所需的时间以及数据中心中断导致的数据丢失量会有所不同，这具体取决于用户要在应用程序中使用上述哪些业务连续性功能。事实上，可以根据应用程序需求，选择结合使用数据库备份与活动异地复制。若要探讨使用这些业务连续性功能为独立数据库和弹性池设计应用程序时的注意事项，请参阅[设计用于云灾难恢复的应用程序](/documentation/articles/sql-database-designing-cloud-solutions-for-disaster-recovery/)和[弹性池灾难恢复策略](/documentation/articles/sql-database-disaster-recovery-strategies-for-applications-with-elastic-pool/)。

以下各节概述使用数据库备份或活动异地复制进行恢复的步骤。若要了解包括计划需求在内的详细步骤、恢复后步骤，以及如何模拟中断以执行灾难恢复演练，请参阅[在中断时恢复 SQL 数据库](/documentation/articles/sql-database-disaster-recovery/)。

### 为中断做好准备

无论使用哪种业务连续性功能，都必须：

- 识别并准备目标服务器，包括服务器级防火墙规则、登录名和 master 数据库级权限。
- 确定如何将客户端和客户端应用程序重定向到新服务器
- 记录其他依赖项，例如审核设置和警报
 
如果不进行适当的计划和准备，故障转移或恢复后，需要花更多时间让应用程序联机，还有可能要在这种情况下进行故障排除 — 可谓雪上加霜。

### 故障转移到异地复制的辅助数据库 

如果将活动异地复制作为恢复机制，请[强制故障转移到异地复制的辅助数据库](/documentation/articles/sql-database-disaster-recovery/#failover-to-geo-replicated-secondary-database)。辅助数据库只需几秒钟就能提升为新的主数据库，而且可以记录新事务并响应任何查询 — 仅丢失几秒尚未复制的数据。有关自动执行故障转移过程的信息，请参阅[设计用于云灾难恢复的应用程序](/documentation/articles/sql-database-designing-cloud-solutions-for-disaster-recovery/)。

> [AZURE.NOTE] 数据中心重新联机时，可以选择故障回复到原始的主数据库。

### 执行异地还原 

如果将自动备份和异地冗余存储复制搭配作为恢复机制，请[使用异地还原启动数据库恢复](/documentation/articles/sql-database-disaster-recovery/#recover-using-geo-restore)。恢复通常在 12 小时内进行 — 最多丢失 1 小时的数据，具体取决于最后一次每小时差异备份的执行和复制时间。在恢复完成之前，数据库无法记录任何事务或响应任何查询。

> [AZURE.NOTE] 如果数据中心在应用程序切换到恢复的数据库之前就重新联机，只要取消恢复即可。

### 执行故障转移/恢复后任务 

从任一恢复机制恢复后，都必须执行以下附加任务，用户和应用程序才能重新运行：

- 将客户端和客户端应用程序重定向到新服务器和还原的数据库
- 确保设置适当的服务器级防火墙规则，供用户连接（或使用[数据库级防火墙](/documentation/articles/sql-database-firewall-configure/#creating-database-level-firewall-rules)）
- 确保设置适当的登录名和 master 数据库级权限（或使用[包含用户](https://msdn.microsoft.com/zh-cn/library/ff929188.aspx)）
- 视情况配置审核
- 视情况配置警报

## 在停机时间最短的情况下升级应用程序

有时，应用程序因为计划内维护（例如应用程序升级）必须脱机。[管理应用程序升级](/documentation/articles/sql-database-manage-application-rolling-upgrade/)介绍如何使用活动异地复制滚动升级云应用程序，将升级时的停机时间缩到最短，并在发生错误时提供恢复路径。本文介绍了编排升级过程的两种不同方法，并讨论了每种方法的优点和缺点。

## 后续步骤

若要探讨为独立数据库和弹性池设计应用程序时的注意事项，请参阅[设计用于云灾难恢复的应用程序](/documentation/articles/sql-database-designing-cloud-solutions-for-disaster-recovery/)和[弹性池灾难恢复策略](/documentation/articles/sql-database-disaster-recovery-strategies-for-applications-with-elastic-pool/)。

<!---HONumber=Mooncake_1024_2016-->