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
	ms.date="07/01/2016" 
	wacn.date="08/02/2016"/>

# DocumentDB 简介：一种 NoSQL JSON 数据库

Azure Document DB 提供完全托管的 NoSQL 数据库服务，可加速并预测性能、实现高度可用、自动缩放、简易开发。凭借其灵活的数据模型，一贯的低延迟以及丰富的查询功能，Azure Document DB非常适合需无缝缩放的应用程序，诸如web, mobile, gaming和IoT等。

想快速了解使用，请遵循以下三个步骤：

1.	观看[什么是 DocumentDB？](https://azure.microsoft.com/documentation/videos/what-is-azure-documentdb/)视频—时长两分钟，介绍使用 DocumentDB 的优势。
2.	观看[在 Azure 上创建 DocumentDB](/documentation/articles/documentdb-get-started/) 视频—重点介绍如何通过 Azure 门户使用 DocumentDB。
3.	访问[数据游乐园](http://www.documentdb.com/sql/demo)—学习不同的查询语法，了解 DocumentDB 中的丰富查询功能。

	打开“沙盒”选项卡—运行自定义 的SQL 查询，试用 DocumentDB 。

下面对其进行深入探讨，探讨问题如下：

* [什么是 DocumentDB？它对现代应用程序的价值何在？](#what-is-azure-documentdb)
* [DocumentDB 如何管理数据？如何访问？](#data-management)
* [如何使用 DocumentDB 开发应用程序？](#develop)
* [构建DocumentDB 应用程序的后续步骤？](#next-steps)

## <a name="what-is-azure-documentdb"></a> 什么是 Azure DocumentDB？

现代应用程序可快速生成、使用并响应海量数据，发展速度非常之快，底层数据架构同样如此。为适应这种形势，开发人员倾向于选择无架构的 NoSQL 文档数据库，用这一简单、快速和可扩展的解决方案来存储和处理数据；该数据库同时保留了快速循环访问应用程序数据模型和非结构化数据源的功能。但是，许多无架构数据库无法进行复杂的查询和事务处理，因此很难管理高级数据。微软公司的DocumentDB基于此背景开发，用来管理当下应用程序的数据，满足用户需求。

DocumentDB 是针对当今的移动、Web、游戏和 IoT 应用程序而设计的真正无架构的 NoSQL 数据库服务。它读写快速，架构灵活，可按需缩放数据库。无需为已建立索引的 JSON 文档假设或要求任何架构。默认情况下，它会自动对数据库中的所有文档创建索引，无需任何架构，也无需创建二级索引。DocumentDB 可使用 SQL 语言进行复杂的即席查询，支持定义完好的一致性级别，并且可使用 存储过程、触发器和 用户定义函数（UDF）中的熟悉编程模型，提供集成了 JavaScript 语言的多文档事务处理。

作为一种 JSON 数据库，DocumentDB 本机支持启用了应用程序架构简单迭代的 JSON 文档，并且支持需要键/值、文档或表格数据模型的应用程序。DocumentDB 采用广泛普及的 JSON 和 JavaScript 语言，消除了应用程序定义的对象和数据库架构之间的不匹配。它对 JavaScript 的深度集成还允许开发人员在数据库事务中的数据库引擎内高效而直接地执行应用程序逻辑。

Azure DocumentDB 主要功能和优势如下：

- **可灵活增减的吞吐量和存储：**DocumentDB JSON 数据库可轻松增大或减小，满足应用程序需求。存储在固态硬盘 (SSD) 上的数据可实现可预测的低延迟。DocumentDB支持存储JSON数据的容器，该容器称为集锦，其存储量几乎无限大，且可设置吞吐量。应用程序增大时，DocumentDB可随之灵活扩大，且性能可预测，两者无缝对接。

- **使用熟悉的 SQL 语法进行即席查询：**在 DocumentDB 中存储异类 JSON 文档，并通过熟悉的 SQL 语法查询这些文档。DocumentDB 使用高并发、无锁、日志结构化索引技术为所有文档内容自动创建索引。这样可以实现各种实时查询，而无需指定架构提示、二级索引或视图。要了解更多信息，请参阅[查询 DocumentDB](/documentation/articles/documentdb-sql-query/)。

- **在数据库中执行 JavaScript：**使用标准 JavaScript 将应用程序逻辑表示为存储过程、触发器和用户定义函数 (UDF)。这样，应用程序逻辑可基于数据进行运作，无需担心应用程序和数据库架构之间的不匹配。DocumentDB可在数据库引擎中直接执行JavaScript应用程序逻辑的完整事务。深度集成的JavaScript可在 JavaScript 程序中将 INSERT、REPLACE、DELETE 和 SELECT 操作作为独立的事务来执行。要了解更多信息，请参阅 [DocumentDB 服务器端编程](/documentation/articles/documentdb-programming/)。

- **可调优的一致性级别：**从 4 个定义完好的一致性级别中做选择，实现一致性和性能之间的最佳平衡。对于查询和读取操作，DocumentDB 提供了四种不同的一致性级别：强级、有限过时级、会话级和最终级。这些细化的定义完好的一致性级别，有助于在一致性、可用性和延迟之间实现合理的平衡。要了解更多信息，请参阅[使用一致性级别最大化 DocumentDB 中的可用性和性能](/documentation/articles/documentdb-consistency-levels/)。

- **完全托管：**无需管理数据库和计算机资源。作为一种完全托管的 Azure 服务，用户无需管理虚拟机、部署并配置软件，也无需管理数据库和资源的增减，或处理复杂的数据层升级。每个数据库都自动备份，以防受到区域故障的影响。可轻松添加 DocumentDB 帐户，按需设置容量，无需耗时操作、管理数据库，只需关注应用程序。

- **源于设计的开放性：**通过使用现有的技能和工具快速入门。针对 DocumentDB 的编程非常简单易学，无需使用新的工具或遵循 JSON 或 JavaScript 的自定义扩展。你可以通过简单的 RESTful HTTP 接口访问所有数据库功能，包括 CRUD、查询和 JavaScript 处理。DocumentDB 包含现有格式、语言和标准，并基于这些内容提供高价值的数据库功能。

可使用 DocumentDB 存储灵活的数据集，这些数据集需要查询检索和事务处理。应用场景包括交互式 Web、移动和游戏应用程序的用户数据，以及 IoT 设备生成的 JSON 数据的存储、检索和处理。数据库可以存储任意数量的 JSON 文档，因为 DocumentDB 非常适合在 Internet 上大规模运行的应用程序。

## <a name="data-management"></a> Azure DocumentDB 资源

Azure DocumentDB 通过定义完好的数据库资源管理数据。这些资源经过复制具有高可用性，并且使用其逻辑 URI 进行唯一寻址。DocumentDB 为所有资源提供简单的 HTTP ，该HTTP基于RESTful 编程模型。

DocumentDB 数据库帐户是访问Azure DocumentDB 的唯一命名空间。在创建数据库帐户之前，必须订阅 Azure ，才能访问 Azure 各种服务。

DocumentDB 中的所有资源都以 JSON 文档的形式建模和存储。将资源作为项（包含元数据的 JSON 文档）进行管理，也可以作为源（项的集合）进行管理。项集包含在它们各自的源中。

下图显示了 DocumentDB 资源之间的关系：

![DocumentDB（一种 NoSQL JSON 数据库）中资源之间的层级关系][1]

数据库帐户包含一组数据库，每个数据库包含多个集合，每个集合又包含存储过程、触发器、UDF、文档及相关附件。数据库也有关联的用户，每个用户都有一组权限访问其他各种集合、存储过程、触发器、UDF、文档或附件。尽管数据库、用户、权限和集合是由系统定义的具有已知架构的资源，文档、存储过程、触发器、UDF 和附件也包含任意的用户定义的 JSON 内容。

## <a name="develop"></a> 使用 Azure DocumentDB 进行开发

Azure DocumentDB 可通过 REST API 公开资源，此 API 能够被发出 HTTP/HTTPS 请求的任何语言调用。另外，DocumentDB 还为多种主流语言提供编程库。这些库通过处理诸如地址缓存、异常管理、自动重试等细节，简化了 Azure DocumentDB 工作的许多方面。当前这些库可用于以下语言和平台：

下载 | 文档
--- | ---
[.NET SDK](http://go.microsoft.com/fwlink/?LinkID=402989) | [.NET 库](https://msdn.microsoft.com/zh-cn/library/azure/dn948556.aspx)
[Node.js SDK](http://go.microsoft.com/fwlink/?LinkID=402990) | [Node.js 库](http://azure.github.io/azure-documentdb-node/)
[Java SDK](http://go.microsoft.com/fwlink/?LinkID=402380) | [Java 库](http://azure.github.io/azure-documentdb-java/)
[JavaScript SDK](http://go.microsoft.com/fwlink/?LinkID=402991) | [JavaScript 库](http://azure.github.io/azure-documentdb-js/)
不适用 | [服务器端 JavaScript SDK](http://azure.github.io/azure-documentdb-js-server/)
[Python SDK](https://pypi.python.org/pypi/pydocumentdb) | [Python 库](http://azure.github.io/azure-documentdb-python/)

除了基本的创建、读取、更新和删除操作，DocumentDB 还提供丰富的 SQL 查询接口用于检索 JSON 文档； 还为JavaScript 应用程序逻辑的事务执行提供服务器端提供支持。查询和脚本执行接口在所有平台库以及 REST API中均可用。

### SQL 查询

Azure DocumentDB 支持使用 SQL 语言（来源于 JavaScript 类型系统）和支持关系、层级和空间查询的表达式来查询文档。DocumentDB 查询语言是个简单而强大的接口，用于查询 JSON 文档。该语言支持部分 ANSI SQL 语法，深度集成 JavaScript 对象、数组、对象构造和函数调用。DocumentDB 提供的查询模型没有显式架构，也没有来自开发人员的索引提示。

DocumentDB 中可注册用户定义函数 (UDF)，并将其作为 SQL 查询的一部分进行引用，从而将语法扩展为支持自定义的应用程序逻辑。这些 UDF 编写为 JavaScript 程序，并在数据库中执行。

对于 .NET 开发人员，DocumentDB 还提供作为 [.NET SDK](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.linq.aspx) 一部分的 LINQ 查询提供程序。

### 事务和 JavaScript 执行

DocumentDB 可将应用程序逻辑编写为完全使用 JavaScript 编写的命名程序。这些程序为集合而注册，可以对指定集合内的文档发布数据库操作。 JavaScript 同样可注册为触发器、存储过程或用户定义函数。触发器和存储过程可以创建、读取、更新和删除文档；用户定义函数作为查询执行逻辑的一部分执行，没有集合的写访问权限。

DocumentDB 中的 JavaScript 执行是在关系型数据库系统所支持的概念的基础之上建立的，只是现在将 Transact-SQL 换成了 JavaScript。所有 JavaScript 逻辑都在使用快照隔离的 ACID 环境事务内执行。在其执行过程中，如果 JavaScript 引发异常，则整个事务中止。

## <a name="next-steps"></a> 后续步骤

已有 Azure 帐户的用户可以在 [Azure 门户预览](https://portal.azure.cn/#gallery/Microsoft.DocumentDB)中通过[创建 DocumentDB 数据库帐户](/documentation/articles/documentdb-create-account/)使用 DocumentDB。

如果没有 Azure 帐户，则可以：

- 注册 [Azure 试用版](/pricing/1rmb-trial/)，可提供 30 天试用期和 1,500 元人民币用于试用 Azure 所有服务。

[1]: ./media/documentdb-introduction/json-database-resources1.png
 
