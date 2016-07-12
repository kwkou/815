<properties 
	pageTitle="DocumentDB 简介：一种 JSON 数据库 | Azure" 
	description="了解 Azure DocumentDB，一种 NoSQL JSON 数据库。此文档数据库是针对大数据、灵活的可扩展性和高可用性构建的。" 
	keywords="json 数据库，文档数据库"
	services="documentdb" 
	authors="mimig1" 
	manager="jhubbard" 
	editor="monicar" 
	documentationCenter=""/>

<tags 
	ms.service="documentdb" 
	ms.date="03/30/2016" 
	wacn.date="07/04/2016"/>

# DocumentDB 简介：一种 NoSQL JSON 数据库

Azure DocumentDB 是一个完全托管的 NoSQL 数据库服务，其构建目的是为了实现快速且可预测的性能、高可用性、自动扩展和易于开发。该数据库的灵活的数据模型、一贯的低延迟和丰富的查询功能使其非常适合用于 Web、移动、游戏和 IoT 应用程序，以及其他许多需要无缝扩展的应用程序。

要快速了解此 JSON 数据库和在实际操作中使用该数据库，请遵循以下三个步骤：

1. 观看两分钟的[什么是 DocumentDB？](https://azure.microsoft.com/documentation/videos/what-is-azure-documentdb/)视频，该视频介绍了使用 DocumentDB 具有哪些优势。
2. 观看三分钟的[在 Azure 上创建 DocumentDB](https://azure.microsoft.com/documentation/videos/create-documentdb-on-azure/) 视频，该视频重点介绍了如何通过 Azure 门户预览开始使用 DocumentDB。
3. 请访问[数据游乐园](http://www.documentdb.com/sql/demo)，你可以在其中学习不同的查询语法，以便了解 DocumentDB 中提供的丰富的查询功能。接着前往“沙盒”选项卡并运行你自己的自定义 SQL 查询，并对 DocumentDB 进行试用。

然后返回到本文中，我们将更加深入的探讨该数据库，你将了解以下问题的答案：

-	[什么是 DocumentDB 以及它对现代应用程序有哪些价值？](#what-is-azure-documentdb)
-	[DocumentDB 是如何管理我的数据的，以及我如何访问该数据库？](#data-management)
-	[如何使用 DocumentDB 开发应用程序？](#develop)
-	[构建一个 DocumentDB 应用程序的后续步骤有哪些？](#next-steps)  

## <a name="what-is-azure-documentdb"></a>什么是 Azure DocumentDB？  

现代应用程序生成、使用并快速响应非常大量的数据。这些应用程序发展非常迅速，相应的底层数据架构也是如此。为适应这种形势，开发人员日益倾向于选择无架构的 NoSQL 文档数据库作为存储和处理数据的简单、快速和可扩展的解决方案，并且该数据库同时保留了快速循环访问应用程序数据模型和非结构化数据源的功能。但是，许多无架构数据库无法进行复杂的查询和事务处理，这使高级数据管理变得困难。这正是 DocumentDB 的用武之地。在管理当今应用程序的数据时，Microsoft 开发了 DocumentDB 来满足这些需求。

DocumentDB 是针对当今的移动、Web、游戏和 IoT 应用程序设计的一个真正无架构的 NoSQL 数据库服务。DocumentDB 提供了持续快速的读取和写入功能、架构灵活性，以及能够根据需要轻松扩大和减小数据库规模的功能。无需为已建立索引的 JSON 文档假设或要求任何架构。默认情况下，将自动对数据库中的所有文档创建索引，并且无需期望或要求任何架构或创建二级索引。DocumentDB 可使用 SQL 语言进行复杂的即席查询，支持定义完好的一致性级别，并且提供使用 Stored Procedure, Trigger 和 UDF 的熟悉编程模型的集成了 JavaScript 语言的多文档事务处理。

作为一种 JSON 数据库，DocumentDB 原生支持启用了应用程序架构简单迭代的 JSON 文档，并且支持需要键/值、文档或表格数据模型的应用程序。DocumentDB 采用广泛普及的 JSON 和 JavaScript 语言，消除了应用程序定义的对象和数据库架构之间的不匹配。对 JavaScript 的深度集成还允许开发人员在数据库事务中，在数据库引擎内高效而直接地执行应用程序逻辑。

Azure DocumentDB 具有以下主要功能和优势：

-	**可灵活增减的吞吐量和存储：**轻松增大或减小 DocumentDB JSON 数据库规模来满足你的应用程序的需求。你的数据存储在固态硬盘 (SSD) 上，以实现可预测的低延迟。DocumentDB 支持使用容器来存储称为集合的 JSON 数据，这些数据可以扩展到几乎无限的存储大小和设置的吞吐量。随着应用程序规模的增长，你可以灵活无缝地扩展具有可预测的性能的 DocumentDB。 

-	**使用熟悉的 SQL 语法进行即席查询：**在 DocumentDB 中存储异类 JSON 文档，并通过熟悉的 SQL 语法查询这些文档。DocumentDB 使用高并发、无锁、日志结构化索引技术为所有文档内容自动创建索引。这样可以实现各种实时查询，而无需指定架构提示、二级索引或视图。要了解更多信息，请参阅[查询 DocumentDB](/documentation/articles/documentdb-sql-query/)。

-	**在数据库中执行 JavaScript：**使用标准 JavaScript 将应用程序逻辑表示为存储过程、触发器和用户定义函数 (UDF)。这样，你的应用程序逻辑可基于数据进行运作，而无需担心应用程序和数据库架构之间的不匹配。DocumentDB 支持在数据库引擎内部直接进行 JavaScript 应用程序逻辑的完全事务执行。对 JavaScript 的深度集成支持在一个 JavaScript 程序中将 INSERT、REPLACE、DELETE 和 SELECT 操作作为独立的事务来执行。要了解更多信息，请参阅 [DocumentDB 服务器端编程](/documentation/articles/documentdb-programming/)。

-	**可调优的一致性级别：**从 4 个定义完好的一致性级别中选择，以实现一致性和性能之间的最佳平衡。对于查询和读取操作，DocumentDB 提供了四种不同的一致性级别：强、有限过时、会话和最终。通过这些细化的定义完好的一致性级别，你可以在一致性、可用性和延迟之间实现合理的平衡。要了解更多信息，请参阅[使用一致性级别最大化 DocumentDB 中的可用性和性能](/documentation/articles/documentdb-consistency-levels/)。

-	**完全托管：**无需管理数据库和计算机资源。作为一种完全托管的 Azure 服务，你无需管理虚拟机、部署并配置软件、管理数据库和资源的增减，或处理复杂的数据层升级。每个数据库都将自动备份，以防受到区域故障的影响。你可以轻松添加 DocumentDB 帐户并按照你的需求设置容量，从而使你专注于你的应用程序而不是操作和管理你的数据库。

-	**源于设计的开放性：**通过使用现有的技能和工具快速入门。针对 DocumentDB 的编程非常简单易学，你无需使用新的工具或遵循 JSON 或 JavaScript 的自定义扩展。你可以通过简单的 RESTful HTTP 接口访问所有数据库功能，包括 CRUD、查询和 JavaScript 处理。DocumentDB 包含现有格式、语言和标准，并同时基于这些内容提供高价值的数据库功能。

可以使用 DocumentDB 来存储灵活的数据集，这些数据集需要查询检索和事务处理。应用场景包括交互式 Web、移动和游戏应用程序的用户数据，以及 IoT 设备生成的 JSON 数据的存储、检索和处理。一个数据库可以存储任意数量的 JSON 文档，因为 DocumentDB 非常适合在 Internet 上大规模运行的应用程序。

##<a name="data-management"></a>Azure DocumentDB 资源
Azure DocumentDB 通过定义完好的数据库资源管理数据。这些资源经过复制具有高可用性，并且使用其逻辑 URI 进行唯一寻址。DocumentDB 为所有资源提供简单的基于 HTTP 的 RESTful 编程模型。

DocumentDB 数据库帐户是授予你 Azure DocumentDB 访问权限的唯一命名空间。在创建数据库帐户之前，你必须具有 Azure 订阅，以便为你提供访问各种 Azure 服务的权限。

DocumentDB 中的所有资源都以 JSON 文档的形式建模和存储。将资源作为项（包含元数据的 JSON 文档）进行管理，也可以作为源（项的集合）进行管理。项集包含在它们各自的源中。

下图显示了 DocumentDB 资源之间的关系：

![DocumentDB（一种 NoSQL JSON 数据库）中资源之间的层级关系][1]

一个数据库帐户可以包含一组数据库，每个数据库都包含多个集合，每个集合又包含存储过程、触发器、UDF、文档及相关附件。数据库也有关联的用户，每个用户都有一组权限来访问其他各种集合、存储过程、触发器、UDF、文档或附件。尽管数据库、用户、权限和集合是系统定义的具有已知架构的资源，文档、存储过程、触发器、UDF 和附件也包含任意的用户定义的 JSON 内容。

##<a name="develop"></a>使用 Azure DocumentDB 进行开发
Azure DocumentDB 通过 REST API 公开资源，此 API 可以使用能够发出 HTTP/HTTPS 请求的任何语言调用。另外，DocumentDB 还为多种主流语言提供了编程库。这些库通过处理一些细节，例如地址缓存、异常管理、自动重试等，简化了使用 Azure DocumentDB 的许多方面。当前这些库可用于以下语言和平台：

下载 | 文档
--- | ---
[.NET SDK](http://go.microsoft.com/fwlink/?LinkID=402989) | [.NET 库](https://msdn.microsoft.com/library/azure/dn948556.aspx)
[Node.js SDK](http://go.microsoft.com/fwlink/?LinkID=402990) | [Node.js 库](http://azure.github.io/azure-documentdb-node/)
[Java SDK](http://go.microsoft.com/fwlink/?LinkID=402380) | [Java 库](http://azure.github.io/azure-documentdb-java/)
[JavaScript SDK](http://go.microsoft.com/fwlink/?LinkID=402991) | [JavaScript 库](http://azure.github.io/azure-documentdb-js/)
不适用 | [服务器端 JavaScript SDK](http://azure.github.io/azure-documentdb-js-server/)
[Python SDK](https://pypi.python.org/pypi/pydocumentdb) | [Python 库](http://azure.github.io/azure-documentdb-python/)

除了基本的创建、读取、更新和删除操作，DocumentDB 还提供丰富的 SQL 查询接口用于检索 JSON 文档和针对 JavaScript 应用程序逻辑的事务执行的服务器端支持。可通过所有平台库获取查询和脚本执行接口以及 REST API。

### SQL 查询
Azure DocumentDB 支持使用 SQL 语言（来源于 JavaScript 类型系统）和支持关系、层级和空间查询的表达式来查询文档。DocumentDB 查询语言是一种用于查询 JSON 文档的简单而强大的接口。该语言支持部分 ANSI SQL 语法，并深度集成了 JavaScript 对象、数组、对象构造和函数调用。DocumentDB 提供的查询模型没有任何显式架构或来自开发人员的索引提示。

可以在 DocumentDB 中注册用户定义函数 (UDF)，并将其作为 SQL 查询的一部分进行引用，从而将语法扩展为支持自定义的应用程序逻辑。这些 UDF 编写为 JavaScript 程序，并在数据库中执行。

对于 .NET 开发人员，DocumentDB 还提供作为 [.NET SDK](https://msdn.microsoft.com/library/azure/microsoft.azure.documents.linq.aspx) 一部分的 LINQ 查询提供程序。

### 事务和 JavaScript 执行
DocumentDB 允许将应用程序逻辑编写为完全使用 JavaScript 编写的命名程序。这些程序是为集合注册的，可以对指定集合内的文档发布数据库操作。可针对执行将 JavaScript 注册为触发器、存储过程或用户定义函数。触发器和存储过程可以创建、读取、更新和删除文档，而用户定义函数作为查询执行逻辑的一部分执行，并且没有集合的写访问权限。

DocumentDB 中的 JavaScript 执行是在关系型数据库系统所支持的概念的基础之上建立的，只是现代性的将 Transact-SQL 换成了 JavaScript。所有 JavaScript 逻辑都在使用快照隔离的环境 ACID 事务内执行。在其执行过程中，如果 JavaScript 引发异常，则整个事务将被中止。

## <a name="next-steps"></a>后续步骤
如果你已经有 Azure 帐户，则可以在 [Azure 门户预览](https://portal.azure.cn/#gallery/Microsoft.DocumentDB)中通过[创建 DocumentDB 数据库帐户](/documentation/articles/documentdb-create-account/)开始使用 DocumentDB。

如果你没有 Azure 帐户，则可以：

- 注册 [Azure 试用版](/pricing/free-trial/)，将为你提供 30 天试用期和 200 美元用于试用所有 Azure 服务。 



[1]: ./media/documentdb-introduction/json-database-resources1.png
 

<!---HONumber=Mooncake_0523_2016-->