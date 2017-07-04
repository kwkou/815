<properties
    pageTitle="用于 DocumentDB 的 DocumentDB API Python 示例 | Azure"
    description="在 github 上查找有关 DocumentDB 中常见任务的 Python 示例，包括 CRUD 操作。"
    keywords="python 示例"
    services="documentdb"
    author="moderakh"
    manager="jhubbard"
    editor="monicar"
    documentationcenter="python" />
<tags
    ms.assetid="7f4f8db3-e9db-4645-92ef-7819d486a349"
    ms.service="documentdb"
    ms.workload="data-services"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="04/18/2016"
    ms.author="moderakh"
    wacn.date="05/31/2017"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="4a18b6116e37e365e2d4c4e2d144d7588310292e"
    ms.openlocfilehash="3cf0b2a4a665ff90c6431a20af6abefbf81814ca"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/19/2017" />

# <a name="azure-documentdb-python-examples"></a>DocumentDB Python 示例
> [AZURE.SELECTOR]
- [.NET 示例](/documentation/articles/documentdb-dotnet-samples/)
- [Node.js 示例](/documentation/articles/documentdb-nodejs-samples/)
- [Python 示例](/documentation/articles/documentdb-python-samples/)
- [Azure 代码示例库](https://azure.microsoft.com/documentation/samples/?service=documentdb)

对 DocumentDB 资源执行 CRUD 操作和其他常见操作的示例解决方案包含在 [azure-documentdb-python](https://github.com/Azure/azure-documentdb-python/tree/master/samples) GitHub 存储库中。 本文将提供：

- 每个 Python 示例项目文件中的任务链接。 
- 指向相关的 API 参考内容的链接。

**先决条件**

1. 需要一个 Azure 帐户才能使用这些 Python 示例：
   - 可以[建立 Azure 试用帐户](/pricing/1rmb-trial/)。

2. 还需要 [Python SDK](/documentation/articles/documentdb-sdk-python/)。 
   
   > [AZURE.NOTE]
   > 每个示例都是独立的，自行对自身进行设置并在完成后自行进行清理。 因此，这些示例对 [document_client.CreateCollection](http://azure.github.io/azure-documentdb-python/api/pydocumentdb.document_client.html) 发出多个调用。 每次执行完此操作，均会按照正在创建的集合的性能层，向订阅收取使用 1 小时的费用。 
   > 
   > 

## <a name="database-examples"></a>数据库示例
[DatabaseManagement](https://github.com/Azure/azure-documentdb-python/tree/master/samples/DatabaseManagement) 项目的 [Program.py](https://github.com/Azure/azure-documentdb-python/tree/master/samples/DatabaseManagement/Program.py) 文件演示了如何执行以下任务。

| 任务 | API 参考 |
| --- | --- |
| [创建数据库](https://github.com/Azure/azure-documentdb-python/blob/d78170214467e3ab71ace1a7400f5a7fa5a7b5b0/samples/DatabaseManagement/Program.py#L65-L76) |[document_client.CreateDatabase](http://azure.github.io/azure-documentdb-python/api/pydocumentdb.document_client.html) |
| [查询数据库帐户](https://github.com/Azure/azure-documentdb-python/blob/d78170214467e3ab71ace1a7400f5a7fa5a7b5b0/samples/DatabaseManagement/Program.py#L49-L62) |[document_client.QueryDatabases](http://azure.github.io/azure-documentdb-python/api/pydocumentdb.document_client.html) |
| [按 ID 读取数据库](https://github.com/Azure/azure-documentdb-python/blob/d78170214467e3ab71ace1a7400f5a7fa5a7b5b0/samples/DatabaseManagement/Program.py#L79-L96) |[document_client.ReadDatabase](http://azure.github.io/azure-documentdb-python/api/pydocumentdb.document_client.html) |
| [列出帐户的数据库](https://github.com/Azure/azure-documentdb-python/blob/d78170214467e3ab71ace1a7400f5a7fa5a7b5b0/samples/DatabaseManagement/Program.py#L99-L110) |[document_client.ReadDatabases](http://azure.github.io/azure-documentdb-python/api/pydocumentdb.document_client.html) |
| [删除数据库](https://github.com/Azure/azure-documentdb-python/blob/d78170214467e3ab71ace1a7400f5a7fa5a7b5b0/samples/DatabaseManagement/Program.py#L113-L126) |[document_client.DeleteDatabase](http://azure.github.io/azure-documentdb-python/api/pydocumentdb.document_client.html) |

## <a name="collection-examples"></a>集合示例
[CollectionManagement](https://github.com/Azure/azure-documentdb-python/tree/master/samples/CollectionManagement) 项目的 [Program.py](https://github.com/Azure/azure-documentdb-python/tree/master/samples/CollectionManagement/Program.py) 文件演示了如何执行以下任务。

| 任务 | API 参考 |
| --- | --- |
| [创建集合](https://github.com/Azure/azure-documentdb-python/blob/d78170214467e3ab71ace1a7400f5a7fa5a7b5b0/samples/CollectionManagement/Program.py#L84-L135) |[document_client.CreateCollection](http://azure.github.io/azure-documentdb-python/api/pydocumentdb.document_client.html#CreateCollection) |
| [读取数据库中所有集合的列表](https://github.com/Azure/azure-documentdb-python/blob/d78170214467e3ab71ace1a7400f5a7fa5a7b5b0/samples/CollectionManagement/Program.py#L198-L225) |[document_client.ListCollections](http://azure.github.io/azure-documentdb-python/api/pydocumentdb.document_client.html#CreateCollection) |
| [按 ID 获取集合](https://github.com/Azure/azure-documentdb-python/blob/d78170214467e3ab71ace1a7400f5a7fa5a7b5b0/samples/CollectionManagement/Program.py#L178-L195) |[document_client.ReadCollection](http://azure.github.io/azure-documentdb-python/api/pydocumentdb.document_client.html#CreateCollection) |
| [获取集合的性能层](https://github.com/Azure/azure-documentdb-python/blob/d78170214467e3ab71ace1a7400f5a7fa5a7b5b0/samples/CollectionManagement/Program.py#L139-L161) |[DocumentQueryable.QueryOffers](http://azure.github.io/azure-documentdb-python/api/pydocumentdb.document_client.html#CreateCollection) |
| [更改集合的性能层](https://github.com/Azure/azure-documentdb-python/blob/d78170214467e3ab71ace1a7400f5a7fa5a7b5b0/samples/CollectionManagement/Program.py#L163-L175) |[document_client.ReplaceOffer](http://azure.github.io/azure-documentdb-python/api/pydocumentdb.document_client.html#CreateCollection) |
| [删除集合](https://github.com/Azure/azure-documentdb-python/blob/d78170214467e3ab71ace1a7400f5a7fa5a7b5b0/samples/CollectionManagement/Program.py#L212-L225) |[document_client.DeleteCollection](http://azure.github.io/azure-documentdb-python/api/pydocumentdb.document_client.html#CreateCollection) |

<!---Update_Description: wording update -->