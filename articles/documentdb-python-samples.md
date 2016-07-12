<properties 
	pageTitle="DocumentDB 的 NoSQL Python 示例 | Azure" 
	description="在 github 上找到 NoSQL Python 示例用于 DocumentDB 中的普通任务，包括 NoSQL 数据库中的 JSON 文档的 CRUD 操作。" 
	keywords="Python 示例"
	services="documentdb" 
	authors="ryancrawcour" 
	manager="jhubbard" 
	editor="monicar" 
	documentationCenter="python"/>

<tags 
	ms.service="documentdb" 
	ms.date="03/22/2016" 
	wacn.date="06/29/2016"/>


# DocumentDB Python SDK 示例

> [AZURE.SELECTOR]
- [.NET 示例](/documentation/articles/documentdb-dotnet-samples/)
- [Node.js 示例](/documentation/articles/documentdb-nodejs-samples/)
- [Python 示例](/documentation/articles/documentdb-python-samples/)
- [Azure 代码示例库](https://azure.microsoft.com/documentation/samples/?service=documentdb)

对 DocumentDB 资源执行 CRUD 操作和其他常见操作的示例解决方案包括在 [azure-documentdb-python](https://github.com/Azure/azure-documentdb-python/tree/master/samples) GitHub 存储库。本文将提供：

- 每个 Python 示例项目文件中的任务链接。 
- 指向相关的 API 参考内容的链接。

**先决条件**

1. 你需要一个 Azure 帐户以使用这些 Python 示例：
    - 可以[建议个 Azure 试用帐户](/pricing/free-trial/)：获取可用来试用付费版 Azure 服务的信用额度，甚至在用完信用额度后，你仍可以保留帐户和使用免费的 Azure 服务（如网站）。你的信用卡将永远不会付费，除非你显式更改设置并要求付费。

2. 你还需要 [Python SDK](/documentation/articles/documentdb-sdk-python/)。 

    > [AZURE.NOTE] 每个示例都是独立的，自行对自身进行设置并在完成后自行进行清理。因此，示例问题将多次调用 [document\_client.CreateCollection](http://azure.github.io/azure-documentdb-python/api/pydocumentdb.document_client.html)。每次执行完此操作，均会按照正在创建的集合的性能等级，向你的订阅收取使用 1 小时的费用。

## 数据库示例

[DatabaseManagement](https://github.com/Azure/azure-documentdb-python/tree/master/samples/DatabaseManagement) 项目的 [Program.py](https://github.com/Azure/azure-documentdb-python/tree/master/samples/DatabaseManagement/Program.py) 文件演示了如何执行以下任务。

任务 | API 参考
--- | ---
[创建数据库](https://github.com/Azure/azure-documentdb-python/blob/d78170214467e3ab71ace1a7400f5a7fa5a7b5b0/samples/DatabaseManagement/Program.py#L65-L76) | [document\_client.CreateDatabase](http://azure.github.io/azure-documentdb-python/api/pydocumentdb.document_client.html)
[查询数据库帐户](https://github.com/Azure/azure-documentdb-python/blob/d78170214467e3ab71ace1a7400f5a7fa5a7b5b0/samples/DatabaseManagement/Program.py#L49-L62) | [document\_client.QueryDatabases](http://azure.github.io/azure-documentdb-python/api/pydocumentdb.document_client.html)
[按 ID 读取数据库](https://github.com/Azure/azure-documentdb-python/blob/d78170214467e3ab71ace1a7400f5a7fa5a7b5b0/samples/DatabaseManagement/Program.py#L79-L96) | [document\_client.ReadDatabase](http://azure.github.io/azure-documentdb-python/api/pydocumentdb.document_client.html)
[列出帐户的数据库](https://github.com/Azure/azure-documentdb-python/blob/d78170214467e3ab71ace1a7400f5a7fa5a7b5b0/samples/DatabaseManagement/Program.py#L99-L110) | [document\_client.ReadDatabases](http://azure.github.io/azure-documentdb-python/api/pydocumentdb.document_client.html)
[删除数据库](https://github.com/Azure/azure-documentdb-python/blob/d78170214467e3ab71ace1a7400f5a7fa5a7b5b0/samples/DatabaseManagement/Program.py#L113-L126) | [document\_client.DeleteDatabase](http://azure.github.io/azure-documentdb-python/api/pydocumentdb.document_client.html)

## 集合示例 

[CollectionManagement](https://github.com/Azure/azure-documentdb-python/tree/master/samples/CollectionManagement) 项目的 [Program.py](https://github.com/Azure/azure-documentdb-python/tree/master/samples/CollectionManagement/Program.py) 文件演示了如何执行以下任务。

任务 | API 参考
--- | ---
[创建集合](https://github.com/Azure/azure-documentdb-python/blob/d78170214467e3ab71ace1a7400f5a7fa5a7b5b0/samples/CollectionManagement/Program.py#L84-L135) | [document\_client.CreateCollection](http://azure.github.io/azure-documentdb-python/api/pydocumentdb.document_client.html#CreateCollection)
[在数据库中读取所有集合的列表](https://github.com/Azure/azure-documentdb-python/blob/d78170214467e3ab71ace1a7400f5a7fa5a7b5b0/samples/CollectionManagement/Program.py#L198-L225) | [document\_client.ListCollections](http://azure.github.io/azure-documentdb-python/api/pydocumentdb.document_client.html#CreateCollection)
[按 ID 获取集合](https://github.com/Azure/azure-documentdb-python/blob/d78170214467e3ab71ace1a7400f5a7fa5a7b5b0/samples/CollectionManagement/Program.py#L178-L195) | [document\_client.ReadCollection](http://azure.github.io/azure-documentdb-python/api/pydocumentdb.document_client.html#CreateCollection)
[获取集合的性能等级](https://github.com/Azure/azure-documentdb-python/blob/d78170214467e3ab71ace1a7400f5a7fa5a7b5b0/samples/CollectionManagement/Program.py#L139-L161) | [DocumentQueryable.QueryOffers](http://azure.github.io/azure-documentdb-python/api/pydocumentdb.document_client.html#CreateCollection)
[更改集合的性能等级](https://github.com/Azure/azure-documentdb-python/blob/d78170214467e3ab71ace1a7400f5a7fa5a7b5b0/samples/CollectionManagement/Program.py#L163-L175) | [document\_client.ReplaceOffer](http://azure.github.io/azure-documentdb-python/api/pydocumentdb.document_client.html#CreateCollection)
[删除集合](https://github.com/Azure/azure-documentdb-python/blob/d78170214467e3ab71ace1a7400f5a7fa5a7b5b0/samples/CollectionManagement/Program.py#L212-L225) | [document\_client.DeleteCollection](http://azure.github.io/azure-documentdb-python/api/pydocumentdb.document_client.html#CreateCollection)

<!---HONumber=Mooncake_0425_2016-->