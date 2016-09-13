<properties
   pageTitle="云业务连续性 - 异地还原 | Azure"
   description="了解 Azure SQL 数据库如何支持云业务连续性和数据库恢复以及如何帮助保持运行任务关键型云应用程序。"
   services="sql-database"
   documentationCenter=""
   authors="stevestein"
   manager="jhubbard"
   editor="monicar"/>

<tags
   ms.service="sql-database"
   ms.date="06/17/2016"
   wacn.date="07/18/2016"/>
# 概述：从异地冗余备份还原 Azure SQL 数据库

> [AZURE.SELECTOR]
- [业务连续性概述](/documentation/articles/sql-database-business-continuity/)
- [时间点还原](/documentation/articles/sql-database-point-in-time-restore/)
- [还原已删除的数据库](/documentation/articles/sql-database-restore-deleted-database-powershell/)
- [异地还原](/documentation/articles/sql-database-geo-restore/)
- [活动异地复制](/documentation/articles/sql-database-geo-replication-overview/)
- [业务连续性方案](/documentation/articles/sql-database-business-continuity-scenarios/)


使用异地还原，你可以在任何 Azure 区域的任何服务器上从最新的[日常自动备份](/documentation/articles/sql-database-automated-backups/)还原 SQL 数据库。异地还原使用异地冗余备份作为源，即使由于停电而无法访问数据库或数据中心，也依然能够使用它来恢复数据库。可以使用 [PowerShell](/documentation/articles/sql-database-geo-restore-powershell/) 或 [REST (createMode=Restore)](https://msdn.microsoft.com/zh-cn/library/azure/mt163685.aspx)

> [AZURE.SELECTOR]
- [概述](/documentation/articles/sql-database-geo-restore/)
- [PowerShell](/documentation/articles/sql-database-geo-restore-powershell/)

当你的数据库因其所在的区域发生事故而不可用时，异地还原是默认的恢复选项。可以在任意 Azure 区域中的任何服务器上创建数据库。异地还原依赖于异地冗余 Azure 存储空间中的[自动数据库备份](/documentation/articles/sql-database-automated-backups/)，可以从异地复制的备份副本进行还原，因此可以弹性应对主要区域的存储中断情况。

## 异地还原详述

异地还原使用与时间点还原一样的技术，但二者存在一个重要差异。它从异地复制的 blob 存储 (RA-GRS) 中的最新每日备份的副本还原数据库。对于每个活动数据库，服务将维护一个备份链，其中包括每周完整备份、多个每日差异备份，以及每 5 分钟保存一次的事务日志。这些 blob 是异地复制的，这样可确保即使在主要区域发生大规模故障后，每日备份仍可用。下面显示了复制到存储容器的每周和每日备份的异地复制。

![异地还原](./media/sql-database-geo-restore/geo-restore-1.png)


如果区域中出现的大规模事件导致你的数据库应用程序不可用，你可以使用异地还原，将数据库从最新备份还原到任何其他区域中的服务器。所有备份是异地复制的，并且执行备份这一操作与异地复制到不同区域中的 Azure blob 这一操作之间可能会存在延迟。此延迟可能长达一小时，因此发生灾难时，会有长达 1 小时的数据丢失风险，即最多 1 小时的 RPO。下面显示了从上次的每日备份进行的数据库还原。


![异地还原](./media/sql-database-geo-restore/geo-restore-2.png)

使用[获取可恢复数据库](https://msdn.microsoft.com/zh-cn/library/dn800985.aspx) (LastAvailableBackupDate) 获取最新的异地复制还原点。

## 异地还原的恢复时间

恢复时间受几个因素影响：数据库的大小、数据库的性能级别以及目标区域中正在处理的并行恢复请求的数目。如果一个区域出现长时间的服务中断，则可能是因为存在大量正在由其他区域处理的异地还原请求。如果存在大量请求，则可能会延长该区域中数据库还原的时间。还原数据库所需的时间取决于多种因素，例如数据库大小、事务日志数、网络带宽，等等。大部分数据库还原操作可在 12 小时内完成。

## 摘要

异地还原在所有服务层均可用，并且异地还原是 SQL 数据库提供的灾难恢复解决方案中最基本的，具有最长的 RPO 和估计恢复时间 (ERT)。对于最大为 2GB 的基本数据库，异地还原提供了 ERT 为 12 小时的合理灾难恢复解决方案。对于较大的标准或高级数据库，如果需要在很短的恢复时间内恢复，或要减少数据丢失的可能性，应考虑使用活动异地复制。活动异地复制可提供低得多的 RPO 和 ERT，因为它只要求你启动故障转移，故障转移到连续复制的辅助数据库。有关详细信息，请参阅[活动异地复制](/documentation/articles/sql-database-geo-replication-overview/)。

## 后续步骤

- 有关如何使用 PowerShell 从异地冗余备份还原 Azure SQL 数据库的详细步骤，请参阅[使用 PowerShell 进行异地还原](/documentation/articles/sql-database-geo-restore-powershell/)
- 有关如何在中断后进行恢复的完整讨论，请参阅[在中断后恢复](/documentation/articles/sql-database-disaster-recovery/)

## 其他资源

- [业务连续性方案](/documentation/articles/sql-database-business-continuity-scenarios/)

<!---HONumber=Mooncake_0711_2016-->