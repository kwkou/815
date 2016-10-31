<properties
   pageTitle="SQL 数据仓库还原 | Azure"
   description="在 Azure SQL 数据仓库中恢复数据库时的数据库还原选项概述。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="Lakshmi1812"
   manager="barbkess"
   editor=""/>  


<tags
   ms.service="sql-data-warehouse"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="data-services"
   ms.date="09/29/2016"
   wacn.date="10/31/2016"/>  



# SQL 数据仓库还原

> [AZURE.SELECTOR]
- [概述][]
- [门户][]
- [PowerShell][]
- [REST][]

SQL 数据仓库提供本地和异地还原功能，这是其数据仓库灾难恢复功能的一部分。使用数据仓库备份可以将数据仓库还原到主要区域的某个还原点，而使用异地冗余备份则可还原到另一地理区域。本文介绍了还原数据仓库的细节。

## 什么是数据仓库还原？

数据仓库还原是指通过现有数据仓库或已删除数据仓库的备份创建的新数据仓库。还原的数据仓库重新创建了在特定时间备份的数据仓库。由于 SQL 数据仓库属于分布式系统，因此数据仓库还原是从存储在 Azure blob 中的许多备份文件创建的。

数据库还原是任何业务连续性和灾难恢复策略的基本组成部分，因为数据库还原可以在意外损坏或删除数据后重新创建数据。

有关详细信息，请参阅：

-  [SQL 数据仓库备份](/documentation/articles/sql-data-warehouse-backups/)
-  [业务连续性概述](/documentation/articles/sql-database/sql-database-business-continuity/)

## 数据仓库还原点

使用 Azure 高级存储的好处是，SQL 数据仓库可以使用 Azure 存储 Blob 快照备份主数据仓库。每个快照都有一个还原点，代表启动快照的时间。若要还原数据仓库，请选择一个还原点，然后发出还原命令。

SQL 数据仓库始终将备份还原到新的数据仓库。可以保留还原的数据仓库和当前的数据仓库，也可以删除其中一个。若要将当前的数据仓库替换为还原的数据仓库，将其重命名即可。



## 异地冗余还原

如果使用的是异地冗余存储，则可将数据仓库还原到另一地理区域的[配对数据中心](/documentation/articles/best-practices-availability-paired-regions/)。从上次的每日备份还原数据仓库。

## 还原时间线

可以将数据库还原到过去 7 天的任何可用还原点。快照 4 到 8 小时启动一次，可供使用 7 天。快照超过 7 天将过期，其还原点不再可用。

## 还原费用

已还原的数据仓库的存储费用按 Azure 高级存储费率计算。

如果暂停还原的数据仓库，则存储费用按 Azure 高级存储费率计算。暂停的好处是不会对 DWU 计算资源收费。

有关 SQL 数据仓库定价的详细信息，请参阅 [SQL 数据仓库定价](/pricing/details/sql-data-warehouse/)。

## 还原的用途

数据仓库还原的主要用途是在意外丢失或损坏数据后恢复数据。

数据仓库还原还可用于保留那些时间超过 7 天的备份。还原备份以后，数据仓库处于联机状态，可以无限次将其暂停以节省计算费用。暂停的数据库按 Azure 高级存储费率收取存储费用。

## 相关主题

### 方案

- 有关业务连续性概述，请参阅[业务连续性概述](/documentation/articles/sql-database-business-continuity)


<!-- ### Tasks -->


若要执行数据仓库还原，请使用以下还原方式：

- Azure 门户：请参阅 [Restore a data warehouse using the Azure portal](/documentation/articles/sql-data-warehouse-restore-database-portal/)（使用 Azure 门户还原数据仓库）
- PowerShell cmdlets：请参阅 [Restore a data warehouse using PowerShell cmdlets](/documentation/articles/sql-data-warehouse-restore-database-powershell/)（使用 PowerShell cmdlet 还原数据仓库）
- REST API：请参阅 [Restore a data warehouse using the REST APIs](/documentation/articles/sql-data-warehouse-restore-database-rest-api/)（使用 REST API 还原数据仓库）

<!-- ### Tutorials -->


<!--Image references-->

<!--Article references-->
[Azure SQL Database business continuity overview]: /documentation/articles/sql-database-business-continuity
[概述]: /documentation/articles/sql-data-warehouse-restore-database-overview
[门户]: /documentation/articles/sql-data-warehouse-restore-database-portal
[PowerShell]: /documentation/articles/sql-data-warehouse-restore-database-powershell
[REST]: /documentation/articles/sql-data-warehouse-restore-database-rest-api

<!--MSDN references-->


<!--Other Web references-->

<!---HONumber=Mooncake_1024_2016-->