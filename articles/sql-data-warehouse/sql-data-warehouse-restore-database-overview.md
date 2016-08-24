<properties
   pageTitle="还原 Azure SQL 数据仓库（概述）| Azure"
   description="在 Azure SQL 数据仓库中恢复数据库时的数据库还原选项概述。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="sonyam"
   manager="barbkess"
   editor=""/>  


<tags
   ms.service="sql-data-warehouse"
   ms.date="06/28/2016"
   wacn.date="08/22/2016"/>  



# 还原 Azure SQL 数据仓库（概述）

> [AZURE.SELECTOR]
- [概述][]
- [门户][]
- [PowerShell][]
- [REST][]

Azure SQL 数据仓库可通过本地冗余存储和自动执行的备份来保护数据。自动化的备份提供了一种零管理方式来使数据库防范意外损坏或删除。如果用户无意中或不小心修改或删除了数据，则你可以将数据库还原到较早的时间点，以确保业务连续性。SQL 数据仓库使用 Azure 存储空间快照无缝备份数据库，无需任何停机时间。

## 自动化的备份

**活动**数据库会以至少每 8 小时一次的频率自动备份并保留 7 天。这使你可以将活动数据库还原到过去 7 天内的多个还原点之一。

暂停数据库之后，新备份会停止，以前的备份会在保留时间达到 7 天时删除。如果数据库的暂停时间超过 7 天，则会保存最近的备份，从而确保你始终至少具有一个备份。

删除数据库时，最后一个备份会保存 7 天。

对活动 SQL 数据仓库运行此查询可查看创建最后一个备份的时间：


	select top 1 *
	from sys.pdw_loader_backup_runs 
	order by run_id desc;


如果需要保留备份的时间超过 7 天，则可以仅仅将一个还原点还原到新数据库，然后可以选择暂停该数据库，以便只需为该备份的存储空间付费。

## 数据冗余

除了备份，SQL 数据仓库还可以通过[本地冗余 (LRS)][] Azure 高级存储来保护数据。将在本地数据中心内维护数据的多个同步副本，确保在发生局部故障时能够保证提供透明的数据保护。数据冗余可确保数据可以经受得住基础结构问题，例如磁盘故障等。数据冗余通过容错和高可用性基础结构来确保业务连续性。

## 还原数据库

还原 SQL 数据仓库是一种简单操作，可以在 Azure 门户中进行，也可以使用 PowerShell 或 REST API 自动进行。


## 后续步骤
若要了解 Azure SQL 数据库版本的业务连续性功能，请阅读 [Azure SQL 数据库业务连续性概述][]。

<!--Image references-->

<!--Article references-->
[Azure SQL 数据库业务连续性概述]: /documentation/articles/sql-database-business-continuity/
[本地冗余 (LRS)]: /documentation/articles/storage-redundancy/
[概述]: /documentation/articles/sql-data-warehouse-restore-database-overview/
[门户]: /documentation/articles/sql-data-warehouse-restore-database-portal/
[PowerShell]: /documentation/articles/sql-data-warehouse-restore-database-powershell/
[REST]: /documentation/articles/sql-data-warehouse-restore-database-rest-api/

<!--MSDN references-->


<!--Other Web references-->

<!---HONumber=Mooncake_0815_2016-->