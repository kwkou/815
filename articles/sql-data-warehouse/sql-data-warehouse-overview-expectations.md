<!-- Remove create ticket function -->
<properties
   pageTitle="SQL 数据仓库的预期功能 | Azure"
   description="SQL 数据仓库公共功能摘要，以及正式版的目标。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="happynicolle"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="06/11/2016"
   wacn.date="07/18/2016"/>


# SQL 数据仓库的预期功能

本文介绍 SQL 数据仓库的功能，以及正式版 (GA) 的服务目标。在增强公共功能的过程中，我们将不断更新本文中的信息。

我们的 SQL 数据仓库目标：

- 可预测的性能以及 PB 量级数据的线性缩放性
- 所有数据仓库操作均具有高可靠性
- 短时间即可跨关系数据和非关系数据完成从数据加载到数据洞察在内的操作

在 SQL 数据仓库使用期间，我们会继续努力实现这些目标。

## 可预测且可缩放的性能

SQL 数据仓库引入了数据仓库单位 (DWU) 作为度量数据仓库可用计算资源（CPU、内存、存储 I/O）的方法。增加 DWU 数目可增加资源。当 DWU 数目增加时，SQL 数据仓库操作（例如数据加载或查询）可跨越更多分布式资源并行运行。这可减少延迟并改善性能。

任一数据仓库都有 2 个基本性能指标：

- 加载速率。每秒可载入数据仓库的记录数量。我们会专门度量可通过 PolyBase 从 Azure Blob 存储导入包含群集列存储索引的表中的记录数量。
- 扫描速率。每秒可从数据仓库依序检索的记录数量。我们会专门度量查询从包含群集列存储索引中返回的记录数量。

我们正在度量一些重要的性能增强功能，不久就能分享预期的速率。在预览期，我们将持续增强服务质量（例如提高压缩和缓存）以提高这些速率，并确保能够以可预测的方式调整这些速率。

## 数据保护

SQL 数据仓库将 Azure 中的所有数据存储在本地冗余存储中。将在本地数据中心内维护数据的多个同步副本，确保在发生局部故障时能够保证提供透明的数据保护。此外，SQL 数据仓库会使用 Azure 存储空间快照，定期自动备份活动（未暂停）数据库。若要详细了解备份和还原的工作原理，请参阅[备份和还原概述][]。

## 查询可靠性

SQL 数据仓库以大规模并行处理 (MPP) 体系结构为基础而构建。SQL 数据仓库可自动检测并迁移计算和控制节点故障。但是，操作（例如数据加载或查询）可能会由于节点故障或迁移而失败。在预览期，我们将持续增强，使节点故障不会影响到操作的成功完成。

## 升级和停机

SQL 数据仓库会定期进行升级，以便添加新功能和安装关键修补程序。这些升级可能会造成中断，而且在目前情况下，升级不是按预定计划进行的。如果你发现此过程造成过多的中断，则建议你[创建支持票证][]，由我们来帮助你解决此过程中的问题。

## 后续步骤

公共[入门][]。

<!--Image references-->

<!--Article references-->
[创建支持票证]: /support/support-ticket-form/?l=zh-cn
[入门]: /documentation/articles/sql-data-warehouse-get-started-provision-powershell/
[备份和还原概述]: /documentation/articles/sql-data-warehouse-restore-database-overview/

<!--MSDN references-->

<!--Other Web references-->

<!---HONumber=Mooncake_0711_2016-->