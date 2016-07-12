<properties 
	pageTitle="DocumentDB 数据库问题 — 常见问题 | Azure" 
	description="获取有关 Azure DocumentDB（用于 JSON 的 NoSQL 文档数据库服务）常见问题的解答。解答有关容量、性能级别和缩放的数据库问题。" 
	keywords="数据库问题, 常见问题, documentdb, azure, Microsoft azure"
	services="documentdb" 
	authors="mimig1" 
	manager="jhubbard" 
	editor="monicar" 
	documentationCenter=""/>

<tags 
	ms.service="documentdb" 
	ms.date="03/30/2016" 
	wacn.date="06/28/2016"/>


#有关 DocumentDB 的常见问题

## 有关 Azure DocumentDB 基本原理的数据库问题

### 什么是 Azure DocumentDB？ 
Azure DocumentDB 是高度可缩放的 NoSQL 文档数据库即服务，可针对无架构数据提供丰富的查询，帮助提供可靠的可配置性能，并支持快速开发，这一切都通过一个托管平台来实现，而该平台有 Azure 强大的功能与影响力作为后盾。如果 Web、移动、游戏和 IoT 应用程序的关键要求是可预测的吞吐量、低延迟和无架构数据模型，那么，DocumentDB 无疑是最合适的解决方案。DocumentDB 通过本机 JSON 数据模型提供架构灵活性和丰富索引，并且对集成的 JavaScript 提供多文档事务性支持。
  
有关部署和使用此服务的更多数据库问题、解答及说明，请参阅 [DocumentDB 文档页面](/home/features/documentdb/)。

### DocumentDB 是何种数据库？
DocumentDB 是面向 NoSQL 文档的数据库，以 JSON 格式存储数据。DocumentDB 支持嵌套的独立数据结构，可通过丰富的 DocumentDB [SQL 查询语法](/documentation/articles/documentdb-sql-query/)来查询。DocumentDB 通过[存储过程、触发器和用户定义函数](/documentation/articles/documentdb-programming/)，提供服务器端 JavaScript 的高性能事务处理功能。该数据库还支持可由开发人员调整的一致性级别以及相关联的[性能级别](/documentation/articles/documentdb-performance-levels/)。
 
### DocumentDB 数据库是否像关系数据库 (RDBMS) 一样有许多表？
没有，DocumentDB 将数据存储在 JSON 文档集合中。有关 DocumentDB 资源的信息，请参阅 [DocumentDB 资源模型和概念](/documentation/articles/documentdb-resources/)。

### DocumentDB 数据库是否支持无架构数据？
是，DocumentDB 允许应用程序存储任意的 JSON 文档，而不需要架构定义或提示。通过 DocumentDB SQL 查询界面便可立即查询数据。

### DocumentDB 是否支持 ACID 事务？
是，DocumentDB 支持以 JavaScript 存储过程和触发器形式表示的跨文档事务。事务以每个集合内的单个分区为范围，且以 ACID 语义执行，也就是全有或全无，与其他并发执行的代码和用户请求隔离。如果在服务器端执行 JavaScript 应用程序代码的过程中引发了异常，则会回滚整个事务。

### DocumentDB 的典型用例有哪些？  
对于侧重于以下要求的新 Web、移动、游戏和 IoT 应用程序而言，DocumentDB 是一个不错的选择：自动扩展、可预测的性能、毫秒响应时间的快速排序，以及查询无架构数据的能力。DocumentDB 有助于快速开发，且支持应用程序数据模型的连续迭代。用于管理用户生成的内容和数据的应用程序就是 [DocumentDB 的常见用例](/documentation/articles/documentdb-use-cases/)。

### DocumentDB 如何提供可预测的性能？
请求单位 (RU) 是 DocumentDB 中吞吐量的衡量单位。1 个 RU 相当于读取 1KB 文档的吞吐量。在 DocumentDB 中进行的每个操作（包括读、写、SQL 查询和执行存储过程）都将具有一个确定的请求单位值，该值基于完成该操作所需的吞吐量。你无需考虑 CPU、IO 和内存，以及它们会怎样影响你的应用程序吞吐量，而是从单个请求单位度量值的角度进行考虑。

每个 DocumentDB 集合可以配置以每秒请求单位表示的预配吞吐量。对于任何规模的应用程序，你可以将单个请求设为基准以测量其请求单位值，并预配集合以处理所有请求的请求单位总和。你也可以随着应用程序的发展需求，相应增加或减少集合的吞吐量。如需请求单位的详细信息以及帮助确定你的集合需求，请阅读[管理性能和容量](/documentation/articles/documentdb-manage/)。

### DocumentDB 是否符合 HIPAA 标准？
是，DocumentDB 符合 HIPAA 标准。HIPAA 针对可识别个人身份的健康信息的使用、泄露与保护制定了要求。有关详细信息，请参阅 [Microsoft 信任中心](https://www.microsoft.com/zh-cn/TrustCenter/Compliance/HIPAA)。

### DocumentDB 的存储限制有哪些？ 
对于集合可以存储在 DocumentDB 中的数据总量，理论上没有任何限制。如果你想在单个集合中存储超过 250 GB 的数据，请[与支持部门联系](/documentation/articles/documentdb-increase-limits/)。

### DocumentDB 的吞吐量限制有哪些？ 
如果你的工作负荷可以大致平均分配给足够大量的分区键，则在 DocumentDB 中集合可以支持的吞吐量总量理论上没有限制。如果你希望在单个集合或帐户中每秒超过 250,000 个请求单位，请[与支持部门联系](/documentation/articles/documentdb-increase-limits/)。

### Azure DocumentDB 的费用是多少？
请参考 [DocumentDB 定价详细信息](/pricing/details/documentdb/)页面了解详细信息。DocumentDB 使用量费用取决于正在使用的集合数目、集合的在线小时数，以及每个集合的已使用存储和已预配吞吐量。

### 有免费的帐户吗？
如果你不熟悉 Azure，可以注册 [Azure 试用帐户](/pricing/free-trial/)，这样可以得到 30 天试用期和 200 美元的 Azure 信用额度，让你试用所有 Azure 服务。

### 如何获取 DocumentDB 的其他帮助？
如果你需要任何帮助，请在 [Stack Overflow](http://stackoverflow.com/questions/tagged/azure-documentdb)或 [Azure DocumentDB MSDN 开发人员论坛](https://social.msdn.microsoft.com/forums/azure/home?forum=AzureDocumentDB)联系我们，或者安排[与 DocumentDB 工程团队进行 1 对 1 交谈](http://www.askdocdb.com/)。

## 设置 Azure DocumentDB

### 我如何注册 Azure DocumentDB？
[Azure 门户预览][azure-portal]中已提供 Azure DocumentDB。首先，你必须注册 Azure 订阅。注册 Azure 订阅之后，你可以将 DocumentDB 帐户添加到 Azure 订阅。有关添加 DocumentDB 帐户的说明，请参阅[创建 DocumentDB 数据库帐户](/documentation/articles/documentdb-create-account/)。

### 什么是主密钥？
主密钥是用于访问帐户的所有资源的安全令牌。拥有此密钥的人对数据库帐户中的所有资源具有读取和写入访问权。分发主密钥时要格外谨慎。[Azure 门户预览][azure-portal]的**密钥**边栏选项卡中提供主要主密钥和辅助主密钥。有关密钥的详细信息，请参阅[查看、复制和重新生成访问密钥](/documentation/articles/documentdb-manage-account/#keys)。

### 我如何创建数据库？
你可以如[创建 DocumentDB 数据库](/documentation/articles/documentdb-create-database/)中所述使用 [Azure 门户预览]()、利用某个 [DocumentDB SDK](/documentation/articles/documentdb-dotnet-samples/) 或通过 [REST API](https://msdn.microsoft.com/library/azure/dn781481.aspx) 来创建数据库。

### 什么是集合？
集合是 JSON 文档和相关联的 JavaScript 应用程序逻辑的容器。集合是一个计费实体，其中[成本](/documentation/articles/documentdb-performance-levels/)由与集合关联的性能级别确定。集合可以跨一个或多个分区/服务器，并且能扩展以处理几乎无限制增长的存储或吞吐量。

集合也是 DocumentDB 的计费实体。每个集合根据预配的吞吐量和使用的存储空间按小时计费。有关详细信息，请参阅 [DocumentDB 定价](/pricing/details/documentdb/)。

### 我如何设置用户和权限？
你可以使用某个 [DocumentDB SDK](/documentation/articles/documentdb-dotnet-samples/) 或通过 [REST API](https://msdn.microsoft.com/library/azure/dn781481.aspx) 来创建用户和权限。

## 针对 Azure DocumentDB 进行开发的相关数据库问题

### 如何开始针对 DocumentDB 进行开发？
[SDK](/documentation/articles/documentdb-dotnet-samples/) 适用于 .NET、Python、Node.js、JavaScript 和 Java。开发人员也可以利用 [RESTful HTTP API](https://msdn.microsoft.com/library/azure/dn781481.aspx)，从各种平台、使用各种语言与 DocumentDB 资源进行交互。

GitHub 上提供 DocumentDB [.NET](/documentation/articles/documentdb-dotnet-samples/)、[Java](https://github.com/Azure/azure-documentdb-java)、[Node.js](/documentation/articles/documentdb-nodejs-samples/) 和 [Python](/documentation/articles/documentdb-python-samples/) SDK 的示例。

### DocumentDB 是否支持 SQL？
DocumentDB SQL 查询语言是 SQL 支持的查询功能增强子集。DocumentDB SQL 查询语言通过基于 JavaScript 的用户定义函数 (UDF)，提供丰富的分层和关系运算符以及可扩展性。JSON 语法允许将 JSON 文档模型化为以标签作为树节点的树状，由 DocumentDB 的自动索引技术及 DocumentDB 的 SQL 查询方言使用。有关如何使用 SQL 语法的详细信息，请参阅[查询 DocumentDB][query] 一文。

### DocumentDB 支持什么数据类型？
DocumentDB 支持的基元数据类型与 JSON 相同。JSON 有一套简单的类型系统，包含字符串、数值（IEEE754 双精度）和布尔值（true、false 和 Null）。通过使用 { } 运算符创建嵌套对象和使用 [ ] 运算符创建数组，可以在 JSON 和 DocumentDB 中表示更复杂的数据类型（例如 DateTime、Guid、Int64 和 Geometry）。

### DocumentDB 如何提供并发？
DocumentDB 通过 HTTP 实体标记或 ETag 支持积极并发控制 (OCC)。每个 DocumentDB 资源都有一个 ETag，DocumentDB 客户端会将最新读取的版本包含在写入请求中。如果 ETag 是最新的，表示已提交更改。如果值已从外部更改，则服务器会拒绝写入，并返回“HTTP 412 Precondition failure”响应代码。客户端必须读取最新版的资源，然后重试请求。

### 我如何在 DocumentDB 中执行事务？
DocumentDB 通过 JavaScript 存储过程和触发器支持语言集成式事务。如果是单一分区集合，则脚本内的所有数据库操作会在集合范围内的快照隔离下执行；如果集合已分区，则在集合中具有相同分区键值的文档下执行。文档版本 (ETag) 的快照是在事务开始时获取的，且只有当脚本成功运行时才会提交。如果 JavaScript 引发错误，则会回滚事务。有关更多详细信息，请参阅 [DocumentDB 服务器端编程](/documentation/articles/documentdb-programming/)。

### 我如何将文档批量插入到 DocumentDB？ 
可通过三种方式将文件批量插入 DocumentDB：

- 数据迁移工具，如[将数据导入 DocumentDB](/documentation/articles/documentdb-import-data/) 所述。
- Azure 门户预览中的文档资源管理器，如[使用文档资源管理器批量添加文档](/documentation/articles/documentdb-view-json-document-explorer/#BulkAdd)所述。
- 存储过程，如 [DocumentDB 服务器端编程](/documentation/articles/documentdb-programming/)所述。

### DocumentDB 是否支持资源链接缓存？
是，因为 DocumentDB 是 RESTful 服务，而资源链接固定不变，所以可以缓存。DocumentDB 客户端可以指定“If-None-Match”标头来读取任何资源，例如文档或集合，且只有当服务器版本更改时才会更新本地副本。




[azure-portal]: https://portal.azure.cn
[query]: /documentation/articles/documentdb-sql-query/
 
<!---HONumber=Mooncake_0425_2016-->