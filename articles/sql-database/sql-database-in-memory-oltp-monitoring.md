<properties
	pageTitle="监视 XTP 内存中存储 | Azure"
	description="估算和监视 XTP 内存中存储用量与容量；解决容量错误 41805"
	services="sql-database"
	documentationCenter=""
	authors="jodebrui"
	manager="jhubbard"
	editor=""/>


<tags
	ms.service="sql-database"
	ms.date="02/11/2016"
	wacn.date="06/14/2016"/>


# 监视 In-Memory OLTP 存储

使用 In-Memory 时，内存优化表中的数据和表变量将驻留在内存中 OLTP 存储内。每个高级服务层都有最大的内存中存储大小，如 [SQL 数据库服务层](/documentation/articles/sql-database-service-tiers/#service-tiers-for-single-databases)一文中所述。一旦超过此限制，插入和更新操作可能会开始失败（错误 41805）。到时，你需要删除数据以回收内存，或升级数据库的性能层。

## 确定数据是否在内存中存储容量限制范围内

确定存储限制：有关不同高级服务层的存储限制的信息，请参阅 [SQL 数据库服务层](/documentation/articles/sql-database-service-tiers/#service-tiers-for-single-databases)一文。

估计内存优化表的内存要求，如同在 Azure SQL 数据库中估计 SQL Server 的内存要求一样。花费几分钟时间查看 [MSDN](https://msdn.microsoft.com/zh-cn/library/dn282389.aspx) 上的相关主题。

请注意，表和表变量行以及索引都将计入最大用户数据大小。此外，ALTER TABLE 需要足够的空间来创建新版的完整表及其索引。

## 监视和警报

可以在 Azure [管理门户](/documentation/articles/sql-database-service-tiers/#service-tiers-for-single-databases)中，通过[性能层的存储限制](https://manage.windowsazure.cn)百分比来监视内存中存储用量：

- 在“数据库”边栏选项卡上，找出“资源使用率”框并单击“编辑”。
- 然后选择度量值“In-Memory OLTP 存储”百分比。
- 若要添加警报，请单击“资源使用率”框以打开“度量值”边栏选项卡，然后单击“添加警报”。

或者，使用以下查询来显示内存中存储的使用率：

    select xtp_storage_percent from sys.dm_db_resource_stats


## 更正内存不足情况 - 错误 41805

内存不足会导致插入、更新和创建操作失败，并出现错误消息 41805。

错误消息 41805 指示内存优化表和表变量已超过大小上限。

若要解决此错误，请执行以下操作之一：


- 从内存优化表中删除数据，为此，可以将数据卸载到传统的基于磁盘的表；或者，
- 将服务层升级到具有足够内存中存储的服务层，使保存你需要保留在内存优化表中的数据。

## 后续步骤
了解有关[使用动态管理视图监视 Azure SQL 数据库](/documentation/articles/sql-database-monitoring-with-dmvs/)的详细信息

<!---HONumber=Mooncake_0606_2016-->