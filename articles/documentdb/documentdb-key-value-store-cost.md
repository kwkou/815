<properties
    pageTitle="用作键值存储的 DocumentDB - 费用概述 | Azure"
    description="了解将 DocumentDB 用作键值存储时的低成本。"
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
    wacn.date="05/31/2017"
    ms.author="acomet"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="4a18b6116e37e365e2d4c4e2d144d7588310292e"
    ms.openlocfilehash="c8f375ad90a70f4826e30007a3e7ca5937e733c8"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/19/2017" />

# <a name="azure-documentdb-as-a-key-value-store---cost-overview"></a>用作键值存储的 DocumentDB - 费用概述

DocumentDB 是全球范围的分布式多模型数据库服务，用于轻松构建高度可用的大规模应用程序。 默认情况下，DocumentDB 会自动为其引入的所有数据高效编制索引。 因此，可针对任何类型的数据执行快速一致的 [SQL](/documentation/articles/documentdb-sql-query/)（和 [JavaScript](/documentation/articles/documentdb-programming/)）查询。 

本文介绍使用 DocumentDB 作为键/值存储执行简单写入和读取操作时产生的成本。 写入操作包括文档的插入、替换、删除和更新插入。 除了保证 99.99% 的高可用性以外，DocumentDB 还保证读取延迟小于 10 毫秒，（索引）写入延迟小于 15 毫秒，SLA 高达 99%。 

## <a name="why-we-use-request-units-rus"></a>为何使用请求单位 (RU)

DocumentDB 的性能基于分区的预配[请求单位](/documentation/articles/documentdb-request-units/) (RU) 数量。 预配属于另一种粒度，根据每秒 RU 数（[请不要与每小时计费相混淆](/pricing/details/documentdb/)）购买。 应将 RU 视为一种货币，用于简化应用程序所需吞吐量的预配过程。 客户无需考虑读取和写入容量单位之间的差异。 RU 的单一货币模型能够有效地在读取和写入之间分享预配的容量。 这种预配的容量模型使服务能够提供可预测且一致的吞吐量，保证低延迟、高可用性。 最后，使用 RU 为吞吐量建模，但每个预配的 RU 还具有定义数量的资源（内存、核心）。 每秒 RU 数不仅仅是 IOPS。

作为一种全球分布式数据库系统，DocumentDB 是唯一除提供高可用性外还在延迟、吞吐量和一致性方面提供 SLA 的 Azure 服务。 预配的吞吐量将应用到与 DocumentDB 数据库帐户关联的每个区域。 对于读取，DocumentDB 提供多个妥善定义的[一致性级别](/documentation/articles/documentdb-consistency-levels/)供用户选择。 DocumentDB 是全球分布式多模型数据库服务，用于轻松构建高度可用的大规模[全球分布式](/documentation/articles/documentdb-distribute-data-globally/)应用程序。 默认情况下，DocumentDB 会自动为它引入的所有数据高效地编制索引。 因此，可针对任何类型的数据执行快速一致的 [SQL](/documentation/articles/documentdb-sql-query/)（和 [JavaScript](/documentation/articles/documentdb-programming/)）查询。 

本文介绍使用 DocumentDB 作为键/值存储执行简单写入和读取操作时产生的成本。 写入操作包括文档的插入、替换、删除和更新插入。 除了保证 99.99% 的高可用性以外，DocumentDB 还保证读取延迟小于 10 毫秒，（索引）写入延迟小于 15 毫秒，SLA 高达 99%。 

## <a name="why-we-use-request-units-rus"></a>为何使用请求单位 (RU)

DocumentDB 的性能基于分区的预配[请求单位](/documentation/articles/documentdb-request-units/) (RU) 数量。 预配属于另一种粒度，根据每秒 RU 数和每分钟 RU 数（[请不要与每小时计费相混淆](/pricing/details/documentdb/)）购买。 应将 RU 视为一种货币，用于简化应用程序所需吞吐量的预配过程。 客户无需考虑读取和写入容量单位之间的差异。 RU 的单一货币模型能够有效地在读取和写入之间分享预配的容量。 这种预配的容量模型使服务能够提供可预测且一致的吞吐量，保证低延迟、高可用性。 最后，使用 RU 为吞吐量建模，但每个预配的 RU 还具有定义数量的资源（内存、核心）。 每秒 RU 数不仅仅是 IOPS。

作为一种全球分布式数据库系统，DocumentDB 是唯一除提供高可用性外还在延迟、吞吐量和一致性方面提供 SLA 的 Azure 服务。 预配的吞吐量将应用到与 DocumentDB 数据库帐户关联的每个区域。 对于读取，DocumentDB 提供多个妥善定义的[一致性级别](/documentation/articles/documentdb-consistency-levels/)供用户选择。 

下表显示基于 1KB 和 100KB 文档大小执行读取和写入事务所需的 RU 数量。

|文档大小|1 次读取|1 次写入|
|-------------|------|-------|
|1 KB|1 RU|5 RU|
|100 KB|10 RU|50 RU|

## <a name="cost-of-reads-and-writes"></a>读取和写入成本

请参阅[此处](/pricing/details/documentdb/)的详细信息。

## <a name="next-steps"></a>后续步骤

请持续关注有关优化 DocumentDB 资源预配的新文章。 同时，欢迎使用我们的 [RU 计算器](https://www.documentdb.com/capacityplanner)。

<!-- Update_Description: wording update -->