<properties
   pageTitle="Azure SQL 数据库 Web 和企业版停用常见问题 | Azure"
   description="了解 Azure SQL Web 和企业数据库何时停用，并了解新服务层的特性和功能。"
   services="sql-database"
   documentationCenter="na"
   authors="stevestein"
   manager="jhubbard"
   editor="monicar" />
<tags
   ms.service="sql-database"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="data-management"
   ms.date="08/08/2016"
   wacn.date="12/19/2016"
   ms.author="sstein" />

# Web 和企业版停用常见问题

Azure SQL Web 和企业数据库现已停用。基本、标准、高级和弹性层将取代即将停用的 Web 和企业数据库。

为了帮助你升级 Web 和企业数据库，SQL 数据库服务会根据历史工作负荷，为每个数据库建议适当的服务层和性能级别（定价层）。

**若要获取定价层建议：**

- [使用 PowerShell 升级到 SQL 数据库 V12](/documentation/articles/sql-database-upgrade-server-powershell/)



## 为何 Azure 门户显示我的 Web 和企业版数据库已停用？

由于 Web 和企业版数据库在 2015 年 9 月后将不可用，因此门户将 Web 和企业数据库标记为“已停用”。“已停用”标签还旨在提醒你应该将所有 Web 和企业数据库升级到标准、基本或高级服务层。

## 最好是将我的现有 Web 或企业数据库升级到哪个新服务层？

为现有 Web 或企业数据库选择适当的新服务层和性能级别取决于应用程序的具体功能和性能要求。

## Azure 为何要引入新的服务层？

根据客户反馈，Azure SQL 数据库将引入新的服务层，以帮助客户更方便地为关系数据库工作负荷提供支持。这些新层的设计能够以七个级别提供可预测的性能，以满足从轻型到重型事务应用程序的需求。此外，这些新层还提供一系列业务连续性功能、更有利的运行时间 SLA、成本更低但容量更大的数据库，以及改进的计费体验。

## 我可以在哪里了解有关新服务层的详细信息？

有关新服务层和性能模型的详细信息，请参阅[服务层](/documentation/articles/sql-database-service-tiers/)。有关新服务层的详细定价信息，请参阅 [SQL 数据库定价](/pricing/details/sql-database/)。

## 基本、标准和高级版中将不会提供哪些特性或功能？

联合功能将随 Web 和企业版一起停用。我们建议需要扩大其数据库的客户改用或迁移到[弹性数据库工具](/documentation/articles/sql-database-elastic-scale-get-started/)以获取 [Azure SQL 数据库](/documentation/articles/sql-database-elastic-scale-get-started/)，以便简化使用分片的应用程序的生成和管理过程。.NET 客户端库允许应用程序定义将数据映射到分片的方式，以及将 OLTP 请求路由到相应数据库的方式。为了支持重新配置数据到各个分片的分布方式的管理操作，将随附 Azure 云服务模板，用户可以在自己的 Azure 订阅中托管该模板。除了[弹性数据库工具](/documentation/articles/sql-database-elastic-scale-get-started/)以外，Microsoft 还将根据与客户深层互动后了解到的信息，继续创建和发布[自定义分片模式与实践指南](https://msdn.microsoft.com/zh-cn/library/azure/dn764977.aspx)。需要扩大功能的新客户应查看[弹性数据库工具](/documentation/articles/sql-database-elastic-scale-get-started/)，和/或与 Azure 技术支持联系以评估自己的选项。

另外，Azure 正在改变高级数据库的数据库复制体验。以前的高级数据库配额存在限制，在 T-SQL 中使用 CREATE DATABASE … AS A COPY OF 会创建没有保留资源的“挂起高级”数据库，而这种数据库的收费费率与企业数据库相同。由于我们现在以更自由的形式提供高级配额，“挂起高级”数据库不再受支持。数据库副本现在将以与源数据库相同的版本和性能级别创建，并相应地进行计费。如果需要，客户可以选择将复制的数据库降级到其他服务层或性能级别，以降低成本。随着此版本的发行，现有的“挂起高级”数据库将转换为企业版。预计[时间点还原](/documentation/articles/sql-database-recovery-using-backups/#point-in-time-restore)的引入会减少生成数据库备份副本的需要。

## 基本、标准和高级版如何改进我的计费体验？

基本、标准和高级 Azure SQL 数据库按小时计费，可以增加或减少各个数据库。我们将会根据你为每个小时选择的最高服务层和性能级别，以固定的费率向你计费。此外，帐单中会细分性能级别（例如：“基本”、“S1”和“P2”），让你更轻松地查看在单个月份中每个性能级别使用数据库天数/小时数。Web 和企业数据库仍然根据数据库大小，使用数据库单元进行计费。请访问 [SQL 数据库定价页](/pricing/details/sql-database/)，以了解有关新服务层的定价及其差异的详细信息。


## 另请参阅

[Azure SQL 数据库](/documentation/services/sql-databases/)

[Web 和企业定价](/pricing/details/sql-database/)

[服务层](/documentation/articles/sql-database-service-tiers/)

<!---HONumber=Mooncake_Quality_Review_1202_2016-->