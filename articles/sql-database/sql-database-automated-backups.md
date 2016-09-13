<properties
   pageTitle="云业务连续性 - SQL 数据库的内置备份 | Azure"
   description="了解 SQL 数据库的内置备份，借助该备份可将 Azure SQL 数据库回滚到以前的时间点，或者将数据库复制到地理区域中的新数据库（最多 35 天）。"
   services="sql-database"
   documentationCenter=""
   authors="carlrabeler"
   manager="jhubbard"
   editor="monicar"/>

<tags
   ms.service="sql-database"
   ms.date="06/16/2016"
   wacn.date="07/11/2016"/>

# 概述：SQL 数据库自动备份

Azure SQL 数据库服务通过自动备份保护所有数据库，自动备份将分别保留 7 天（基本版）、14 天（标准版）和 35 天（高级版）。在发生意外的数据损坏或删除之后，可以使用这些自动备份执行时间点还原以及还原已删除的数据库。

数据库备份是自动采用的，不需要选择加入和收取额外的费用。这些自动备份和时间点还原提供零成本、零管理的方式来保护数据库免于遭受意外损坏或删除，不论其原因是什么。

## 自动备份的成本

Azure SQL 数据库提供了高达你的备份存储的最大已设置数据库存储两倍的容量，不需要支付额外的成本。例如，如果你有一个标准数据库实例并且设置的数据库大小为 250 GB，则会向你提供 500 GB 的备份存储并且不额外收费。如果数据库超过提供的备份存储，则可以选择与 Azure 支持联系来缩短保留期，或针对按标准读取访问地域冗余存储 (RA-GRS) 费率计费的额外备份存储支付费用。

## 自动备份的详细信息

基本版、标准版和高级版的数据库全都受自动备份保护。完整备份每周进行一次，差异备份每天进行一次，日志备份每 5 分钟进行一次。会在数据库创建后立即计划第一次完整备份。这通常会在 30 分钟内完成，但也可能花费更长时间。如果数据库已经很大，例如它的创建是大数据库复制或还原的结果，那么第一次完整备份可能要花费更长的时间才能完成。在进行了第一次完整备份后，将在后台自动计划和管理所有后续备份。完整备份和差异备份的确切时间由系统决定，以便平衡总体负载。备份文件存储在具有读取访问权限 (RA-GRS) 的异地冗余存储帐户中，以便确保进行灾难恢复时可用。

## 后续步骤

- [业务连续性概述](/documentation/articles/sql-database-business-continuity)
- [还原已删除的数据库](/documentation/articles/sql-database-restore-deleted-database-powershell/)
- [时间点还原](/documentation/articles/sql-database-point-in-time-restore)
- [异地还原](/documentation/articles/sql-database-geo-restore)
- [活动异地复制](/documentation/articles/sql-database-geo-replication-overview)
- [数据库复制](/documentation/articles/sql-database-copy)

## 其他资源

- [在中断后恢复](/documentation/articles/sql-database-disaster-recovery)
- [从用户错误中恢复](/documentation/articles/sql-database-user-error-recovery)
- [执行灾难恢复演练](/documentation/articles/sql-database-disaster-recovery-drills)
- [设计用于云灾难恢复的应用程序](/documentation/articles/sql-database-designing-cloud-solutions-for-disaster-recovery)

<!---HONumber=Mooncake_0704_2016-->