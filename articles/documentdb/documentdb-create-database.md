<properties 
	pageTitle="如何在 DocumentDB 中创建数据库 | Azure" 
	description="了解如何使用在线服务门户预览为 Azure DocumentDB 创建运行速度飞快且可全局缩放的 NoSQL 数据库。" 
	keywords="如何创建数据库" 
	services="documentdb" 
	authors="mimig1" 
	manager="jhubbard" 
	editor="monicar" 
	documentationCenter=""/>

<tags 
	ms.service="documentdb" 
	ms.date="05/16/2016" 
	wacn.date="06/28/2016"/>

# 如何使用 Azure 门户预览创建 DocumentDB 数据库


若要使用 Azure DocumentDB，你必须拥有 [DocumentDB 帐户](/documentation/articles/documentdb-create-account/)、数据库、集合和文档。本教程说明如何在 Azure 门户预览中创建 DocumentDB 数据库。有关如何使用某个 SDK 创建数据库的信息，请参阅[创建 DocumentDB 数据库的其他方法](#other-ways-to-create-a-documentdb-database)。

![屏幕截图：演示如何创建数据库，突出显示“浏览”边栏选项卡上的“DocumentDB 帐户”以及“DocumentDB 帐户”边栏选项卡上的 DocumentDB 帐户](./media/documentdb-create-database/docdb-database-creation-1-2.png)

1.  在 [Azure 门户预览](https://portal.azure.cn/)的跳转栏中，单击“DocumentDB 帐户”。如果“DocumentDB 帐户”不可见，则单击“浏览”，再单击“DocumentDB 帐户”。

2.  在“DocumentDB 帐户”边栏选项卡中，选择要在其中添加 DocumentDB NoSQL 数据库的帐户。如果你没有任何列出的帐户，则需要[创建一个 DocumentDB 帐户](/documentation/articles/documentdb-create-account/)。

    ![屏幕截图：演示如何创建数据库，突出显示“添加数据库”按钮、“ID”框和“确定”按钮](./media/documentdb-create-database/docdb-database-creation-3-5.png)

3. 在“DocumentDB 帐户”边栏选项卡中单击“添加数据库”。

4. 在“添加数据库”边栏选项卡中输入新数据库的 ID。对名称进行验证后，“ID”框中会出现一个绿色的对号标记。

5. 单击屏幕底部的“确定”，以创建新的数据库。

6. 新的数据库现在便会出现在“DocumentDB 帐户”边栏选项卡上的“数据库”可重用功能区中。
 
	![屏幕截图：“DocumentDB 帐户”边栏选项卡中的新数据库](./media/documentdb-create-database/docdb-database-creation-6.png)

## <a name="other-ways-to-create-a-documentdb-database"></a>创建 DocumentDB 数据库的其他方法

数据库不一定要使用门户预览来创建，你也可以使用 [DocumentDB SDK](/documentation/articles/documentdb-sdk-dotnet/) 或 [REST API](https://msdn.microsoft.com/library/mt489072.aspx) 来创建数据库。有关使用 .NET SDK 处理数据库的信息，请参阅 [.NET 数据库示例](/documentation/articles/documentdb-dotnet-samples/#database-examples)。有关使用 Node.js SDK 处理数据库的信息，请参阅 [Node.js 数据库示例](/documentation/articles/documentdb-nodejs-samples/#database-examples)。

## 后续步骤

现在，你已了解如何创建 DocumentDB 数据库，下一步是[创建集合](/documentation/articles/documentdb-create-collection/)。

创建集合之后，你可以使用门户预览中的文档资源管理器来[添加 JSON 文档](/documentation/articles/documentdb-view-json-document-explorer/)，使用 DocumentDB 数据迁移工具向集合[导入文档](/documentation/articles/documentdb-import-data/)，或使用某个 [DocumentDB SDK](/documentation/articles/documentdb-sdk-dotnet/) 来执行 CRUD 操作。DocumentDB 有 .NET、Java、Python、Node.js 和 JavaScript API SDK。有关说明如何创建、移除、更新和删除文档的 .NET 代码示例，请参阅 [.NET 文档示例](/documentation/articles/documentdb-dotnet-samples/#document-examples)。有关使用 Node.js SDK 处理文档的信息，请参阅 [Node.js 文档示例](/documentation/articles/documentdb-nodejs-samples/#document-examples)。

当集合中有文档后，你就可以利用门户预览中的[查询资源管理器](/documentation/articles/documentdb-query-collections-query-explorer/)、[REST API](https://msdn.microsoft.com/library/azure/dn781481.aspx) 或某个 [SDK](/documentation/articles/documentdb-sdk-dotnet/)，来针对文档使用 [DocumentDB SQL](/documentation/articles/documentdb-sql-query/) [执行查询](/documentation/articles/documentdb-sql-query/#executing-sql-queries)。

<!---HONumber=Mooncake_0627_2016-->