<properties 
	pageTitle="DocumentDB 的 NoSQL Node.js 示例 | Azure" 
	description="在 github 上找到 NoSQL Node.js 示例用于执行 DocumentDB 中的常见任务，包括 NoSQL 数据库中 JSON 文档的 CRUD 操作。" 
	keywords="Node.js 示例"
	services="documentdb" 
	authors="mimig1" 
	manager="jhubbard" 
	editor="monicar" 
	documentationCenter="nodejs"/>

<tags 
	ms.service="documentdb" 
	ms.date="03/22/2016" 
	wacn.date="06/29/2016"/>


# DocumentDB Node.js 示例

> [AZURE.SELECTOR]
- [.NET 示例](/documentation/articles/documentdb-dotnet-samples/)
- [Node.js 示例](/documentation/articles/documentdb-nodejs-samples/)
- [Python 示例](/documentation/articles/documentdb-python-samples/)
- [Azure 代码示例库](https://azure.microsoft.com/documentation/samples/?service=documentdb)

对 DocumentDB 资源执行 CRUD 操作和其他常见操作的示例解决方案包括在 [azure-documentdb-nodejs](https://github.com/Azure/azure-documentdb-node/tree/master/samples) GitHub 存储库。本文将提供：

- 指向每个 Node.js 示例项目文件中任务的链接。 
- 指向相关的 API 参考内容的链接。

**先决条件**

1. 你需要一个 Azure 帐户以使用这些 Node.js 示例：
    - 可以[免费建立一个 Azure 帐户](/pricing/free-trial/)：获取可用来试用付费版 Azure 服务的信用额度，甚至在用完信用额度后，你仍可以保留帐户和使用免费的 Azure 服务（如网站）。你的信用卡将永远不会付费，除非你显式更改设置并要求付费。

2. 你还需要 [Node.js SDK](/documentation/articles/documentdb-sdk-node/)。 

    > [AZURE.NOTE] 每个示例都是独立的，自行对自身进行设置并在完成后自行进行清理。因此，这些示例对 [DocumentClient.createCollection](http://azure.github.io/azure-documentdb-node/DocumentClient.html) 发出多个调用。每次执行完此操作，均会按照正在创建的集合的性能等级，向你的订阅收取使用 1 小时的费用。

## 数据库示例

[DatabaseManagement](https://github.com/Azure/azure-documentdb-node/tree/master/samples/DatabaseManagement) 项目的 [app.js](https://github.com/Azure/azure-documentdb-node/blob/master/samples/DatabaseManagement/app.js) 文件演示如何执行以下任务。

任务 | API 参考
--- | ---
[创建数据库](https://github.com/Azure/azure-documentdb-node/blob/ef53e5f6707a5dc45920fb6ad54d9c7e008a6c18/samples/DocumentDB.Samples.DatabaseManagement/app.js#L121-L131) | [DocumentClient.createDatabase](http://azure.github.io/azure-documentdb-node/DocumentClient.html#createDatabase)
[查询数据库帐户](https://github.com/Azure/azure-documentdb-node/blob/ef53e5f6707a5dc45920fb6ad54d9c7e008a6c18/samples/DocumentDB.Samples.DatabaseManagement/app.js#L146-L171) | [DocumentClient.queryDatabases](http://azure.github.io/azure-documentdb-node/DocumentClient.html#queryDatabase)
[按 ID 读取数据库](https://github.com/Azure/azure-documentdb-node/blob/ef53e5f6707a5dc45920fb6ad54d9c7e008a6c18/samples/DocumentDB.Samples.DatabaseManagement/app.js#L89-L99) | [DocumentClient.readDatabase](http://azure.github.io/azure-documentdb-node/DocumentClient.html#readDatabase)
[列出帐户的数据库](https://github.com/Azure/azure-documentdb-node/blob/ef53e5f6707a5dc45920fb6ad54d9c7e008a6c18/samples/DocumentDB.Samples.DatabaseManagement/app.js#L111-L119) | [DocumentClient.readDatabases](http://azure.github.io/azure-documentdb-node/DocumentClient.html#readDatabase)
[删除数据库](https://github.com/Azure/azure-documentdb-node/blob/ef53e5f6707a5dc45920fb6ad54d9c7e008a6c18/samples/DocumentDB.Samples.DatabaseManagement/app.js#L133-L144) | [DocumentClient.deleteDatabase](http://azure.github.io/azure-documentdb-node/DocumentClient.html#deleteDatabase)

## 集合示例 

[CollectionManagement](https://github.com/Azure/azure-documentdb-node/tree/master/samples/CollectionManagement) 项目的 [app.js](https://github.com/Azure/azure-documentdb-node/blob/master/samples/CollectionManagement/app.js) 文件演示如何执行以下任务。

任务 | API 参考
--- | ---
[创建集合](https://github.com/Azure/azure-documentdb-node/blob/ef53e5f6707a5dc45920fb6ad54d9c7e008a6c18/samples/DocumentDB.Samples.CollectionManagement/app.js#L97-L118) | [DocumentClient.createCollection](http://azure.github.io/azure-documentdb-node/DocumentClient.html#createCollection)
[在数据库中读取所有集合的列表](https://github.com/Azure/azure-documentdb-node/blob/ef53e5f6707a5dc45920fb6ad54d9c7e008a6c18/samples/DocumentDB.Samples.CollectionManagement/app.js#L120-L130) | [DocumentClient.listCollections](http://azure.github.io/azure-documentdb-node/DocumentClient.html#listCollections)
[按 \_self 获取集合](https://github.com/Azure/azure-documentdb-node/blob/ef53e5f6707a5dc45920fb6ad54d9c7e008a6c18/samples/DocumentDB.Samples.CollectionManagement/app.js#L132-L141) | [DocumentClient.readCollection](http://azure.github.io/azure-documentdb-node/DocumentClient.html#readCollection)
[按 ID 获取集合](https://github.com/Azure/azure-documentdb-node/blob/ef53e5f6707a5dc45920fb6ad54d9c7e008a6c18/samples/DocumentDB.Samples.CollectionManagement/app.js#L143-L156) | [DocumentClient.readCollection](http://azure.github.io/azure-documentdb-node/DocumentClient.html#readCollection)
[获取集合的性能等级](https://github.com/Azure/azure-documentdb-node/blob/ef53e5f6707a5dc45920fb6ad54d9c7e008a6c18/samples/DocumentDB.Samples.CollectionManagement/app.js#L158-L186) | [DocumentQueryable.queryOffers](http://azure.github.io/azure-documentdb-node/DocumentClient.html#queryOffers)
[更改集合的性能等级](https://github.com/Azure/azure-documentdb-node/blob/ef53e5f6707a5dc45920fb6ad54d9c7e008a6c18/samples/DocumentDB.Samples.CollectionManagement/app.js#L188-L202) | [DocumentClient.replaceOffer](http://azure.github.io/azure-documentdb-node/DocumentClient.html#replaceOffer)
[删除集合](https://github.com/Azure/azure-documentdb-node/blob/ef53e5f6707a5dc45920fb6ad54d9c7e008a6c18/samples/DocumentDB.Samples.CollectionManagement/app.js#L204-L215) | [DocumentClient.deleteCollection](http://azure.github.io/azure-documentdb-node/DocumentClient.html#deleteCollection)

## 文档示例

[DocumentManagement](https://github.com/Azure/azure-documentdb-node/tree/master/samples/DocumentManagement) 项目的 [app.js](https://github.com/Azure/azure-documentdb-node/blob/master/samples/DocumentManagement/app.js) 文件演示如何执行以下任务。

任务 | API 参考
--- | ---
[创建文档](https://github.com/Azure/azure-documentdb-node/blob/ef53e5f6707a5dc45920fb6ad54d9c7e008a6c18/samples/DocumentDB.Samples.DocumentManagement/app.js#L153-L177) | [DocumentClient.createDocument](http://azure.github.io/azure-documentdb-node/DocumentClient.html#createDocument)
[读取集合的文档源](https://github.com/Azure/azure-documentdb-node/blob/ef53e5f6707a5dc45920fb6ad54d9c7e008a6c18/samples/DocumentDB.Samples.DocumentManagement/app.js#L179-L189) | [DocumentClient.readDocument](http://azure.github.io/azure-documentdb-node/DocumentClient.html#readDocument)
[按 ID 读取文档](https://github.com/Azure/azure-documentdb-node/blob/ef53e5f6707a5dc45920fb6ad54d9c7e008a6c18/samples/DocumentDB.Samples.DocumentManagement/app.js#L191-L201) | [DocumentClient.readDocument](http://azure.github.io/azure-documentdb-node/DocumentClient.html#readDocument)
[仅当文档已更改时读取文档](https://github.com/Azure/azure-documentdb-node/blob/0778eadea7abb2af41e8c22a239dc872c584f421/samples/DocumentManagement/app.js#L79-L107) | [DocumentClient.readDocument](http://azure.github.io/azure-documentdb-node/DocumentClient.html#readDocument)<br/>[RequestOptions.accessCondition](http://azure.github.io/azure-documentdb-node/global.html#RequestOptions)
[查询文档](https://github.com/Azure/azure-documentdb-node/blob/ef53e5f6707a5dc45920fb6ad54d9c7e008a6c18/samples/DocumentDB.Samples.DocumentManagement/app.js#L82-L110) | [DocumentClient.queryDocuments](http://azure.github.io/azure-documentdb-node/DocumentClient.html#queryDocuments) 
[替换文档](https://github.com/Azure/azure-documentdb-node/blob/ef53e5f6707a5dc45920fb6ad54d9c7e008a6c18/samples/DocumentDB.Samples.DocumentManagement/app.js#L112-L119) | [DocumentClient.replaceDocument](http://azure.github.io/azure-documentdb-node/DocumentClient.html#replaceDocument)
[使用条件 ETag 检查替换文档](https://github.com/Azure/azure-documentdb-node/blob/0778eadea7abb2af41e8c22a239dc872c584f421/samples/DocumentManagement/app.js#L147-L164) | [DocumentClient.replaceDocument](http://azure.github.io/azure-documentdb-node/DocumentClient.html#replaceDocument)<br/>[RequestOptions.accessCondition](http://azure.github.io/azure-documentdb-node/global.html#RequestOptions)
[删除文档](https://github.com/Azure/azure-documentdb-node/blob/ef53e5f6707a5dc45920fb6ad54d9c7e008a6c18/samples/DocumentDB.Samples.DocumentManagement/app.js#L122-L133) | [DocumentClient.deleteDocument](http://azure.github.io/azure-documentdb-node/DocumentClient.html#deleteDocument)

## 索引示例

[IndexManagement](https://github.com/Azure/azure-documentdb-node/tree/master/samples/IndexManagement) 项目的 [app.js](https://github.com/Azure/azure-documentdb-node/blob/master/samples/IndexManagement/app.js) 文件演示如何执行以下任务。

任务 | API 参考
--- | ---
使用默认索引创建集合 | [DocumentClient.createDocument](http://azure.github.io/azure-documentdb-node/DocumentClient.html)
[手动为特定文档编制索引](https://github.com/Azure/azure-documentdb-node/blob/ef53e5f6707a5dc45920fb6ad54d9c7e008a6c18/samples/DocumentDB.Samples.IndexManagement/app.js#L185-L238) | [indexingDirective: 'include'](http://azure.github.io/azure-documentdb-node/global.html#indexingDirective)
[手动从索引中排除特定文档](https://github.com/Azure/azure-documentdb-node/blob/ef53e5f6707a5dc45920fb6ad54d9c7e008a6c18/samples/DocumentDB.Samples.IndexManagement/app.js#L120-L183) | [RequestOptions.indexingDirective](http://azure.github.io/azure-documentdb-node/global.html#RequestOptions)
[对批量导入使用延迟索引或读取大量集合](https://github.com/Azure/azure-documentdb-node/blob/ef53e5f6707a5dc45920fb6ad54d9c7e008a6c18/samples/DocumentDB.Samples.IndexManagement/app.js#L240-L269) | [IndexingMode.Lazy](http://azure.github.io/azure-documentdb-node/global.html#indexingmode)
[在索引中包含文档的特定路径](https://github.com/Azure/azure-documentdb-node/blob/ef53e5f6707a5dc45920fb6ad54d9c7e008a6c18/samples/DocumentDB.Samples.IndexManagement/app.js#L433-L444) | [IndexingPolicy.IncludedPaths](http://azure.github.io/azure-documentdb-node/global.html#indexingpolicy) 
[从索引中排除某些路径](https://github.com/Azure/azure-documentdb-node/blob/ef53e5f6707a5dc45920fb6ad54d9c7e008a6c18/samples/DocumentDB.Samples.IndexManagement/app.js#L427-L450) | [ExcludedPath](http://azure.github.io/azure-documentdb-node/global.html#indexingpolicy)
[允许在范围操作期间扫描字符串路径](https://github.com/Azure/azure-documentdb-node/blob/ef53e5f6707a5dc45920fb6ad54d9c7e008a6c18/samples/DocumentDB.Samples.IndexManagement/app.js#L271-L347)| [ExcludedPath.EnableScanInQuery](http://azure.github.io/azure-documentdb-node/global.html#FeedOptions)
[在字符串路径上创建范围索引](https://github.com/Azure/azure-documentdb-node/blob/ef53e5f6707a5dc45920fb6ad54d9c7e008a6c18/samples/DocumentDB.Samples.IndexManagement/app.js#L349-L425) | [DocumentClient.queryDocument](http://azure.github.io/azure-documentdb-node/DocumentClient.html#queryDocument)
[使用默认 indexPolicy 创建集合，然后联机进行更新](https://github.com/Azure/azure-documentdb-node/blob/ef53e5f6707a5dc45920fb6ad54d9c7e008a6c18/samples/DocumentDB.Samples.IndexManagement/app.js#L519-L614) | [DocumentClient.createCollection](http://azure.github.io/azure-documentdb-node/DocumentClient.html#createCollection)<br>[DocumentClient.replaceCollection#replaceCollection](http://azure.github.io/azure-documentdb-node/DocumentClient.html)

有关索引的详细信息，请参阅 [DocumentDB 索引策略](/documentation/articles/documentdb-indexing-policies/)。

## 服务器端编程示例

[ServerSideScripts](https://github.com/Azure/azure-documentdb-node/tree/master/samples/ServerSideScripts) 项目的 [app.js](https://github.com/Azure/azure-documentdb-node/blob/master/samples/ServerSideScripts/app.js) 文件演示如何执行以下任务。

任务 | API 参考
--- | ---
[创建存储过程](https://github.com/Azure/azure-documentdb-node/blob/ef53e5f6707a5dc45920fb6ad54d9c7e008a6c18/samples/DocumentDB.Samples.ServerSideScripts/app.js#L44-L71) | [DocumentClient.createStoredProcedure](http://azure.github.io/azure-documentdb-node/DocumentClient.html#createStoredProcedure)
[执行存储过程](https://github.com/Azure/azure-documentdb-node/blob/ef53e5f6707a5dc45920fb6ad54d9c7e008a6c18/samples/DocumentDB.Samples.ServerSideScripts/app.js#L73-L90) | [DocumentClient.executeStoredProcedure](http://azure.github.io/azure-documentdb-node/DocumentClient.html#executeStoredProcedure)

有关服务器端编程的详细信息，请参阅 [DocumentDB 服务器端编程：存储过程、数据库触发器和 UDF](/documentation/articles/documentdb-programming/)。

## 分区示例

[Partitioning](https://github.com/Azure/azure-documentdb-node/tree/master/samples/Partitioning) 项目的 [app.js](https://github.com/Azure/azure-documentdb-node/blob/master/samples/Partitioning/app.js) 文件演示如何执行以下任务。

任务 | API 参考
--- | ---
[使用 HashPartitionResolver](https://github.com/Azure/azure-documentdb-node/blob/ce0fc3c4e70b0279091a1e03620a668d93a14fc2/samples/Partitioning/app.js#L53-L103) | [HashPartitionResolver](http://documentdb.com)

有关在 DocumentDB 中对数据进行分区的详细信息，请参阅[在 DocumentDB 中对数据进行分区和缩放](/documentation/articles/documentdb-partition-data/)。


<!---HONumber=Mooncake_0425_2016-->