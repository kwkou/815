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
   ms.date="05/10/2016"
   wacn.date="06/14/2016"/>

# 概述：SQL 数据库异地还原

异地还原让你能够从最新的每日备份还原 SQL 数据库，并自动为所有服务层启用，而无需支付额外费用。异地还原使用异地冗余备份作为源，即使由于停电而无法访问数据库或数据中心，也依然能够使用它来恢复数据库。

启动异地还原将创建一个新 SQL 数据库，此数据库可在任何 Azure 区域中的任何服务器上创建。


|任务（门户） | PowerShell | REST |
|:--|:--|:--|
| Recover a SQL database from a copy in a different region（从其他区域中的副本恢复 SQL 数据库） | [PowerShell](/documentation/articles/sql-database-geo-restore-powershell/) | [REST (createMode=Restore)](https://msdn.microsoft.com/zh-cn/library/azure/mt163685.aspx) |



当你的数据库因其所在的区域发生事故而不可用时，异地还原会提供默认的恢复选项。与[时间点还原](/documentation/articles/sql-database-point-in-time-restore/)一样，异地还原依赖异地冗余 Azure 存储空间中的数据库备份。它会从异地复制的备份副本中还原，因此可以灵活应对主要区域中的存储中断。



## 异地还原详述

异地还原使用与时间点还原一样的技术，但二者存在一个重要差异。它从异地复制的 blob 存储 (RA-GRS) 中的最新每日备份的副本还原数据库。对于每个活动数据库，服务将维护一个备份链，其中包括每周完整备份、多个每日差异备份，以及每 5 分钟保存一次的事务日志。这些 blob 是异地复制的，这样可确保即使在主要区域发生大规模故障后，每日备份仍可用。下面显示了复制到存储容器的每周和每日备份的异地复制。

![异地还原](./media/sql-database-geo-restore/geo-restore-1.png)


如果区域中出现的大规模事件导致你的数据库应用程序不可用，你可以使用异地还原，将数据库从最新备份还原到任何其他区域中的服务器。所有备份是异地复制的，并且执行备份这一操作与异地复制到不同区域中的 Azure blob 这一操作之间可能会存在延迟。此延迟可能长达一小时，因此发生灾难时，会有长达 1 小时的数据丢失风险，即最多 1 小时的 RPO。下面显示了从上次的每日备份进行的数据库还原。


![异地还原](./media/sql-database-geo-restore/geo-restore-2.png)



## 异地还原的恢复时间

恢复时间受几个因素影响：数据库的大小、数据库的性能级别以及目标区域中正在处理的并行恢复请求的数目。如果一个区域出现长时间的服务中断，则可能是因为存在大量正在由其他区域处理的异地还原请求。如果存在大量请求，则可能会延长该区域中数据库还原的时间。


## 摘要

异地还原在所有服务层均可用，并且异地还原是 SQL 数据库提供的灾难恢复解决方案中最基本的，具有最长的 RPO 和估计恢复时间 (ERT)。对于最大为 2GB 的基本数据库，异地还原提供了 ERT 为 12 小时的合理灾难恢复解决方案。对于较大的标准或高级数据库，如果需要在很短的恢复时间内恢复，或要减少数据丢失的可能性，应考虑使用活动异地复制。活动异地复制可提供低得多的 RPO 和 ERT，因为它只要求你启动故障转移，故障转移到连续复制的辅助数据库。有关详细信息，请参阅[活动异地复制](/documentation/articles/sql-database-geo-replication-overview/)。

## 其他资源

- [SQL 数据库 BCDR 常见问题](/documentation/articles/sql-database-bcdr-faq/)
- [业务连续性概述](/documentation/articles/sql-database-business-continuity/)
- [时间点还原](/documentation/articles/sql-database-point-in-time-restore/)
- [活动异地复制](/documentation/articles/sql-database-geo-replication-overview/)
- [设计用于云灾难恢复的应用程序](/documentation/articles/sql-database-designing-cloud-solutions-for-disaster-recovery/)
- [确认已恢复的 Azure SQL 数据库](/documentation/articles/sql-database-recovered-finalize/)

<!---HONumber=Mooncake_0530_2016-->
