<!-- This page was renamed sql-data-warehouse-restore-database-overview in ACOM -->
<properties
   pageTitle="Azure SQL 数据仓库中的数据还原（概述）| Azure"
   description="在 Azure SQL 数据仓库中恢复数据库时的数据库还原选项概述。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="elfisher"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="05/05/2016"
   wacn.date="06/20/2016"/>


# Azure SQL 数据仓库中的数据还原（概述）

> [AZURE.SELECTOR]
- [概述](/documentation/articles/sql-data-warehouse-overview-manage-database-restore/)
- [门户](/documentation/articles/sql-data-warehouse-manage-database-restore-portal/)
- [PowerShell](/documentation/articles/sql-data-warehouse-manage-database-restore-powershell/)
- [REST](/documentation/articles/sql-data-warehouse-manage-database-restore-rest-api/)

描述在Azure SQL 数据仓库中还原数据库的选项。这些选项包括：还原实时数据库、还原已删除的数据库，以及还原无法访问的数据库。实时数据库和已删除的数据库在还原时使用同一数据中心提供的备份。无法访问的数据库可能是由于停电而脱机的。在这种情况下，将通过位于其他地理位置的数据中心进行还原。


## 恢复方案

**发生基础结构故障后恢复：**此方案是指在发生基础结构问题（例如磁盘故障等）后恢复。客户想要使用容错和高可用性基础结构来确保业务连续性。

**发生用户错误后恢复：**此方案是指发生意外或偶发的数据损坏或删除后进行恢复。如果用户无意中或不小心修改或删除了数据，客户希望能够将数据库还原到较早的时间点，以确保业务连续性。

**发生灾难后恢复 (DR)：**此方案是指发生重大灾难后进行恢复。在自然灾害或区域性停电等中断性事件造成数据库变得不可用的情况下，客户想要恢复不同区域的数据库，使业务得以持续运营。

## 备份策略

[AZURE.INCLUDE [SQL 数据仓库备份保留策略](../includes/sql-data-warehouse-backup-retention-policy)]


## 数据库还原功能

让我们看看 SQL 数据仓库如何增强数据库的可靠性，并在发生上述情况时实现恢复和持续运营。


### 数据冗余

由于 SQL 数据仓库会隔离计算和存储，因此所有数据将直接写入异地冗余的 Azure 存储空间 (RA-GRS)。地域冗余存储将数据复制到距主区域数百英里以外的辅助区域。在主要区域和次要区域，你的数据将分别复制到次要区域三次，并且是在不同的容错域和升级域之间复制。即使其中一个区域发生全区域性的停电或灾难，造成服务中断，也仍可确保数据的持久性。若要了解有关读取访问权限异地冗余存储的详细信息，请参阅 [Azure 存储冗余选项][]。

### 数据库还原

数据库点还原旨在将数据库还原到以前的某个时间点。Azure SQL 数据仓库服务至少每隔 8 小时使用自动存储快照来保护所有数据库，并将快照保留 7 天，为你提供一系列不同的还原点。这些备份存储在 RA-GRS Azure 存储空间中，因此，默认情况下是异地冗余的。自动备份和还原功能不收取额外的费用，提供零成本和免管理的方式来避免数据库遭到意外损坏或删除。若要了解有关数据库还原的详细信息，请参阅 [数据库还原任务][]。

### 异地还原

在由于发生中断性事件而使数据库不可用时，异地还原可以恢复数据库。你可以联系支持人员，以便从异地冗余的备份还原数据库，并在任一 Azure 区域中创建新数据库。由于备份是异地冗余的，即使由于停电而无法访问数据库，也能够使用备份来恢复数据库。可以在任何 Azure 区域中的任何服务器上创建已还原的数据库。除了发生中断后进行恢复，异地还原也可以用于其他情况，例如将数据库迁移或复制到不同的服务器或区域。

**何时启动恢复**
恢复操作在恢复时要求更改 SQL 连接字符串，并可能会导致数据永久丢失。因此，仅当中断的持续时间超过了应用程序的 RTO 时，才应执行恢复操作。使用以下数据点来声明有必要进行恢复：

- 数据库永久连接故障。
- Azure 门户显示了警报，指出区域中的某个事件会造成广泛的影响。


## 后续步骤
有关其他重要的管理任务，请参阅[管理概述][]；

<!--Image references-->

<!--Article references-->
[Azure 存储冗余选项]: /documentation/articles/storage-redundancy/#read-access-geo-redundant-storage
[Backup and restore tasks]: /documentation/articles/sql-data-warehouse-database-restore-portal/
[Finalize a recovered database]: /documentation/articles/sql-database-recovered-finalize/
[管理概述]: /documentation/articles/sql-database-business-continuity/

<!--MSDN references-->


<!--Other Web references-->

<!---HONumber=Mooncake_0613_2016-->