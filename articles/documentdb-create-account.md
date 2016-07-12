<properties
	pageTitle="如何创建 DocumentDB 帐户 | Azure"
	description="使用 Azure DocumentDB 创建 NoSQL 数据库。遵循本说明文档创建 DocumentDB 帐户，并开始构建运行速度飞快且可全局缩放的 NoSQL 数据库。" 
	keywords="构建数据库"
	services="documentdb"
	documentationCenter=""
	authors="mimig1"
	manager="jhubbard"
	editor="monicar"/>

<tags
	ms.service="documentdb"
	ms.date="05/16/2016"
	wacn.date="06/30/2016"/>

# 如何使用 Azure 门户预览创建 DocumentDB 数据库帐户

> [AZURE.SELECTOR]
- [Azure 门户预览](/documentation/articles/documentdb-create-account/)
- [Azure CLI 和 ARM](/documentation/articles/documentdb-automation-resource-manager-cli/)

若要使用 Azure DocumentDB 构建数据库，必须：

- 有一个 Azure 帐户。如果你没有 Azure 帐户，可以获取[试用 Azure 帐户](https://www.azure.cn/pricing/1rmb-trial/)。 
- 创建一个 DocumentDB 帐户。  

可以使用 Azure 门户预览、Azure Resource Manager 模板或 Azure 命令行接口 (CLI) 来创建 DocumentDB 帐户。本文说明如何使用 Azure 门户预览创建数据库帐户。若要使用 Azure Resource Manager 或 Azure CLI 创建帐户，请参阅 [Automate DocumentDB database account creation（自动创建 DocumentDB 数据库帐户）](/documentation/articles/documentdb-automation-resource-manager-cli/)。

[AZURE.INCLUDE [documentdb-create-dbaccount](../includes/documentdb-create-dbaccount.md)]

## 后续步骤

现在你已经有了 DocumentDB 帐户，下一步是创建 DocumentDB 数据库。你可以使用下面其中一项来创建数据库：

- C# .NET 示例，位于 GitHub 上 [azure-documentdb-net](https://github.com/Azure/azure-documentdb-net/tree/master/samples/code-samples) 存储库的 [DatabaseManagement](https://github.com/Azure/azure-documentdb-net/tree/master/samples/code-samples/DatabaseManagement) 项目中。
- Azure 门户预览，如[使用 Azure 门户预览创建 DocumentDB 数据库](/documentation/articles/documentdb-create-database/)中所述。
- 综合教程：[.NET](/documentation/articles/documentdb-get-started/)、[.NET MVC](/documentation/articles/documentdb-dotnet-application/)、[Java](/documentation/articles/documentdb-java-application/)、[Node.js](/documentation/articles/documentdb-nodejs-application/) 或 [Python](/documentation/articles/documentdb-python-application/)。
- [DocumentDB SDK](/documentation/articles/documentdb-sdk-dotnet/)。DocumentDB 有 .NET、Java、Python、Node.js 和 JavaScript API SDK。


创建数据库后，你必须向数据库[添加一个或多个集合](/documentation/articles/documentdb-create-collection/)，然后向集合[添加文档](/documentation/articles/documentdb-view-json-document-explorer/)。

当集合中有文档后，你就可以利用门户预览中的[查询资源管理器](/documentation/articles/documentdb-query-collections-query-explorer/)、[REST API](https://msdn.microsoft.com/library/azure/dn781481.aspx) 或某个 [SDK](/documentation/articles/documentdb-sdk-dotnet/)，来针对文档使用 [DocumentDB SQL](/documentation/articles/documentdb-sql-query/) [执行查询](/documentation/articles/documentdb-sql-query/#executing-queries)。

若要详细了解 DocumentDB，请浏览以下资源：

-	[DocumentDB 资源模型和概念](/documentation/articles/documentdb-resources/)

<!---HONumber=Mooncake_0627_2016-->