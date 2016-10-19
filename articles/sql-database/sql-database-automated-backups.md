<properties
   pageTitle="云业务连续性 - SQL 数据库的内置备份 |Azure"
   description="了解 SQL 数据库的内置备份，借助该备份可将 Azure SQL 数据库回退到以前的时间点，或者将数据库复制到地理区域中的新数据库（最多 35 天）。"
   services="sql-database"
   documentationCenter=""
   authors="CarlRabeler"
   manager="jhubbard"
   editor="monicar"/>

<tags
   ms.service="sql-database"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="09/06/2016"
   wacn.date="10/17/2016"
   ms.author="carlrab"/>  


# SQL 数据库自动备份

Azure SQL 数据库服务通过自动备份保护所有数据库，自动备份将分别保留 7 天（基本版）、35 天（标准版）和 35 天（高级版）。请参阅[服务层](/documentation/articles/sql-database-service-tiers/)，详细了解每个服务层提供的功能。

数据库备份是自动采用的，不需要选择加入和收取额外的费用。这些自动备份和时间点还原提供零成本、零管理的方式来保护数据库免于遭受意外损坏或删除，不论其原因是什么。在发生意外的数据损坏或删除之后，可以使用这些自动备份执行时间点还原以及还原已删除的数据库。

## 自动备份的成本

Azure SQL 数据库提供了高达你的备份存储的最大已设置数据库存储两倍的容量，不需要支付额外的成本。例如，如果你有一个标准数据库实例并且设置的数据库大小为 250 GB，则会向你提供 500 GB 的备份存储并且不额外收费。如果数据库超过提供的备份存储，则可以选择与 Azure 支持联系来缩短保留期，或针对按标准读取访问地域冗余存储 (RA-GRS) 费率计费的额外备份存储支付费用。

## 自动备份的详细信息

基本版、标准版和高级版的数据库全都受自动备份保护。完整数据库备份每周进行一次，差异数据库备份每小时进行一次，事务日志备份每 5 分钟进行一次。会在数据库创建后立即计划第一次完整备份。这通常会在 30 分钟内完成，但也可能花费更长时间。如果数据库已经很大，例如它的创建是大数据库复制或还原的结果，那么第一次完整备份可能要花费更长的时间才能完成。在进行了第一次完整备份后，将在后台自动计划和管理所有后续备份。完整备份和差异备份的确切时间由系统决定，以便平衡总体负载。

## 异地冗余

备份文件存储在具有读取访问权限 (RA-GRS) 的异地冗余存储帐户中，以便确保进行灾难恢复时可用。这样可确保将备份文件复制到配对数据中心。下面显示了对每周备份和每日备份进行的异地复制，这些备份存储在具有读取访问权限 (RA-GRS) 的异地冗余存储帐户中，确保进行灾难恢复时可用。

![异地还原](./media/sql-database-geo-restore/geo-restore-1.png)  


## 使用自动化的备份

可以在[保留期](/documentation/articles/sql-database-service-tiers/)内[从自动化的备份将数据库还原](/documentation/articles/sql-database-recovery-using-backups/)到：

- 恢复到保留期内指定时间点的同一逻辑服务器上的新数据库。
- 恢复到已删除数据库的删除时间的同一逻辑服务器上的数据库。
- 任何区域中恢复到异地复制 blob 存储 (RA-GRS) 的最新每日备份的任何逻辑服务器上的新数据库。

还可以使用 [SQL 数据库自动备份](/documentation/articles/sql-database-automated-backups/)在任何区域中的任何逻辑服务器上创建在事务上与当前 SQL 数据库一致的[数据库副本](/documentation/articles/sql-database-copy/)。可以使用数据库副本和[导出到 BACPAC](/documentation/articles/sql-database-export-powershell/) 将事务上一致的数据库副本存档以便在保留期以外长期存储，或者将数据库副本传输到本地或 SQL Server 的 Azure VM 实例。

## 降级/升级服务层时，还原点保留期会发生什么情况？

在降级到更低的性能层后，还原点的保留期会立即缩短到当前数据库性能层的保留期。如果升级了服务层，保留期只会在升级数据库之后才会开始延长。例如，如果将数据库降级到“基本”，保留期将从 35 天立即更改为 7 天，35 天前的所有还原点将不再可用。以后，如果将数据库升级到“标准”或“高级”，保留期最初仍为 7 天，然后再开始延长到 35 天。

## 已删除的数据库的保留期有多长？ 
保留期由存在的数据库的服务层或数据库存在的天数确定（以两者中天数少的为准）。

> [AZURE.IMPORTANT] 如果删除 Azure SQL 数据库服务器实例，其所有数据库也会一并删除，并且无法恢复。目前不支持还原已删除的服务器。

## 后续步骤

- 若要了解如何使用自动备份进行恢复，请参阅[从服务启动的备份中还原数据库](/documentation/articles/sql-database-recovery-using-backups/)
- 若要了解更快的恢复选项，请参阅[活动异地复制](/documentation/articles/sql-database-geo-replication-overview/)
- 若要了解如何使用自动备份进行存档，请参阅[数据库复制](/documentation/articles/sql-database-copy/)
- 有关业务连续性概述，请参阅[业务连续性概述](/documentation/articles/sql-database-business-continuity/)

<!---HONumber=Mooncake_1010_2016-->