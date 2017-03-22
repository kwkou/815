<properties
    pageTitle="用作键值存储的 Azure DocumentDB — 成本概述 | Azure"
    description="了解将 Azure DocumentDB 用作键值存储的低成本。"
    keywords="键值存储"
    services="documentdb"
    author="ArnoMicrosoft"
    manager="jhubbard"
    editor=""
    tags=""
    documentationcenter="" />
    
<tags
    ms.assetid="7f765c17-8549-4509-9475-46394fc3a218"
    ms.service="documentdb"
    ms.workload="data-services"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/30/2017"
    wacn.date="03/22/2017"
    ms.author="acomet" />  


# 用作键值存储的 DocumentDB — 成本概述

Azure DocumentDB 是完全托管的全球分布式 NoSQL 数据库服务，用于轻松构建高度可用的大规模[全球分布式](/documentation/articles/documentdb-distribute-data-globally/)应用程序。默认情况下，DocumentDB 会有效地为它引入的所有数据编制索引。因此，可针对任何类型的数据执行快速一致的 [SQL](/documentation/articles/documentdb-sql-query/)（和 [JavaScript](/documentation/articles/documentdb-programming/)）查询。

本文介绍使用 DocumentDB 作为键/值存储执行简单写入和读取操作时产生的成本。写入操作包括文档的插入、替换、删除和更新插入。除了保证 99.99% 的高可用性以外，DocumentDB 还保证读取延迟小于 10 毫秒，（索引）写入延迟小于 15 毫秒，SLA 高达 99%。

## 为何使用请求单位 (RU)

DocumentDB 的性能基于分区的[预配请求](/documentation/articles/documentdb-programming/)单位 (RU) 数量。预配属于另一种粒度，根据每秒 RU 数（[请不要与每小时计费相混淆](/pricing/details/documentdb/)）购买。应将 RU 视为一种货币，用于简化应用程序所需吞吐量的预配过程。客户无需考虑读取和写入容量单位之间的差异。RU 的单一货币模型能够有效地在读取和写入之间分享预配的容量。这种预配的容量模型使服务能够提供可预测且一致的吞吐量，保证低延迟、高可用性。最后，我们使用 RU 来为吞吐量建模，但每个预配的 RU 还具有定义数量的资源（内存、核心）。每秒 RU 数不仅仅是 IOPS。

作为一种全球分布式数据库系统，DocumentDB 是仅限 Azure 的服务，除了高可用性以外，还在延迟、吞吐量和一致性方面提供 SLA。预配的吞吐量将应用到与 DocumentDB 数据库帐户关联的每个区域。对于读取，DocumentDB 提供多个妥善定义的[一致性级别](/documentation/articles/documentdb-consistency-levels/)供用户选择。

下表显示基于 1KB 和 100KB 文档大小执行读取和写入事务所需的 RU 数量。

|文档大小|1 次读取|1 次写入|
|-------------|------|-------|
|1 KB|1 RU|5 RU|
|100 KB|10 RU|50 RU|

## 读取和写入成本

请参阅[此处](/pricing/details/documentdb/)的详细信息。

## 后续步骤

请持续关注有关优化 DocumentDB 资源预配的新文章。同时，欢迎使用我们的 [RU 计算器](https://www.documentdb.com/capacityplanner)。

<!---HONumber=Mooncake_0313_2017-->
