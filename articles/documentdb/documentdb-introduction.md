<properties
    pageTitle="DocumentDB 简介：一种 JSON 数据库 | Azure"
    description="了解 Azure DocumentDB，一种 NoSQL JSON 数据库。 此文档数据库是针对大数据、灵活的可扩展性和高可用性构建的。"
    keywords="json 数据库，文档数据库"
    services="documentdb"
    author="mimig1"
    manager="jhubbard"
    editor="monicar"
    documentationcenter=""
    translationtype="Human Translation" />
    
<tags
    ms.assetid="686cdd2b-704a-4488-921e-8eefb70d5c63"
    ms.service="documentdb"
    ms.workload="data-services"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.date="03/14/2017"
    wacn.date="04/17/2017"
    ms.author="mimig"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="1e77b0dd999c7613ce7b2d1541136267189d18bd"
    ms.lasthandoff="04/07/2017" />

# <a name="introduction-to-documentdb-a-nosql-json-database"></a>DocumentDB 简介：一种 NoSQL JSON 数据库
## <a name="what-is-documentdb"></a>什么是 DocumentDB？
DocumentDB 是一个完全托管的 NoSQL 数据库服务，其构建目的是为了实现快速且可预测的性能、高可用性、弹性缩放、全局分发和易于开发。 作为一种无架构的 NoSQL 数据库，DocumentDB 提供了丰富且熟悉的 SQL 查询功能，对 JSON 数据具有一致的低延迟，可确保 99% 的读取在不到 10 毫秒的时间内得到处理，99% 的写入在不到 15 毫秒的时间内得到处理。 这些独有的优势使得 DocumentDB 非常适合于 Web、移动、游戏和 IoT 应用程序，以及其他需要无缝缩放和全局复制的许多应用程序。

## <a name="how-can-i-learn-about-documentdb"></a>如何了解 DocumentDB？
了解 DocumentDB 的捷径是访问[查询板块](http://www.documentdb.com/sql/demo)，可以在其中完成不同的活动，以便了解 DocumentDB 中提供的丰富的查询功能。 接着前往“沙盒”选项卡并运行你自己的自定义 SQL 查询，并对 DocumentDB 进行试用。

然后，返回到本文，我们将在其中深入探究。  

## <a name="what-capabilities-and-key-features-does-documentdb-offer"></a>DocumentDB 提供哪些功能和主要特性？
Azure DocumentDB 具有以下主要功能和优势：

- **可灵活增减的吞吐量和存储：** 轻松增大或减小 DocumentDB JSON 数据库规模来满足你的应用程序需求。 你的数据存储在固态硬盘 (SSD) 上，以实现可预测的低延迟。 DocumentDB 支持使用容器来存储称为集合的 JSON 数据，这些数据可以扩展到几乎无限的存储空间大小和设置的吞吐量。 随着应用程序规模的增长，你可以灵活无缝地扩展具有可预测的性能的 DocumentDB。 
- **多区域复制：** DocumentDB 以透明方式将数据复制到与 DocumentDB 帐户关联的所有区域，使用户可以开发那些对全局性数据访问有要求的应用程序，与此同时还在一致性、可用性和性能方面做出权衡，所有这些都有相应的保证。 DocumentDB 提供了具有多宿主 API 的透明区域故障转移，还可以弹性缩放全局吞吐量和存储空间。 有关详细信息，请参阅 [使用 DocumentDB 全局分发数据](/documentation/articles/documentdb-distribute-data-globally/)。
- **使用熟悉的 SQL 语法进行即席查询：** 在 DocumentDB 中存储异类 JSON 文档，并通过熟悉的 SQL 语法查询这些文档。 DocumentDB 使用高并发、无锁、日志结构化索引技术为所有文档内容自动创建索引。 这样可以实现各种实时查询，而无需指定架构提示、二级索引或视图。 有关详细信息，请参阅 [DocumentDB 查询](/documentation/articles/documentdb-sql-query/)。 
- **在数据库中执行 JavaScript：** 使用标准 JavaScript 将应用程序逻辑表示为存储过程、触发器和用户定义函数 (UDF)。 这样，你的应用程序逻辑可基于数据进行运作，而无需担心应用程序和数据库架构之间的不匹配。 DocumentDB 支持在数据库引擎内部直接进行 JavaScript 应用程序逻辑的完全事务执行。 对 JavaScript 的深度集成支持在一个 JavaScript 程序中将 INSERT、REPLACE、DELETE 和 SELECT 操作作为独立的事务来执行。 有关详细信息，请参阅 [DocumentDB 服务器端编程](/documentation/articles/documentdb-programming/)。
- **可调整的一致性级别：** 从 4 个定义完好的一致性级别中选择，以实现一致性和性能之间的最佳平衡。 对于查询和读取操作，DocumentDB 提供了四种不同的一致性级别：强、有限过时、会话和最终。 通过这些细化的定义完好的一致性级别，你可以在一致性、可用性和延迟之间实现合理的平衡。 有关详细信息，请参阅 [使用一致性级别最大化 DocumentDB 中的可用性和性能](/documentation/articles/documentdb-consistency-levels/)。
- **完全托管：** 无需管理数据库和计算机资源。 作为一种完全托管的 Azure 服务，用户无需管理虚拟机、部署并配置软件、管理缩放或处理复杂的数据层升级。 每个数据库都将自动备份，以防受到区域故障的影响。 你可以轻松添加 DocumentDB 帐户并按照你的需求设置容量，从而使你专注于你的应用程序而不是操作和管理你的数据库。 
- **源于设计的开放性：** 通过使用现有的技能和工具快速入门。 针对 DocumentDB 的编程非常简单易学，你无需使用新的工具或遵循 JSON 或 JavaScript 的自定义扩展。 你可以通过简单的 RESTful HTTP 接口访问所有数据库功能，包括 CRUD、查询和 JavaScript 处理。 DocumentDB 包含现有格式、语言和标准，并同时基于这些内容提供高价值的数据库功能。
- **自动索引：**默认情况下，DocumentDB 将自动为数据库中的所有文档编制，无需任何架构或创建二级索引。 不想索引所有内容？ 别担心，还可以 [退出 JSON 文件中的路径](/documentation/articles/documentdb-indexing-policies/) 。
- **兼容 MongoDB 应用：**使用适用于 MongoDB 的 DocumentDB: API 时，你可以将 DocumentDB 数据库用作针对 MongoDB 编写的应用的数据存储。 这意味着，通过使用 MongoDB 数据库的现有驱动程序，为 MongoDB 编写的应用程序现在可与 DocumentDB 进行通信，并可使用 DocumentDB 数据库而不是 MongoDB 数据库。 在许多情况下，只需更改连接字符串便可从使用 DocumentDB 切换到使用 MongoDB。 在[什么是适用于 MongoDB 的 DocumentDB: API？](/documentation/articles/documentdb-protocol-mongodb/)中了解详细信息

## <a name="data-management"></a>DocumentDB 如何管理数据？
Azure DocumentDB 通过定义完好的数据库资源管理 JSON 数据。 这些资源经过复制具有高可用性，并且使用其逻辑 URI 进行唯一寻址。 DocumentDB 为所有资源提供简单的基于 HTTP 的 RESTful 编程模型。 

DocumentDB 数据库帐户是授予你 Azure DocumentDB 访问权限的唯一命名空间。 在创建数据库帐户之前，你必须具有 Azure 订阅，以便为你提供访问各种 Azure 服务的权限。 

DocumentDB 中的所有资源都以 JSON 文档的形式建模和存储。 将资源作为项（包含元数据的 JSON 文档）进行管理，也可以作为源（项的集合）进行管理。 项集包含在它们各自的源中。

下图显示了 DocumentDB 资源之间的关系：

![DocumentDB（一种 NoSQL JSON 数据库）中资源之间的层级关系][1] 

一个数据库帐户可以包含一组数据库，每个数据库都包含多个集合，每个集合又包含存储过程、触发器、UDF、文档及相关附件。 数据库也有关联的用户，每个用户都有一组权限来访问其他各种集合、存储过程、触发器、UDF、文档或附件。 尽管数据库、用户、权限和集合是系统定义的具有已知架构的资源，文档、存储过程、触发器、UDF 和附件也包含任意的用户定义的 JSON 内容。  

## <a name="develop"></a> 如何使用 DocumentDB 开发应用？
Azure DocumentDB 通过 REST API 公开资源，此 API 可以使用能够发出 HTTP/HTTPS 请求的任何语言调用。 另外，DocumentDB 还为多种主流语言提供了编程库，且兼容 MongoDB API。 客户端库通过处理一些细节，例如地址缓存、异常管理、自动重试等，简化了使用 Azure DocumentDB 的许多方面。 当前这些库可用于以下语言和平台：  

| 下载 | 文档 |
| --- | --- |
| [.NET SDK](http://go.microsoft.com/fwlink/?LinkID=402989) |[.NET 库](https://msdn.microsoft.com/zh-cn/library/azure/dn948556.aspx) |
| [Node.js SDK](http://go.microsoft.com/fwlink/?LinkID=402990) |[Node.js 库](http://azure.github.io/azure-documentdb-node/) |
| [Java SDK](http://go.microsoft.com/fwlink/?LinkID=402380) |[Java 库](http://azure.github.io/azure-documentdb-java/) |
| [JavaScript SDK](http://go.microsoft.com/fwlink/?LinkID=402991) |[JavaScript 库](http://azure.github.io/azure-documentdb-js/) |
| 不适用 |[服务器端 JavaScript SDK](http://azure.github.io/azure-documentdb-js-server/) |
| [Python SDK](https://pypi.python.org/pypi/pydocumentdb) |[Python 库](http://azure.github.io/azure-documentdb-python/) |
| 不适用 | [适用于 MongoDB 的 API](/documentation/articles/documentdb-protocol-mongodb/)

使用  [Azure DocumentDB Emulator](/documentation/articles/documentdb-nosql-local-emulator/) 可在本地开发和测试应用程序，无需创建 Azure 订阅且不会产生任何费用。 如果对应用程序在 DocumentDB 模拟器中的工作情况感到满意，则可以切换到在云中使用 Azure DocumentDB 帐户。

除了基本的创建、读取、更新和删除操作，DocumentDB 还提供丰富的 SQL 查询接口用于检索 JSON 文档和针对 JavaScript 应用程序逻辑的事务执行的服务器端支持。 可通过所有平台库获取查询和脚本执行接口以及 REST API。 

### <a name="sql-query"></a>SQL 查询
Azure DocumentDB 支持使用 SQL 语言（来源于 JavaScript 类型系统）和支持关系、层级和空间查询的表达式来查询文档。 DocumentDB 查询语言是一种用于查询 JSON 文档的简单而强大的接口。 该语言支持部分 ANSI SQL 语法，并深度集成了 JavaScript 对象、数组、对象构造和函数调用。 DocumentDB 提供的查询模型没有任何显式架构或来自开发人员的索引提示。

可以在 DocumentDB 中注册用户定义函数 (UDF)，并将其作为 SQL 查询的一部分进行引用，从而将语法扩展为支持自定义的应用程序逻辑。 这些 UDF 编写为 JavaScript 程序，并在数据库中执行。 

对于 .NET 开发人员，DocumentDB 还在 [.NET SDK](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.linq.aspx)中提供了 LINQ 查询提供程序。 

### <a name="transactions-and-javascript-execution"></a>事务和 JavaScript 执行
DocumentDB 允许将应用程序逻辑编写为完全使用 JavaScript 编写的命名程序。 这些程序是为集合注册的，可以对指定集合内的文档发布数据库操作。 可针对执行将 JavaScript 注册为触发器、存储过程或用户定义函数。 触发器和存储过程可以创建、读取、更新和删除文档，而用户定义函数作为查询执行逻辑的一部分执行，并且没有集合的写访问权限。

DocumentDB 中的 JavaScript 执行是在关系型数据库系统所支持的概念的基础之上建立的，只是现代性的将 Transact-SQL 换成了 JavaScript。 所有 JavaScript 逻辑都在使用快照隔离的环境 ACID 事务内执行。 在其执行过程中，如果 JavaScript 引发异常，则整个事务将被中止。

## <a name="next-steps"></a>后续步骤
已有 Azure 帐户？ 那么可以在 [Azure 门户预览](https://portal.azure.cn/#gallery/Microsoft.DocumentDB)中通过[创建 DocumentDB 数据库帐户](/documentation/articles/documentdb-create-account/)开始使用 DocumentDB。

还没有 Azure 帐户？ 你可以：

- 注册 [Azure 试用版](/pricing/1rmb-trial/)。 
- 下载 [Azure DocumentDB Emulator](/documentation/articles/documentdb-nosql-local-emulator/)，以便在本地开发应用程序。

[1]: ./media/documentdb-introduction/json-database-resources1.png

<!---HONumber=Mooncake_Quality_Review_1230_2016-->

<!-- Update_Description: wording update -->