<properties
    pageTitle="DocumentDB：DocumentDB API | Azure"
    description="了解如何通过 SQL 和 JavaScript 使用 DocumentDB 以低延迟来存储和查询大量 JSON 文档。"
    keywords="json 数据库，文档数据库"
    services="documentdb"
    author="mimig1"
    manager="jhubbard"
    editor="monicar"
    documentationcenter="" />
<tags
    ms.assetid="686cdd2b-704a-4488-921e-8eefb70d5c63"
    ms.service="documentdb"
    ms.workload="data-services"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.date="05/04/2017"
    wacn.date="05/31/2017"
    ms.author="mimig"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="4a18b6116e37e365e2d4c4e2d144d7588310292e"
    ms.openlocfilehash="b61efd6ffa93368f823572bea5547083a76f6351"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/19/2017" />

# <a name="introduction-to-azure-documentdb-documentdb-api"></a>DocumentDB 简介：DocumentDB API

[DocumentDB](/documentation/articles/documentdb-introduction/) 是 Microsoft 针对任务关键型应用程序提供的全局分布式多模型数据库服务。 DocumentDB 在全球范围内提供[统包全局分发](/documentation/articles/documentdb-distribute-data-globally/)、吞吐量和存储的弹性扩展、99% 的情况下低至个位数的毫秒级延迟、[五个妥善定义的一致性级别](/documentation/articles/documentdb-consistency-levels/)以及得到保证的高可用性，所有这些均由行业领先的 SLA 提供支持。 DocumentDB [自动为数据编制索引](http://www.vldb.org/pvldb/vol8/p1668-shukla.pdf)，无需客户管理架构和索引。 它采用多种模型，支持文档、键-值、图形和列式数据模型。 

借助 DocumentDB API，DocumentDB 可提供丰富且熟悉的 [SQL 查询功能](/documentation/articles/documentdb-sql-query/)，在传输无架构 JSON 数据过程中可持续保持较低的延迟。 本文概述了 DocumentDB 的 DocumentDB API，以及如何使用它来存储大量 JSON 数据，以毫秒级的延迟查询这些数据，以及轻松改进架构。 

## <a name="how-can-i-learn-about-the-documentdb-api"></a>如何了解有关 DocumentDB API 的信息？
了解 DocumentDB 的捷径是访问[查询板块](http://www.documentdb.com/sql/demo)，可以在其中完成不同的活动，以便了解 DocumentDB 中提供的丰富的查询功能。 接着前往“沙盒”选项卡并运行你自己的自定义 SQL 查询，并对 DocumentDB 进行试用。

然后，返回到本文，我们将在其中深入探究。  

## <a name="what-capabilities-and-key-features-does-azure-documentdb-offer"></a>DocumentDB 提供了哪些功能和主要特性？
DocumentDB 通过 DocumentDB API 提供了以下主要功能和优势：

- 可灵活增减吞吐量和存储：根据应用程序需求，轻松增大或减小 JSON 数据库规模。 你的数据存储在固态硬盘 (SSD) 上，以实现可预测的低延迟。 DocumentDB 支持使用容器来存储称为集合的 JSON 数据，这些数据集几乎可以扩展到无限存储大小和预配吞吐量。 随着应用程序规模的增长，可以灵活无缝地扩展 DocumentDB 且其性能可以预测。 


- 多区域复制：DocumentDB 以透明方式将数据复制到与 DocumentDB 帐户关联的所有区域，以便能够开发那些对全局性数据访问有要求的应用程序，与此同时还在一致性、可用性和性能方面做出权衡，所有这些都有相应的保证。 DocumentDB 提供具有多宿主 API 的透明区域故障转移，还可以灵活缩放全局吞吐量和存储。 有关详细信息，请参阅[使用 DocumentDB 全局分发数据](/documentation/articles/documentdb-distribute-data-globally/)。

- 使用熟悉的 SQL 语法进行即席查询：存储异类 JSON 文档，并通过熟悉的 SQL 语法查询这些文档。 DocumentDB 使用高并发、无锁、日志结构化索引技术为所有文档内容自动编制索引。 这样可以实现各种实时查询，而无需指定架构提示、二级索引或视图。 有关详细信息，请参阅[查询 DocumentDB](/documentation/articles/documentdb-sql-query/)。 
- **在数据库中执行 JavaScript：** 使用标准 JavaScript 将应用程序逻辑表示为存储过程、触发器和用户定义函数 (UDF)。 这样，你的应用程序逻辑可基于数据进行运作，而无需担心应用程序和数据库架构之间的不匹配。 DocumentDB API 支持在数据库引擎内部直接进行 JavaScript 应用程序逻辑的完全事务执行。 对 JavaScript 的深度集成支持在一个 JavaScript 程序中将 INSERT、REPLACE、DELETE 和 SELECT 操作作为独立的事务来执行。 有关详细信息，请参阅 [DocumentDB 服务器端编程](/documentation/articles/documentdb-programming/)。

- 可优化的一致性级别：从五个妥善定义的一致性级别中选择，实现一致性与性能之间的最佳平衡。 对于查询和读取操作，DocumentDB 提供五种不同的一致性级别：强一致性、有限过时一致性、会话一致性、前缀一致性和最终一致性。 通过这些细化的定义完好的一致性级别，你可以在一致性、可用性和延迟之间实现合理的平衡。 有关详细信息，请参阅[使用一致性级别最大化可用性和性能](/documentation/articles/documentdb-consistency-levels/)。

- **完全托管：** 无需管理数据库和计算机资源。 作为一种完全托管的 Azure 服务，用户无需管理虚拟机、部署并配置软件、管理缩放或处理复杂的数据层升级。 每个数据库都将自动备份，以防受到区域故障的影响。 可轻松添加 DocumentDB 帐户并根据需要预配容量，从而可以专注于应用程序而不是操作和管理数据库。 

- **源于设计的开放性：** 通过使用现有的技能和工具快速入门。 针对 DocumentDB API 的编程非常简单易学，无需使用新的工具或遵循 JSON 或 JavaScript 的自定义扩展。 你可以通过简单的 RESTful HTTP 接口访问所有数据库功能，包括 CRUD、查询和 JavaScript 处理。 DocumentDB API 包含现有格式、语言和标准，并同时基于这些内容提供高价值的数据库功能。

- 自动编制索引：默认情况下，DocumentDB 将自动为数据库中的所有文档编制索引，无需任何架构或创建二级索引。 不想索引所有内容？ 别担心，还可以 [退出 JSON 文件中的路径](/documentation/articles/documentdb-indexing-policies/) 。

## <a name="data-management"></a>如何使用 DocumentDB API 管理数据？
DocumentDB API 有助于通过定义完好的数据库资源管理 JSON 数据。 这些资源经过复制具有高可用性，并且使用其逻辑 URI 进行唯一寻址。 DocumentDB 为所有资源提供简单的基于 HTTP 的 RESTful 编程模型。 


DocumentDB 数据库帐户是授予对 DocumentDB 的访问权限的唯一命名空间。 在创建数据库帐户之前，你必须具有 Azure 订阅，以便为你提供访问各种 Azure 服务的权限。 

DocumentDB 中的所有资源都以 JSON 文档的形式建模和存储。 将资源作为项（包含元数据的 JSON 文档）进行管理，也可以作为源（项的集合）进行管理。 项集包含在它们各自的源中。

下图显示了 DocumentDB 资源之间的关系：

![DocumentDB 中的资源之间的层级关系][1] 

一个数据库帐户可以包含一组数据库，每个数据库都包含多个集合，每个集合又包含存储过程、触发器、UDF、文档及相关附件。 数据库也有关联的用户，每个用户都有一组权限来访问其他各种集合、存储过程、触发器、UDF、文档或附件。 尽管数据库、用户、权限和集合是系统定义的具有已知架构的资源，文档、存储过程、触发器、UDF 和附件也包含任意的用户定义的 JSON 内容。  

> [AZURE.NOTE]
> 由于 DocumentDB API 之前是作为 Azure DocumentDB 服务提供的，因此可以继续预配、监视和管理通过 Azure Resource Management REST API 或工具使用 Azure DocumentDB 或 DocumentDB 资源名称创建的帐户。 在提到 Azure DocumentDB API 时，我们会互换使用这些名称。 

## <a name="develop"></a> 如何使用 DocumentDB API 开发应用？

DocumentDB 通过 DocumentDB REST API 公开资源，可以使用能够发出 HTTP/HTTPS 请求的任何语言来调用此 API。 另外，还针对多种主流语言为 DocumentDB API 提供了编程库。 客户端库通过处理一些细节，例如地址缓存、异常管理、自动重试等，简化了使用 API 的许多方面。 当前这些库可用于以下语言和平台：  

| 下载 | 文档 |
| --- | --- |
| [.NET SDK](http://go.microsoft.com/fwlink/?LinkID=402989) |[.NET 库](https://msdn.microsoft.com/zh-cn/library/azure/dn948556.aspx) |
| [Node.js SDK](http://go.microsoft.com/fwlink/?LinkID=402990) |[Node.js 库](http://azure.github.io/azure-documentdb-node/) |
| [Java SDK](http://go.microsoft.com/fwlink/?LinkID=402380) |[Java 库](http://azure.github.io/azure-documentdb-java/) |
| [JavaScript SDK](http://go.microsoft.com/fwlink/?LinkID=402991) |[JavaScript 库](http://azure.github.io/azure-documentdb-js/) |
| 不适用 |[服务器端 JavaScript SDK](http://azure.github.io/azure-documentdb-js-server/) |
| [Python SDK](https://pypi.python.org/pypi/pydocumentdb) |[Python 库](http://azure.github.io/azure-documentdb-python/) |
| 不适用 | [适用于 MongoDB 的 API](/documentation/articles/documentdb-protocol-mongodb/)


使用 [DocumentDB 模拟器](/documentation/articles/documentdb-nosql-local-emulator/)可在本地通过 DocumentDB API 开发和测试应用程序，无需创建 Azure 订阅且不会产生任何费用。 如果对应用程序在模拟器中的工作情况感到满意，则可以切换到在云中使用 DocumentDB 帐户。

除了基本的创建、读取、更新和删除操作，DocumentDB API 还提供了丰富的 SQL 查询接口和服务器端支持，前者用于检索 JSON 文档，后者用于以事务形式执行 JavaScript 应用程序逻辑。 可通过所有平台库获取查询和脚本执行接口以及 REST API。 

### <a name="sql-query"></a>SQL 查询
DocumentDB API 支持使用 SQL 语言（来源于 JavaScript 类型系统）和支持关系、层级和空间查询的表达式来查询文档。 DocumentDB 查询语言是一种用于查询 JSON 文档的简单而强大的接口。 该语言支持部分 ANSI SQL 语法，并深度集成了 JavaScript 对象、数组、对象构造和函数调用。 DocumentDB 提供的查询模型没有任何显式架构或来自开发人员的索引提示。

可以在 DocumentDB API 中注册用户定义函数 (UDF)，并将其作为 SQL 查询的一部分进行引用，从而将语法扩展为支持自定义的应用程序逻辑。 这些 UDF 编写为 JavaScript 程序，并在数据库中执行。 

对于 .NET 开发人员，DocumentDB [.NET SDK](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.linq.aspx) 还提供了 LINQ 查询提供程序。 

### <a name="transactions-and-javascript-execution"></a>事务和 JavaScript 执行
DocumentDB API 允许将应用程序逻辑编写为完全使用 JavaScript 编写的命名程序。 这些程序是为集合注册的，可以对指定集合内的文档发布数据库操作。 可针对执行将 JavaScript 注册为触发器、存储过程或用户定义函数。 触发器和存储过程可以创建、读取、更新和删除文档，而用户定义函数作为查询执行逻辑的一部分执行，并且没有集合的写访问权限。

DocumentDB API 中的 JavaScript 执行是在关系型数据库系统所支持的概念的基础之上建立的，只是现代性的将 Transact-SQL 换成了 JavaScript。 所有 JavaScript 逻辑都在使用快照隔离的环境 ACID 事务内执行。 在其执行过程中，如果 JavaScript 引发异常，则整个事务将被中止。

## <a name="next-steps"></a>后续步骤
已有 Azure 帐户？ 那么可以在 [Azure 门户](https://portal.azure.cn/)中通过[创建 DocumentDB 数据库帐户](/documentation/articles/documentdb-create-account/)开始使用 DocumentDB。

[1]: ./media/documentdb-introduction/json-database-resources1.png

<!-- Update_Description: wording update -->