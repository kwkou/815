<properties
    pageTitle="DocumentDB 数据库问题 - 常见问题解答 | Azure"
    description="获取有关 DocumentDB（全球分布式多模型数据库服务）的常见问题的解答。 解答有关容量、性能级别和缩放的数据库问题。"
    keywords="数据库问题, 常见问题, documentdb, azure, Azure"
    services="documentdb"
    author="mimig1"
    manager="jhubbard"
    editor="monicar"
    documentationcenter="" />
<tags
    ms.assetid="b68d1831-35f9-443d-a0ac-dad0c89f245b"
    ms.service="documentdb"
    ms.workload="data-services"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="05/10/2017"
    wacn.date="05/31/2017"
    ms.author="mimig"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="4a18b6116e37e365e2d4c4e2d144d7588310292e"
    ms.openlocfilehash="3264c23ad1ef6569606c059f6f1c3b3a6c125ca2"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/19/2017" />

# <a name="frequently-asked-questions-about-azure-documentdb"></a>DocumentDB 常见问题解答
## <a name="database-questions-about-azure-documentdb-fundamentals"></a>有关 DocumentDB 基本原理的数据库问题
### <a name="what-is-azure-documentdb"></a>什么是 DocumentDB？
DocumentDB 是全球复制的多模型数据库服务，可针对无架构数据提供丰富的查询，帮助提供可靠的可配置性能，并支持快速开发，这一切都通过一个托管平台来实现，而该平台有 Azure 强大的功能与影响力作为后盾。 如果 Web、移动、游戏和 IoT 应用程序的关键要求是可预测的吞吐量、高可用性、低延迟和无架构数据模型，那么，DocumentDB 无疑是最合适的解决方案。 DocumentDB 提供架构灵活性和丰富的索引，并且对集成的 JavaScript 提供多文档事务性支持。  

有关部署和使用此服务的更多数据库问题、解答及说明，请参阅 [DocumentDB 文档页面](/documentation/services/documentdb/)。

### <a name="what-happened-to-documentdb"></a>DocumentDB 有哪些变化？
DocumentDB API 是适用于 DocumentDB 的受支持 API 和数据模型之一。 此外，DocumentDB 支持图形 API（预览）、表 API（预览）和 MongoDB API。 

### <a name="how-do-i-get-to-my-documentdb-account-in-the-azure-portal"></a>如何在 Azure 门户中访问 DocumentDB 帐户？
在 Azure 门户的左侧菜单中单击 DocumentDB 图标即可。 如果你以前已创建了一个 DocumentDB 帐户，则现在也有了一个 DocumentDB 帐户，费用不会有任何变化。

### <a name="what-are-the-typical-use-cases-for-azure-documentdb"></a>DocumentDB 的典型用例有哪些？
对于侧重于以下要求的新 Web、移动、游戏和 IoT 应用程序而言，DocumentDB 是一个不错的选择：自动缩放、可预测的性能、毫秒响应时间的快速排序，以及查询无架构数据的能力。 DocumentDB 有助于快速开发，且支持应用程序数据模型的连续迭代。  用于管理用户生成的内容和数据的应用程序就是 [DocumentDB 的常见用例](/documentation/articles/documentdb-use-cases/)。  

### <a name="how-does-azure-documentdb-offer-predictable-performance"></a>DocumentDB 如何提供可预测的性能？
[请求单位](/documentation/articles/documentdb-request-units/)是 DocumentDB 中吞吐量的衡量单位。 1 个 RU 相当于获取 1KB 文档的吞吐量。 在 DocumentDB 中进行的每个操作（包括读、写、SQL 查询和执行存储过程）都具有一个确定的 RU 值，该值基于完成该操作所需的吞吐量。 你无需考虑 CPU、IO 和内存，以及它们会怎样影响你的应用程序吞吐量，而是从单个 RU 度量值的角度进行考虑。

每个 DocumentDB 容器可以保留以每秒 RU 表示的预配吞吐量。 对于任何规模的应用程序，都可以将单个请求设为基准来测量其 RU 值，并预配容器来处理所有请求的请求单位总和。 也可以随着应用程序的发展需求，扩展或缩减容器的吞吐量。 有关请求单位的详细信息以及确定容器需求方面的帮助，请阅读[估计吞吐量需求](/documentation/articles/documentdb-request-units/#estimating-throughput-needs/)并尝试使用[吞吐量计算器](https://www.documentdb.com/capacityplanner)。 此处的容器是指 DocumentDB API 的集合、图形 API 的图形、MongoDB API 的集合和表 API 的表。  


### <a name="is-azure-documentdb-hipaa-compliant"></a>DocumentDB 是否符合 HIPAA？
是的，DocumentDB 符合 HIPAA。 HIPAA 针对可识别个人身份的健康信息的使用、泄露与保护制定了要求。 有关详细信息，请参阅 [Microsoft 信任中心](https://www.microsoft.com/en-us/TrustCenter/Compliance/HIPAA)。

### <a name="what-are-the-storage-limits-of-azure-documentdb"></a>DocumentDB 的存储限制是什么？
对于容器可以存储在 DocumentDB 中的数据总量并没有任何限制。

### <a name="what-are-the-throughput-limits-of-azure-documentdb"></a>DocumentDB 的吞吐量限制是什么？
DocumentDB 中的容器可以支持的总吞吐量没有限制，但关键在于，需要在足够数量的分区键之间大致平均分配工作负荷。

### <a name="how-much-does-azure-documentdb-cost"></a>DocumentDB 的费用如何？
请参阅 [DocumentDB 定价详细信息](/pricing/details/documentdb/)页了解详细信息。 DocumentDB 使用费取决于预配的容器数、容器的联机小时数，以及每个容器的预配吞吐量。 此处的容器是指 DocumentDB API 的集合、图形 API 的图形、MongoDB API 的集合和表 API 的表。 

### <a name="is-there-a-trial-available"></a>你们是否提供试用版？
如果你是 Azure 新用户，可以注册 [Azure 试用版](/pricing/1rmb-trial/)。  

也可以使用 [DocumentDB 模拟器](/documentation/articles/documentdb-nosql-local-emulator/)在本地免费开发和测试应用程序，无需创建 Azure 订阅。 如果对应用程序在 DocumentDB 模拟器中的工作情况感到满意，则可以切换到在云中使用 DocumentDB 帐户。

### <a name="how-can-i-get-additional-help-with-azure-documentdb"></a>如何获取 DocumentDB 的更多帮助？
如果需要帮助，请在 [Stack Overflow](http://stackoverflow.com/questions/tagged/azure-documentdb) 上向我们提问。

## <a name="set-up-azure-documentdb"></a>设置 DocumentDB
### <a name="how-do-i-sign-up-for-azure-documentdb"></a>如何注册 DocumentDB？
可以在 Azure 门户中注册 DocumentDB。 首先必须注册 Azure 订阅。 注册 Azure 订阅后，可将 DocumentDB API、图形 API（预览）、表 API（预览）或 MongoDB API 帐户添加到 Azure 订阅。

### <a name="what-is-a-master-key"></a>什么是主密钥？
主密钥是用于访问帐户的所有资源的安全令牌。 拥有此密钥的人对数据库帐户中的所有资源具有读取和写入访问权。 分发主密钥时要格外谨慎。 [Azure 门户][azure-portal]的“密钥”边栏选项卡中提供主要主密钥和辅助主密钥。**** 有关密钥的详细信息，请参阅[查看、复制和重新生成访问密钥](/documentation/articles/documentdb-manage-account/#keys/)。

### <a name="is-there-something-i-should-be-aware-of-when-distributing-data-across-the-world-via-azures-data-centers"></a>通过 Azure 数据中心在全球分配数据时是否需要注意什么？ 
DocumentDB 已在所有区域推出。 由于它是核心服务，每个新数据中心都部署了 DocumentDB。 上面提供了至今为止的区域列表。 在这些区域中进行设置时，必须记住 DocumentDB 遵从主权和政府云的要求。 这意味着，如果在这些区域创建帐户并想要对外复制这些区域所不允许复制的数据，则同样无法连接到这些位置来通过外部帐户启用复制。 

### <a name="do-you-plan-to-provide-more-price-options-in-the-future"></a>你们将来是否打算提供更多的价格选项？
是的，DocumentDB 目前提供吞吐量优化模型。 我们打算在不久的将来提供更多的最佳定价模型。 

 
# <a name="documentdb-api"></a>DocumentDB API 
## <a name="database-questions-about-developing-against-documentdb-api"></a>针对 DocumentDB API 进行开发的相关数据库问题

### <a name="how-to-do-i-start-developing-against-documentdb-api"></a>如何开始针对 DocumentDB API 进行开发？
[Azure 门户][azure-portal]中已提供 Microsoft DocumentDB API。  首先必须注册 Azure 订阅。  注册 Azure 订阅后，可将 DocumentDB API 容器添加到 Azure 订阅。 有关添加 DocumentDB 帐户的说明，请参阅[创建 DocumentDB 数据库帐户](/documentation/articles/documentdb-create-account/)。 如果以前已创建了一个 DocumentDB 帐户，则现在也有了一个 DocumentDB 帐户。  

[SDK](/documentation/articles/documentdb-sdk-dotnet/) 适用于 .NET、Python、Node.js、JavaScript 和 Java。  开发人员也可以利用 [RESTful HTTP API](https://msdn.microsoft.com/zh-cn/library/azure/dn781481.aspx)，从各种平台使用各种语言与 DocumentDB 资源进行交互。

### <a name="can-i-access-some-ready-made-samples-for-getting-headstart"></a>是否可以访问一些现成的示例来帮助自己入门？
GitHub 上提供 DocumentDB API [.NET](/documentation/articles/documentdb-dotnet-samples/)、[Java](https://github.com/Azure/azure-documentdb-java)、[Node.js](/documentation/articles/documentdb-nodejs-samples/) 和 [Python](/documentation/articles/documentdb-python-samples/) SDK 的示例。


### <a name="does-documentdb-api--database-support-schema-free-data"></a>DocumentDB API 数据库是否支持无架构数据？
是的，DocumentDB API 允许应用程序存储任意的 JSON 文档，而不需要架构定义或提示。 通过 DocumentDB SQL 查询接口可立即查询数据。   

### <a name="does-documentdb-api-support-acid-transactions"></a>DocumentDB API 是否支持 ACID 事务？
是的，DocumentDB API 支持以 JavaScript 存储过程和触发器形式表示的跨文档事务。 事务以每个集合内的单个分区为范围，且以 ACID 语义执行，也就是全有或全无，与其他并发执行的代码和用户请求隔离。  如果在服务器端执行 JavaScript 应用程序代码的过程中引发了异常，则会回滚整个事务。 有关事务的详细信息，请参阅[数据库程序事务](/documentation/articles/documentdb-programming/#database-program-transactions/)。

### <a name="what-is-a-collection"></a>什么是集合？
集合是一组文档及其关联的 JavaScript 应用程序逻辑。 集合是一个计费实体，其[成本](/documentation/articles/documentdb-performance-levels/)由所使用的吞吐量和存储确定。 集合可以跨一个或多个分区/服务器，并且能伸缩以处理几乎无限制增长的存储或吞吐量。

集合也是 DocumentDB 的计费实体。 每个集合根据预配的吞吐量和使用的存储空间按小时计费。 有关详细信息，请参阅 [DocumentDB API 定价](/pricing/details/documentdb/)。  

### <a name="how-do-i-create-a-database"></a>我如何创建数据库？
可以根据[创建 DocumentDB 集合和数据库](/documentation/articles/documentdb-create-collection/)中所述使用 Azure 门户、某个 [DocumentDB SDK](/documentation/articles/documentdb-sdk-dotnet/) 或 [REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn781481.aspx) 来创建数据库。  

### <a name="how-do-i-set-up-users-and-permissions"></a>我如何设置用户和权限？
可使用某个 [DocumentDB API SDK](/documentation/articles/documentdb-sdk-dotnet/) 或通过 [REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn781481.aspx) 来创建用户和权限。   

### <a name="does-documentdb-api-support-sql"></a>DocumentDB API 是否支持 SQL？
SQL 查询语言是 SQL 支持的查询功能增强子集。 DocumentDB SQL 查询语言通过基于 JavaScript 的用户定义函数 (UDF)，提供丰富的分层和关系运算符以及可扩展性。 JSON 语法允许将 JSON 文档模型化为以标签作为树节点的树状，由 DocumentDB 的自动索引技术及 DocumentDB 的 SQL 查询方言使用。  有关如何使用 SQL 语法的详细信息，请参阅 [QueryDocumentDB][query] 一文。

### <a name="does-documentdb-api--support-sql-aggregation-functions"></a>DocumentDB API 是否支持 SQL 聚合函数？
DocumentDB API 支持通过聚合函数 `COUNT`、`MIN`、`MAX`、`AVG` 和 `SUM` 通过 SQL 语法实现的任何规模的低延迟聚合。 有关详细信息，请参阅[聚合函数](/documentation/articles/documentdb-sql-query/#Aggregates/)。

### <a name="how-does-documentdb-api--provide-concurrency"></a>DocumentDB API 如何提供并发性？
DocumentDB API 通过 HTTP 实体标记或 ETag 支持乐观并发控制 (OCC)。 每个 DocumentDB API 资源都有一个 ETag，并且会在每次更新文档时，在服务器上设置此 ETag。 ETag 标头和当前值会包含在所有响应消息中。 ETag 可与 If-Match 标头配合使用，让服务器能够决定是否应该更新资源。 If-Match 值是用作检查依据的 ETag 值。 如果 ETag 值与服务器的 ETag 值匹配，就会更新资源。 如果 ETag 不再是最新状态，则服务器会拒绝该操作，并提供“HTTP 412 前置条件失败”响应代码。 客户端接着必须重新提取资源，以获取该资源当前的 ETag 值。 此外，ETag 可以与 If-None-Match 标头配合使用，以确定是否需要重新提取资源。

若要在 .NET 中使用乐观并发，可以使用 [AccessCondition](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.client.accesscondition.aspx) 类。 相关 .NET 示例请参阅 github 上 DocumentManagement 示例中的 [Program.cs](https://github.com/Azure/azure-documentdb-dotnet/blob/master/samples/code-samples/DocumentManagement/Program.cs) 。

### <a name="how-do-i-perform-transactions-in-documentdb-api"></a>如何在 DocumentDB API 中执行事务？
DocumentDB API 通过 JavaScript 存储过程和触发器支持语言集成式事务。 如果是单一分区集合，则脚本内的所有数据库操作会在集合范围内的快照隔离下执行；如果集合已分区，则在集合中具有相同分区键值的文档下执行。 文档版本 (ETag) 的快照是在事务开始时获取的，且只有当脚本成功运行时才会提交。 如果 JavaScript 引发错误，则会回滚事务。 有关更多详细信息，请参阅 [DocumentDB API 服务器端编程](/documentation/articles/documentdb-programming/)。

### <a name="how-can-i-bulk-insert-documents-into-documentdb-api"></a>我如何将文档批量插入到 DocumentDB API？
可通过三种方式将文件批量插入 DocumentDB：

- 数据迁移工具，如[将数据导入 DocumentDB API](/documentation/articles/documentdb-import-data/) 中所述。
- Azure 门户中的文档资源管理器，如[使用文档资源管理器批量添加文档](/documentation/articles/documentdb-view-json-document-explorer/#bulk-add-documents/)所述。
- 存储过程，如 [DocumentDB API 服务器端编程](/documentation/articles/documentdb-programming/)中所述。

### <a name="does-documentdb-api-support-resource-link-caching"></a>DocumentDB API 是否支持资源链接缓存？
是的，因为 DocumentDB 是 RESTful 服务，而资源链接固定不变，所以可以缓存。 DocumentDB API 客户端可以指定“If-None-Match”标头来读取任何资源，例如文档或集合，且只有当服务器版本更改时才会更新本地副本。

### <a name="is-a-local-instance-of-documentdb-api-available"></a>DocumentDB API 的本地实例是否可用？
是的。 [DocumentDB 模拟器](/documentation/articles/documentdb-nosql-local-emulator/)提供对 DocumentDB API 服务的高保真模拟。 它支持和 DocumentDB 相同的功能，包括支持创建和查询 JSON 文档、预配集合和调整集合的规模，以及执行存储过程和触发器。 可以使用 DocumentDB 模拟器开发和测试应用程序，并通过对 DocumentDB 的连接终结点进行单一配置更改将其部署到全局范围的 Azure。

## <a name="database-questions-about-developing-against-api-for-mongodb"></a>针对 MongoDB 的 API 进行开发的相关数据库问题
### <a name="what-is-azure-documentdbs-api-for-mongodb"></a>用于 MongoDB 的 DocumentDB API 是什么？
用于 MongoDB 的 DocumentDB API 是一个兼容层，使应用程序可以使用现有的、社区支持的 Apache MongoDB API 和驱动程序轻松透明地与本机 DocumentDB 数据库引擎通信。 开发人员现在可以使用现有 MongoDB 工具链和技能构建利用 DocumentDB、受益于 DocumentDB 的独特功能（包括自动索引、备份维护、得到资金支持的服务级别协议 (SLA) 等）的应用程序。

### <a name="how-to-do-i-connect-to-my-api-for-mongodb-database"></a>如何连接到 MongoDB 的 API 数据库？
连接到用于 MongoDB 的 DocumentDB API 的最快捷方法是使用 [Azure 门户](https://portal.azure.cn)。 导航到你的帐户。 在帐户的*左导航*中，单击“快速启动”。 *快速入门*是获取用于连接到数据库的代码片段的最佳方式。 

DocumentDB 实施严格的安全要求和标准。 DocumentDB 帐户需要通过 *SSL* 进行身份验证和安全通信，因此请确保使用 TLSv1.2。

有关更多详细信息，请参阅[连接到 MongoDB 的 API 数据库](/documentation/articles/documentdb-connect-mongodb-account/)。

### <a name="are-there-additional-error-codes-for-an-api-for-mongodb-database"></a>MongoDB 的 API 数据库是否有其他错误代码？
除了常见的 MongoDB 错误代码外，MongoDB 的 API 具有其自己的特定错误代码。


| 错误               | 代码  | 说明  | 解决方案  |
|---------------------|-------|--------------|-----------|
| TooManyRequests     | 16500 | 使用的请求单位总数已超过了集合的预配请求单位率，已达到限制。 | 请考虑从 Azure 门户缩放集合的吞吐量或重试。 |
| ExceededMemoryLimit | 16501 | 作为一种多租户服务，操作已超出客户端的内存配额。 | 通过限制性更强的查询条件缩小操作的作用域，或者通过 [Azure 门户](https://portal.azure.cn/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)联系技术支持。 <br><br>*Ex:  &nbsp;&nbsp;&nbsp;&nbsp;db.getCollection('users').aggregate([<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{$match: {name: "Andy"}}, <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{$sort: {age: -1}}<br>&nbsp;&nbsp;&nbsp;&nbsp;])*) |

## <a name="database-questions-about-developing-with-azure-documentdb-table-api-preview"></a>使用 DocumentDB：表 API（预览）进行开发的相关数据库问题

### <a name="terms"></a>术语 
DocumentDB：表 API（预览）是 DocumentDB 的一款高级产品，旨在提供 Build 2017 中通告的表支持。 

标准的表 SDK 是预先存在的 Azure 存储表 SDK。 

### <a name="how-do-i-provide-the-feedback-about-the-sdk-bugs"></a>如何提供有关 SDK 和 bug 的反馈？
请使用以下方法之一分享反馈：

- [UserVoice](https://feedback.azure.com/forums/263030-documentdb)
- [MSDN 论坛](https://social.msdn.microsoft.com/forums/azure/en-US/home?forum=AzureDocumentDB)
- [堆栈溢出](http://stackoverflow.com/questions/tagged/azure-documentdb)

### <a name="what-is-the-connection-string-that-i-need-to-use-to-connect-to-the-table-api-preview"></a>连接到表 API（预览）需要使用哪个连接字符串？
连接字符串为 `DefaultEndpointsProtocol=https;AccountName=<AccountNamefromDocumentDB;AccountKey=<FromKeysPaneofDocumentDB>;TableEndpoint=https://<AccountNameFromDocumentDB>.documents.azure.cn;EndpointSuffix=core.chinacloudapi.cn`。 可从 Azure 门户中的“密钥”页获取该连接字符串。 

### <a name="is-there-any-change-to-existing-customers-that-use-the-existing-standard-table-sdk"></a>使用现有标准表 SDK 的现有客户是否需要做出任何变化？
无。 使用现有标准表 SDK 的现有客户或新客户不需要做出任何变化。 

### <a name="how-do-i-view-table-data-that-is-stored-in-azure-documentdb-for-use-with-the-table-api-review"></a>如何查看 DocumentDB 中存储的、在表 API（预览）中使用的表数据？ 
可以使用 Azure 门户浏览数据。 也可以使用下面所述的表 API（预览）代码或工具。 

### <a name="which-tools-will-work-with-table-api-preview"></a>可将哪些工具与表 API（预览）配合使用？ 
旧版的 Azure 资源管理器 (0.8.9)。

能够灵活引入采用前面所指定格式的连接字符串的工具支持新的表 API（预览）。 [Azure 存储客户端工具](/documentation/articles/storage-explorers/)页中提供了表工具列表。 

### <a name="does-powershellcli-work-with-the-new-table-api-preview-"></a>是否可将 PowerShell/CLI 与新的表 API（预览）配合使用？
我们已计划为表 API（预览）添加对 PowerShell/CLI 的支持。 

### <a name="is-the-concurrency-on-operations-controlled"></a>操作并发性是否受控制？
是的，通过使用标准表 API 中所需的 ETag 机制提供乐观并发。 

### <a name="is-odata-query-model-supported-for-entities"></a>实体是否支持 OData 查询模型？ 
是的，表 API（预览）支持 OData 查询和 Linq 查询。 

### <a name="can-i-connect-to-standard-azure-table-and-the-new-premium-table-api-preview-at-side-by-side-in-same-application-"></a>是否可以同时在同一个应用程序中连接到标准 Azure 表和新的高级表 API（预览）？ 
是的，可以创建 CloudTableClient 的两个不同实例并使其通过连接字符串指向其自身的 URI 来实现此目的。

### <a name="how-do-i-migrate-existing-table-storage-application-to-this-new-offering"></a>如何将现有的表存储应用程序迁移到此新产品？
如果想要针对现有的表存储数据使用新的表 API 产品，请联系 askdocdb@microsoft.com。 

### <a name="what-is-the-roadmap-for-this-service-when-will-other-functionality-of-standard-table-api-be-offered"></a>此服务的路线图是什么，何时推出标准表 API 的其他功能？
我们已计划在推出正式版的过程中，不断添加对 SAS 令牌、ServiceContext、统计、加密、分析和其他功能的支持。  请在 [Uservoice](https://feedback.azure.com/forums/263030-documentdb) 上提供反馈。 

### <a name="how-is-expansion-of-the-storage-size-done-for-this-service-say-i-start-with-n-gbs-of-data-and-my-data-will-grow-to-1-tb-overtime"></a>如何为此服务扩展存储大小，比如，最初我有 N GB 的数据，但一段时间后我的数据会增长到 1 TB？  
DocumentDB 允许使用横向缩放提供无限的存储。 我们的服务将会监视并有效地增大存储。 

### <a name="how-do-i-monitor-the-table-api-preview-offering"></a>如何监视表 API（预览）产品？
可以使用表 API（预览）的“指标”窗格来监视请求和存储用量。 

### <a name="how-do-i-calculate-throughput-i-require"></a>如何计算所需的吞吐量？
可以根据[估算请求单位数和数据存储](https://www.documentdb.com/capacityplanner)中所述，使用 Capacity Estimator 来计算操作所需的 TableThroughput。 一般情况下，可以 json 形式呈现实现，并为操作提供数字。 

### <a name="can-i-use-the-new-table-api-preview-sdk-locally-with-the-emulator"></a>是否可在本地将新的表 API（预览）SDK 与模拟器配合使用？
是的，使用新 SDK 时，可以在本地模拟器中使用表 API（预览）。 请从[此处](/documentation/articles/documentdb-nosql-local-emulator/)下载新模拟器。 app.config 中的 StorageConnectionString 值需是 "DefaultEndpointsProtocol=https;AccountName=localhost;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==;TableEndpoint=https://localhost:8081;EndpointSuffix=core.chinacloudapi.cn"。 

### <a name="can-my-existing-application-work-with-the-table-api-preview"></a>现有的应用程序可以使用表 API（预览）？ 
使用 .NET API 执行创建、删除、更新、查询操作时，新表 API（预览）的外围应用与现有的 Azure 标准表 SDK 兼容。 需确保提供行键，因为表 API（预览）需要分区键和行键。我们还计划在推出此服务产品的正式版的过程中，不断添加其他 SDK 支持。

### <a name="do-i-need-to-migrate-existing-azure-table-based-application-to-new-sdk-if-i-do-not-want-to-use-table-api-preview-features"></a>如果不想要使用表 API（预览）功能，是否需要将现有的基于 Azure 表的应用程序迁移到新的 SDK？
不需要，现有客户可以创建和使用现有的标准表资产，不会遇到任何类型的中断。 但是，如果不使用新的表 API（预览），则无法受益于自动索引、其他一致性选项或全局分发。 

### <a name="how-do-i-add-replication-for-the-data-in-premium-table-api-preview-across-multiple-regions-of-azure"></a>如何为高级表 API（预览）中跨多个 Azure 区域的数据添加复制功能？
可以使用 DocumentDB 门户中的全局复制设置来添加适合应用程序的区域。 若要开发全局分布式应用程序，还应添加其 PreferredLocation 信息已设置为本地区域的应用程序，以提供较低的读取延迟。 

### <a name="how-do-i-change-the-primary-write-region-for-the-account-in--premium-table-apipreview"></a>如何更改高级表 API（预览）中的帐户的主要写入区域？
可以使用 DocumentDB 的全局复制门户窗格添加区域，然后故障转移到所需的区域。 有关说明，请参阅[使用多区域 DocumentDB 帐户进行开发](/documentation/articles/documentdb-regional-failovers/)。 

### <a name="how-do-i-think-of-consistency-in-table-api-preview"></a>如何理解表 API（预览）中的一致性？ 
DocumentDB 在一致性、可用性和延迟之间提供合理的平衡。 从此版本开始，共有 5 个一致性级别可供选择。 表 API（预览）开发人员可以使用这些一致性级别。 这样，开发人员便可以选择在在表级别选择适当的一致性模型，并在查询数据时执行相应的请求。  客户端建立连接后，可以指定一致性级别 - 可通过设置 app.config 中的 TableConsistencyLevel 键值更改此级别。 表 API（预览）提供较低的读取延迟。默认情况下，你将以会话一致性读取自己的写入内容。 有关详细信息，请参阅[一致性级别](/documentation/articles/documentdb-consistency-levels/)。 标准表目前默认在区域中提供强一致性，在辅助位置提供最终一致性。  

### <a name="when-global-distribution-is-enabled---how-long-does-it-take-to-replicate-the-data"></a>启用全局分发后，需要花费多长时间复制数据？
我们会在本地区域持续提交数据，然后在几毫秒内将数据立即推送到其他区域，这种复制仅依赖于数据中心的 RTT。 请在 [DocumentDB：Azure 上的全球分布式数据库服务](/documentation/articles/documentdb-distribute-data-globally/)中了解 DocumentDB 的全球分发功能。

### <a name="can-the-read-request-consistency-be-changed"></a>是否可以更改读取请求一致性？
DocumentDB 允许在容器（表）级别设置一致性。 SDK 还允许通过在 app.config 文件中提供 TableConsistencyLevel 键值来更改级别。 下面是可能的值：Strong|Bounded Staleness|Session|ConsistentPrefix|Eventual。 [一致性级别](/documentation/articles/documentdb-consistency-levels/)一文中已对此做了介绍。 此处要记住的要点是，请求一致性不能高于针对表设置的级别。 例如，如果对表设置了最终一致性级别，对请求设置了强一致性级别，则这种设置将不起作用。 

### <a name="how-does-premium-table-api-preview-account-take-care-of-failover-in-case-a-region-goes-down"></a>区域发生故障时，高级表 API（预览）帐户如何处理故障转移？ 
高级表 API（预览）利用 DocumentDB 的全球分布式平台。 若要确保应用程序能够容许数据中心停机，需要在 DocumentDB 门户中为帐户额外启用至少一个区域。请参阅[使用多区域 DocumentDB 帐户进行开发](/documentation/articles/documentdb-regional-failovers/)。 可以使用门户设置区域的优先级。请参阅[使用多区域 DocumentDB 帐户进行开发](/documentation/articles/documentdb-regional-failovers/)。 可为帐户添加任意数目的区域，并通过提供优先级来控制可将该帐户故障转移到哪个位置。 当然，毫无疑问还需要在帐户中提供一个应用程序，以利用数据库。 这样，客户就不会遇到停机情况。 客户端 SDK 可自动寻址 - 它能够检测到有故障的区域，并自动故障转移到新区域。

### <a name="is-premium-table-api-preview-enabled-for-backups"></a>高级表 API（预览）是否支持备份？
是的，高级表 API（预览）利用 DocumentDB 的平台进行备份。 可自动创建备份。 [使用 DocumentDB 联机备份和还原](/documentation/articles/documentdb-online-backup-and-restore/)中已介绍 DocumentDB 备份

 
### <a name="does-the-table-api-preview-index-all-attributes-of-entities-by-default"></a>默认情况下，表 API（预览）是否为实体的所有属性编制索引？
是的，默认情况下会为实体的所有属性编制索引。 [DocumentDB：索引策略](/documentation/articles/documentdb-indexing-policies/)一文中介绍了有关索引的详细信息。 

### <a name="does-this-mean-i-do-not-have-to-create-different-indexes-to-satisfy-the-queries"></a>这是否意味着，无需创建不同的索引来满足查询要求？ 
是的，DocumentDB 针对所有属性提供自动索引，无需任何架构定义。 这可以让开发人员将工作重点放在应用程序上，而不必考虑索引的创建和管理。 [DocumentDB：索引策略](/documentation/articles/documentdb-indexing-policies/)一文中介绍了有关索引的详细信息。

适当地将这些设置编码并转义。

    {
      "indexingMode": "consistent",
      "automatic": true,
      "includedPaths": [
        {
          "path": "/somepath",
          "indexes": [
            {
              "kind": "Range",
              "dataType": "Number",
              "precision": -1
            },
            {
              "kind": "Range",
              "dataType": "String",
              "precision": -1
            } 
          ]
        }
      ],
      "excludedPaths": 
    [
     {
          "path": "/anotherpath"
     }
    ]
    }

### <a name="cosmos-db-as-a-platform-seems-to-have-lot-of-capabilities-like-sorting-aggregates-hierarchy-and-other-functionality---is-this-planned-to-be-added-to-the-table-api-"></a>平台形式的 DocumentDB 似乎有许多的功能，例如排序、聚合、分层和其他功能 - 你们是否已计划将这些功能添加到表 API？ 
在预览版中，对于表 API，DocumentDB 支持与 Azure 表存储相同的查询功能。 DocumentDB 还支持排序、聚合、地理空间查询、层次结构和各种内置函数。 在将来的服务更新中，表 API 将提供更多功能。 有关这些功能的概述，请参阅 [DocumentDB 查询](/documentation/articles/documentdb-sql-query/)。
 
### <a name="when-should-i-change-tablethroughput-for-the-table-api-preview"></a>何时应更改表 API（预览）的 TableThroughput？
对数据执行 ETL 或者想要在短时间内上传大量数据时，应更改 TableThroughput。 

或

由于指标中的“已用吞吐量”超过“预配吞吐量”，并且资源受到限制，因此需要后端容器提供更大的吞吐量。 [设置吞吐量](/documentation/articles/documentdb-set-throughput/)一文中对此做了介绍。

### <a name="can-i-scale-up-or-down-the-throughput-of-my-table-api-preview-table"></a>是否可以扩展或缩减表 API（预览）表的吞吐量？ 
是的，可以使用 DocumentDB 门户的缩放窗格执行此操作。 [设置吞吐量](/documentation/articles/documentdb-set-throughput/)主题中对此做了介绍。

### <a name="can-premium-table-api-preview-leverage-ru-per-minute-offering"></a>高级表 API（预览）是否可以利用“每分钟 RU 数”服务？ 
是的，高级表（预览）利用 DocumentDB 的功能来提供有关性能、延迟、可用性和一致性的 SLA。 因此，可以确保它能根据[请求单位](/documentation/articles/documentdb-request-units/)中所述利用“每分钟 RU 数”服务。 借助此功能，客户无需预配峰值时的吞吐量，而且能使工作负荷的尖峰变得平缓。

### <a name="is-there-a-default-tablethroughput-which-is-set-for-newly-provisioned-tables"></a>新预配的表是否设置了默认的 TableThroughput？
是的，如果未通过 app.config 替代 TableThroughput，并且未使用 DocumentDB 中预先创建的容器，则该服务将创建吞吐量为 400 的表。
 
### <a name="is-there-any-change-of-pricing-for-existing-customers-of-standard-table-api"></a>适用于现有标准表 API 客户的价格是否有变化？
无。 适用于现有标准表 API 客户的价格没有变化。 

### <a name="how-is-the-price-calculated-for-the-table-apipreview"></a>表 API（预览）的价格是怎样计算的？ 
取决于分配的 TableThroughput。 

### <a name="how-do-i-handle-any-throttling-on-the-tables-in-table-api-preview-offering"></a>如何在表 API（预览）产品中处理对表设置的任何限制？ 
如果请求率超过了底层容器的预配吞吐量容量，则会出现错误，SDK 会使用重试策略重试调用。

### <a name="why-do-i-need-to-choose-a-throughput-apart-from-partitionkey-and-rowkey-to-leverage-premium-table-preview-offering-of-documentdb"></a>为何需要选择吞吐量而不是 PartitionKey 和 RowKey 来利用 DocumentDB 的高级表（预览）产品？
如果未在 app.config 中提供吞吐量，DocumentDB 将为容器设置默认的吞吐量。 

DocumentDB 针对操作设置上限，在性能和延迟方面提供保证。 如果引擎可以针对租户的操作实施调控，则可以做到这一点。 简单而言，设置 TableThroughput 可确保在吞吐量和延迟方面获得保障，因为平台现在可保留此容量，保证操作成功。  
吞吐量规范还允许弹性更改吞吐量，以利用应用程序的季节性，满足吞吐量需求并节省成本。

### <a name="azure-storage-sdk-has-been-very-cheap-for-me-as-i-only-pay-to-store-the-data-and-i-rarely-query-documentdbs-new-offering-seems-to-be-charging-me-even-though-i-have-not-performed-single-transaction-or-stored-anything-whats-going-on-here"></a>Azure 存储 SDK 对于我而言成本非常低廉，我只需要支付数据的存储费用，同时我极少查询数据。 但是，即使我未执行任何事务或者存储任何数据，DocumentDB 的新产品似乎也要收费。 这是怎么回事？

DocumentDB 是一个全球分布式的、基于 SLA 的系统，在可用性、延迟和吞吐量方面提供保证。 如果在 DocumentDB 中保留吞吐量，获得的保障与其他系统不同。 DocumentDB 还提供客户长期以来一直期盼的其他功能 - 例如辅助索引、全局分发，等等。在预览期，我们提供吞吐量优化模型，从长远来看，我们已计划提供存储优化模型来满足客户的需求。  

### <a name="i-never-get-quota-full-indicating-a-partition-is-full-when-i-keep-ingesting-data-into-azure-table-storage-with-the-table-apipreview--i-can-get-this-error-is-this-offering-limiting-me-and-forcing-me-to-change-my-present-application"></a>在 Azure 表存储中不断引入数据时，我从未收到过“配额”已满（指示分区已满）的错误。 但使用表 API（预览）时，有时会收到此错误。 是此产品有限制，迫使我更改现有的应用程序吗？

DocumentDB 是基于 SLA 的系统，可提供无限缩放，并在延迟、吞吐量、可用性和一致性方面提供保证。 为了确保获得有保障的高级性能，需确保数据大小和索引可管理且可缩放。 针对每个分区键的实体/项数实施 10 GB 限制是为了确保提供优异的查找和查询性能。 为了确保应用程序即使对于 Azure 存储也能正常缩放，我们要求不能创建热分区（将所有信息存储在一个分区中并查询该分区。  

### <a name="so-partitionkey-and-rowkey-are-still-required-with-the-new-table-api-preview"></a>因此，新的表 API（预览）是否仍然需要 PartitionKey 和 RowKey？ 
是的。 由于表 API（预览）的外围应用与 Azure 表存储 SDK 类似，使用分区键可以更好地分配数据。 行键在该分区中是唯一的。 是的，行键需要存在，且不能为 null（在标准 SDK 中可为 null）。 在预览版中，RowKey 的长度为 255 个字节，PartitionKey 的长度为 100 个字节（即将增大到 1 KB）。 

### <a name="what-will-be-the-error-messages-of-table-api-preview-"></a>表 API（预览）显示哪些错误消息？
由于此预览版与标准表兼容，大多数错误将映射到标准表中的错误。 

### <a name="why-do-i-get-throttled-when-i-try-to-create-lot-of-tables-one-after-another--in-tables-api-preview"></a>在表 API（预览）中尝试一个接一个创建大量的表时，为何会受到限制？
DocumentDB 是基于 SLA 的系统，提供延迟、吞吐量、可用性和一致性方面的保证。 由于它是预配的系统，因此会保留资源来保证满足这些要求。 以较快的频率创建表将被系统检测到并受到限制。 我们建议查看创建表的频率，并适度地将频率降低到每分钟不超过 5 个。 请记住，表 API（预览）是预配的系统，因此，只要预配它，就必须付费。  

# <a name="graph-apipreview"></a>图形 API（预览）
## <a name="database-questions-about-developing-against-graph-apipreview"></a>针对图形 API（预览）进行开发的相关数据库问题
### <a name="how-can-i-leverage-graph-api-preview-with-documentdb"></a>如何在 DocumentDB 中利用图形 API（预览）？
可从扩展库利用图形 API（预览）功能。 此产品称为 Azure Graphs，通过 nuget 提供。 

### <a name="it-looks-like-you-support-gremlin-as-traversal-language-do-you-plan-to-add-some-more-forms-of-query-"></a>你们似乎支持使用 gremlin 作为遍历语言，是否打算添加其他一些查询形式？
是的，我们已计划将来添加其他查询机制。 

[azure-portal]: https://portal.azure.cn
[query]: /documentation/articles/documentdb-sql-query/

<!---Update_Description: wording update -->