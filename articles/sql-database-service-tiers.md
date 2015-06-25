<properties
   pageTitle="SQL Database 服务层"
   description="比较 Azure SQL Database 服务层的性能和业务连续性功能，以便在成本与功能之间找到适当的平衡点，既能根据需要进行缩放，又不会造成停机。"
   services="sql-database"
   documentationCenter=""
   authors="shontnew"
   manager="jeffreyg"
   editor="monicar"/>

<tags
   ms.service="sql-database"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="data-management"
   ms.date="04/15/2015"
   wacn.date="05/25/2015"
   ms.author="shkurhek"/>

# 服务层

[SQL Database](sql-database-technical-overview) 采用基本、标准和高级服务层。这三个服务层都提供 99.99% 的运行时间 [SLA](/support/legal/sla) 和可预测的性能、灵活的业务连续性选项、安全功能和按小时计费。由于每个服务层包含多个性能级别，你可以灵活选择最适合工作负载需求的级别。如果你需要向上或向下缩放，可以在 Azure 门户中轻松更改数据库层，且不会给应用程序造成任何中断。有关详细信息，请参阅[更改数据库服务层和性能级别](https://msdn.microsoft.com/zh-cn/library/azure/dn369872.aspx)。

> [AZURE.IMPORTANT] Web 和企业版即将停用。了解如何[升级 Web 和企业版](sql-database-upgrade-new-service-tiers)。

## 基本

基本层适用于事务工作负载较轻的应用程序。一个典型的用例就是，需要一个小型数据库并且在任意时间点只需执行一项操作的轻型应用程序。

**性能、大小和功能**


| 服务层 | 基本 |
| :-- | :-- |
| 数据库吞吐量单位 (DTU) | 5 |
| 最大数据库大小 | 2 GB |
| 时间点还原 (PITR) | 过去 7 天内的毫秒级时间点 |
| 灾难恢复 | 地域还原，还原到任意 Azure 区域 |
| 性能目标 | 每小时事务率 |


## 标准

标准层是用于处理多个事务工作负载的适中选项。与基本层相比，它的性能和内置业务连续性功能更强。一个典型的用例就是具有多个并发事务的应用程序。

**性能和大小**


| 服务层 | 标准 S0 | 标准 S1 | 标准 S2 | 标准 S3 |
| :-- | :-- | :-- | :-- | :-- |
| 数据库吞吐量单位 (DTU) | 10 | 20 | 50 | 100 |
| 最大数据库大小 | 250 GB | 250 GB | 250 GB | 250 GB |


**功能**


| 服务层 | 标准（S0、S1、S2、S3）|
| :-- | :-- |
| 时间点还原 (PITR) | 过去 14 天内的毫秒级时间点 |
| 灾难恢复 | 标准地域复制，1 个脱机辅助数据库 |
| 性能目标 | 每分钟事务率 |


## 高级

高级层适用于任务关键型应用程序。它提供最佳的性能并允许访问高级业务连续性功能，包括在最多 4 个所选 Azure 区域中进行活动地域复制。一个典型的用例就是，具有较高事务量和许多并发用户的任务关键型应用程序。

**性能和大小**


| 服务层 | 高级 P1 | 高级 P2 | 高级 P3 |
| :-- | :-- | :-- | :-- |
| 数据库吞吐量单位 (DTU) | 125 | 250 | 1000 |
| 最大数据库大小 | 500 GB | 500 GB | 500 GB |


**功能**


| 服务层 | 高级（P1、P2、P3）|
| :-- | :-- |
| 时间点还原 (PITR) | 过去 35 天内的毫秒级时间点 |
| 灾难恢复 | 活动地域复制，多达 4 个联机可读辅助数据库 |
| 性能目标 | 每秒事务率 |


[SQL Database 定价](/home/features/sql-database/#price)中提供了有关这些层的价格详细信息。

现在，你已了解有关 SQL Database 层的信息，欢迎单击[试用](/pricing/1rmb-trial)来试用这些层，并了解[如何创建第一个 SQL 数据库](sql-database-get-started)！

<!--HONumber=55-->