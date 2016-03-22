<properties 
   pageTitle="SQL 数据库业务连续性常见问题" 
   description="客户所提出的，有关 Azure SQL 数据库中用于实现业务连续性和灾难恢复的内置功能和可选功能的常见问题与解答。" 
   services="sql-database" 
   documentationCenter="" 
   authors="elfisher" 
   manager="jeffreyg" 
   editor="monicar"/>

<tags
   ms.service="sql-database"
   ms.date="11/16/2015"
   wacn.date="03/21/2016"/>

# 业务连续性常见问题

## 1\.当我降级/升级服务层时，我的还原点保留期会发生什么情况？
在降级到更低的性能层后，还原点的保留期会立即缩短到当前数据库性能层的保留期。

如果升级了数据库服务层，保留期只会在升级数据库之后才会开始延长。

例如，如果数据库从 P1 降级到 S3，保留期将从 35 天立即更改为 14 天，14 天前的所有还原点将不再可用。以后，如果将数据库再次升级到 P1，保留期最初仍为 14 天，然后再开始延长到 35 天。

## 2\.已删除的数据库的保留期有多长？ 
保留期由存在的数据库的服务层或数据库存在的天数确定（以两者中天数少的为准）。

## 3\.如何还原已删除的服务器？

目前不支持还原已删除的服务器。

## 4\.还原数据库需要多长时间？

还原数据库所需的时间取决于多种因素，例如数据库大小、事务日志数、网络带宽，等等。大部分数据库在 12 小时内即可还原。

## 5\.我是否可以更改数据库的还原点保留期？

不能。

## 6\.如何找出我的数据库的可用还原点？

对于发生用户错误后的恢复 - 当前时间就是最新的可用还原点。若要找出最早的可用还原点，请使用[获取数据库](https://msdn.microsoft.com/zh-cn/library/dn505708.aspx) (*RecoveryPeriodStartDate*) 获取最早的还原点（非异地复制的还原点）。

对于中断后的恢复 - 使用[获取可恢复数据库](https://msdn.microsoft.com/zh-cn/library/dn800985.aspx) (*LastAvailableBackupDate*) 获取最新的异地复制还原点。

## 7\.如何批量还原服务器中的数据库？

没有任何内置功能用于执行批量还原。[Azure SQL 数据库：完全恢复服务器](https://gallery.technet.microsoft.com/Azure-SQL-Database-Full-82941666)脚本是完成此任务的一种方法示例。

## 8\.标准异地复制与活动异地复制之间有什么差别？

对于标准异地复制，辅助数据库是不可读的。它只可用于在中断期间进行故障转移。

对于活动异地复制，所有辅助数据库都是可读的（最多 4 个辅助数据库）。

## 9\.使用标准异地复制或活动异地复制时有多长的复制延迟？

使用 [sys.dm\_geo\_replication\_link\_status](https://msdnstage.redmond.corp.microsoft.com/library/mt575504.aspx) 动态管理视图 (DMV) 可以获取上次复制时间、上次复制延迟和有关复制链接的其他信息。

<!---HONumber=Mooncake_1207_2015-->