<properties 
	pageTitle="如何在 DocumentDB 中创建数据库 | Azure" 
	description="了解如何使用在线服务门户为 Azure DocumentDB 创建运行速度飞快且可全局缩放的 NoSQL 数据库。" 
	keywords="如何创建数据库" 
	services="documentdb" 
	authors="mimig1" 
	manager="jhubbard" 
	editor="monicar" 
	documentationCenter=""/>  


<tags 
	ms.service="documentdb" 
	ms.workload="data-services" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="10/17/2016" 
	ms.author="mimig"
	wacn.date="11/28/2016"/>  


# 如何使用 Azure 门户预览创建 DocumentDB 数据库

若要使用 Azure DocumentDB，必须拥有 [DocumentDB 帐户](/documentation/articles/documentdb-create-account/)、数据库、集合和文档。现在，在 Azure 门户预览中创建集合的同时，会创建 DocumentDB 数据库。

若要在门户预览中创建 DocumentDB 数据库和集合，请参阅[如何使用 Azure 门户预览创建 DocumentDB 集合](/documentation/articles/documentdb-create-collection/)。

## 创建 DocumentDB 数据库的其他方法

数据库不一定要使用门户预览来创建，也可以使用 [DocumentDB SDK](/documentation/articles/documentdb-sdk-dotnet/) 或 [REST API](https://msdn.microsoft.com/zh-cn/library/mt489072.aspx) 来创建数据库。有关使用 .NET SDK 处理数据库的信息，请参阅 [.NET 数据库示例](/documentation/articles/documentdb-dotnet-samples/#database-examples/)。有关使用 Node.js SDK 处理数据库的信息，请参阅 [Node.js 数据库示例](/documentation/articles/documentdb-nodejs-samples/#database-examples/)。

## 后续步骤

创建数据库和集合后，可以使用门户预览中的文档资源管理器来[添加 JSON 文档](/documentation/articles/documentdb-view-json-document-explorer/)，使用 DocumentDB 数据迁移工具向集合[导入文档](/documentation/articles/documentdb-import-data/)，或使用某个 [DocumentDB SDK](/documentation/articles/documentdb-sdk-dotnet/) 来执行 CRUD 操作。DocumentDB 有 .NET、Java、Python、Node.js 和 JavaScript API SDK。有关说明如何创建、移除、更新和删除文档的 .NET 代码示例，请参阅 [.NET 文档示例](/documentation/articles/documentdb-dotnet-samples/#document-examples/)。有关使用 Node.js SDK 处理文档的信息，请参阅 [Node.js 文档示例](/documentation/articles/documentdb-nodejs-samples/#document-examples/)。

当集合中有文档后，就可以利用门户预览中的[查询资源管理器](/documentation/articles/documentdb-query-collections-query-explorer/)、[REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn781481.aspx) 或某个 [SDK](/documentation/articles/documentdb-sdk-dotnet/)，来针对文档使用 [DocumentDB SQL](/documentation/articles/documentdb-sql-query/) [执行查询](/documentation/articles/documentdb-sql-query/#executing-sql-queries/)。

<!---HONumber=Mooncake_1121_2016-->
