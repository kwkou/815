<properties
    pageTitle="DocumentDB: API for MongoDB 简介 | Azure"
    description="了解如何通过常用 OSS MongoDB API 使用 DocumentDB 以低延迟方式存储和查询大量 JSON 文档。"
    keywords="什么是 MongoDB"
    services="documentdb"
    author="AndrewHoh"
    manager="jhubbard"
    editor=""
    documentationcenter="" />
<tags
    ms.assetid="4afaf40d-c560-42e0-83b4-a64d94671f0a"
    ms.service="documentdb"
    ms.workload="data-services"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="05/05/2017"
    wacn.date="05/31/2017"
    ms.author="anhoh"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="4a18b6116e37e365e2d4c4e2d144d7588310292e"
    ms.openlocfilehash="aac8cd6e558da0944465dae7b446070504115eb4"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/19/2017" />

# <a name="introduction-to-azure-documentdb-api-for-mongodb"></a>DocumentDB: API for MongoDB 简介

[DocumentDB](/documentation/articles/documentdb-introduction/) 是 Microsoft 针对任务关键型应用程序提供的全局分布式多模型数据库服务。 DocumentDB 在全球范围内提供[统包全局分发](/documentation/articles/documentdb-distribute-data-globally/)、[吞吐量和存储的弹性缩放](/documentation/articles/documentdb-partition-data/)、99% 的情况下低至个位数的毫秒级延迟、[五个妥善定义的一致性级别](/documentation/articles/documentdb-consistency-levels/)以及得到保证的高可用性，所有这些均由行业领先的 SLA 提供支持。 DocumentDB [自动为数据编制索引](http://www.vldb.org/pvldb/vol8/p1668-shukla.pdf)，不需要你管理架构和索引。 它采用多种模型，支持文档、键-值、图形和列式数据模型。 

可以将 DocumentDB 数据库用作为 [MongoDB](https://docs.mongodb.com/manual/introduction/) 编写的应用的数据存储。 这意味着，通过使用现有[驱动程序](https://docs.mongodb.org/ecosystem/drivers/)，为 MongoDB 编写的应用程序现在可与 DocumentDB 通信，并使用 DocumentDB 数据库而不是 MongoDB 数据库。 在许多情况下，只需更改连接字符串便可从使用 MongoDB 切换到使用 DocumenetDB。 使用此功能，可以在 Azure 云中轻松生成和运行 MongoDB 数据库应用程序（利用 DocumentDB 在全球的分布以及内容广泛且行业领先的 SLA），并可继续使用熟悉的技能和 MongoDB 工具。


## <a name="what-is-the-benefit-of-using-azure-documentdb-for-mongodb-applications"></a>使用适用于 MongoDB 应用程序的 DocumentDB 有什么好处？

- 可灵活增减吞吐量和存储：根据应用程序需求，轻松增大或减小 MongoDB 数据库规模。 你的数据存储在固态硬盘 (SSD) 上，以实现可预测的低延迟。 DocumentDB 支持几乎可以扩展到无限存储大小和预配吞吐量的 MongoDB 集合。 随着应用程序规模的增长，可以灵活无缝地扩展 DocumentDB 且其性能可以预测。 

- 多区域复制：DocumentDB 以透明方式将数据复制到与 MongoDB 帐户关联的所有区域，使你可以开发那些对全局性数据访问有要求的应用程序，与此同时还在一致性、可用性和性能方面做出权衡，所有这些都有相应的保证。 DocumentDB 提供使用多宿主 API 的透明区域故障转移，还可以弹性缩放全局吞吐量和存储。 在[全局分发数据](/documentation/articles/documentdb-distribute-data-globally/)中了解详细信息。

**MongoDB 兼容性**：可以使用现有的 MongoDB 专业技能、应用程序代码和工具。 可以使用 MongoDB 来开发应用程序，然后使用完全托管的全局分布式 DocumentDB 服务将其部署到生产环境。

**没有服务器管理**：无需管理和缩放 MongoDB 数据库。 DocumentDB 是完全托管的服务，这意味着无需自行管理任何基础结构或虚拟机。 DocumentDB 已在 30 多个 [Azure 区域](https://azure.microsoft.com/regions/services/)推出。

- 可调整的一致性级别：从五个妥善定义的一致性级别中选择，实现一致性与性能之间的最佳平衡。 对于查询和读取操作，DocumentDB 提供五种不同的一致性级别：强、有限过时、会话、一致前缀和最终。 通过这些细化的定义完好的一致性级别，你可以在一致性、可用性和延迟之间实现合理的平衡。 有关详细信息，请参阅[使用一致性级别最大化可用性和性能](/documentation/articles/documentdb-consistency-levels/)。

- 自动编制索引：默认情况下，DocumentDB 自动为 MongoDB 数据库中的文档包含的所有属性编制索引，无需任何架构，也无需创建二级索引。

企业级 - DocumentDB 支持多个本地副本，可在本地或区域出现故障时，提供 99.99% 的可用性和数据保护。 DocumentDB 具有企业级[符合性认证](https://www.microsoft.com/trustcenter)和安全功能。 

## <a name="how-to-get-started"></a>如何入门

在 [Azure 门户](https://portal.azure.cn)中创建 DocumentDB 帐户，并将 MongoDB 连接字符串中的连接切换到新帐户。 

*这就是所有的操作！*

有关详细说明，请参阅[创建帐户](/documentation/articles/documentdb-create-account/)和[连接到帐户](/documentation/articles/documentdb-connect-mongodb-account/)。

## <a name="next-steps"></a>后续步骤

有关 DocumentDB 的 MongoDB API 的信息已集成到整个 DocumentDB 文档中，但可以通过以下指南来入门：

- 若要了解如何获取帐户连接字符串信息，请参阅[连接到 MongoDB 帐户](/documentation/articles/documentdb-connect-mongodb-account/)教程。
- 若要了解如何在 MongoChef 中创建 DocumentDB 数据库和 MongoDB 应用之间的连接，请参阅[将 MongoChef 与 DocumentDB 配合使用](/documentation/articles/documentdb-mongodb-mongochef/)教程。
- 按照[将数据迁移到具有 MongoDB 协议支持的 DocumentDB](/documentation/articles/documentdb-mongodb-migrate/) 教程将数据导入到 API for MongoDB 数据库。
- 使用 [Node.js](/documentation/articles/documentdb-mongodb-samples/) 生成第一个 MongoDB 的 API 应用。
- 使用 .[NET](/documentation/articles/documentdb-mongodb-application/) 生成第一个 MongoDB 的 API Web 应用。
- 使用 [Robomongo](/documentation/articles/documentdb-mongodb-robomongo/) 连接到 MongoDB 的 API 帐户。
- 了解你的操作将多少 RU 用于 [GetLastRequestStatistics 命令和 Azure 门户指标](/documentation/articles/documentdb-request-units/#GetLastRequestStatistics/)。
- 了解如何[配置全局分布的应用的读取首选项](/documentation/articles/documentdb-distribute-data-globally/#ReadPreferencesAPIforMongoDB/)。

<!---Update_Description: wording update -->