<properties
    pageTitle="DocumentDB 的 .NET NoSQL 示例 | Azure"
    description="在 github 上查找用于 DocumentDB 中的常见任务的 C# .NET NoSQL 示例，包括针对 NoSQL 数据库中的 JSON 文档执行的 CRUD 操作。"
    keywords="NoSQL 示例"
    services="documentdb"
    author="rnagpal"
    manager="jhubbard"
    editor="monicar"
    documentationcenter=".net" />
<tags
    ms.assetid="d824d517-903e-4d82-ab0a-09fc3b984c84"
    ms.service="documentdb"
    ms.workload="data-services"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/13/2017"
    wacn.date="02/27/2017"
    ms.author="rnagpal" />

# DocumentDB .NET 示例
> [AZURE.SELECTOR]
- [.NET 示例](/documentation/articles/documentdb-dotnet-samples/)
- [Node.js 示例](/documentation/articles/documentdb-nodejs-samples/)
- [Python 示例](/documentation/articles/documentdb-python-samples/)
- [Azure 代码示例库](https://azure.microsoft.com/documentation/samples/?service=documentdb)

对 Azure DocumentDB 资源执行 CRUD 操作和其他常见操作的最新示例解决方案包含在 [azure-documentdb-dotnet](https://github.com/Azure/azure-documentdb-dotnet/tree/master/samples/code-samples) GitHub 存储库中。本文将提供：

- 指向每个示例 C\# 项目文件中各项任务的链接。
- 指向相关 API 参考内容的链接。

**先决条件**

1. 你需要有 Azure 帐户才能使用这些 NoSQL 示例：
   - 可以[建立 Azure 试用版帐户](/pricing/1rmb-trial/)。

2. 你也需要 [Microsoft.Azure.DocumentDB NuGet 程序包](http://www.nuget.org/packages/Microsoft.Azure.DocumentDB/)。

> [AZURE.NOTE]
每个示例都是独立的，自行对自身进行设置并在完成后自行进行清理。同样地，这些示例对 CreateDocumentCollectionAsync\(\) 发出多个调用。每当执行此操作时，即会根据所创建的集合的性能层，对你的订阅收取使用 1 小时的费用。
> 
> 

## 数据库示例 <a name="database-examples"></a>
DatabaseManagement 项目示例的 [RunDatabaseDemo](https://github.com/Azure/azure-documentdb-dotnet/blob/530c8d9cf7c99df7300246da05206c57ce654233/samples/code-samples/DatabaseManagement/Program.cs#L72-L121) 方法演示如何执行以下任务。

| 任务 | API 参考 |
| --- | --- |
| [创建数据库](https://github.com/Azure/azure-documentdb-dotnet/blob/530c8d9cf7c99df7300246da05206c57ce654233/samples/code-samples/DatabaseManagement/Program.cs#L90) |[DocumentClient.CreateDatabaseAsync](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.client.documentclient.createdatabaseasync.aspx) |
| [Query a database](https://github.com/Azure/azure-documentdb-dotnet/blob/530c8d9cf7c99df7300246da05206c57ce654233/samples/code-samples/DatabaseManagement/Program.cs#L81)（查询数据库） |[DocumentQueryable.CreateDatabaseQuery](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.linq.documentqueryable.createdatabasequery.aspx) |
| [按 ID 读取数据库](https://github.com/Azure/azure-documentdb-dotnet/blob/530c8d9cf7c99df7300246da05206c57ce654233/samples/code-samples/DatabaseManagement/Program.cs#L102) |[DocumentClient.ReadDatabaseAsync](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.client.documentclient.readdatabaseasync.aspx) |
| [Read all the databases](https://github.com/Azure/azure-documentdb-dotnet/blob/530c8d9cf7c99df7300246da05206c57ce654233/samples/code-samples/DatabaseManagement/Program.cs#L108-L113)（读取所有数据库） |[DocumentClient.ReadDatabaseFeedAsync](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.client.documentclient.readdatabasefeedasync.aspx) |
| [删除数据库](https://github.com/Azure/azure-documentdb-dotnet/blob/530c8d9cf7c99df7300246da05206c57ce654233/samples/code-samples/DatabaseManagement/Program.cs#L118) |[DocumentClient.DeleteDatabaseAsync](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.client.documentclient.deletedatabaseasync.aspx) |

## 集合示例  <a name="collection-examples"></a>

CollectionManagement 项目示例的 [RunCollectionDemo](https://github.com/Azure/azure-documentdb-dotnet/blob/530c8d9cf7c99df7300246da05206c57ce654233/samples/code-samples/CollectionManagement/Program.cs#L96-L185) 方法演示如何执行以下任务。

| 任务 | API 参考 |
| --- | --- |
| [创建集合](https://github.com/Azure/azure-documentdb-dotnet/blob/89670bc8aefd9bdd932db7f9b6d2fcb9b6acf35e/samples/code-samples/CollectionManagement/Program.cs#L101) |[DocumentClient.CreateDocumentCollectionAsync](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.client.documentclient.createdocumentcollectionasync.aspx) |
| [Get configured performance of a collection](https://github.com/Azure/azure-documentdb-dotnet/blob/95521ff51ade486bb899d6913880995beaff58ce/samples/code-samples/CollectionManagement/Program.cs#L198)（获取集合的已配置性能） |[DocumentQueryable.CreateOfferQuery](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.linq.documentqueryable.createofferquery.aspx) |
| [Change configured performance of a collection](https://github.com/Azure/azure-documentdb-dotnet/blob/95521ff51ade486bb899d6913880995beaff58ce/samples/code-samples/CollectionManagement/Program.cs#L207)（更改集合的已配置性能） |[DocumentClient.ReplaceOfferAsync](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.client.documentclient.replaceofferasync.aspx) |
| [按 ID 获取集合](https://github.com/Azure/azure-documentdb-dotnet/blob/89670bc8aefd9bdd932db7f9b6d2fcb9b6acf35e/samples/code-samples/CollectionManagement/Program.cs#L153) |[DocumentClient.ReadDocumentCollectionAsync](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.client.documentclient.readdocumentcollectionasync.aspx) |
| [Read all the collections in a database](https://github.com/Azure/azure-documentdb-dotnet/blob/89670bc8aefd9bdd932db7f9b6d2fcb9b6acf35e/samples/code-samples/CollectionManagement/Program.cs#L162)（读取数据库中的所有集合） |[DocumentClient.ReadDocumentCollectionFeedAsync](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.client.documentclient.readdocumentcollectionfeedasync.aspx) |
| [删除集合](https://github.com/Azure/azure-documentdb-dotnet/blob/89670bc8aefd9bdd932db7f9b6d2fcb9b6acf35e/samples/code-samples/CollectionManagement/Program.cs#L175) |[DocumentClient.DeleteDocumentCollectionAsync](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.client.documentclient.deletedocumentcollectionasync.aspx) |

## 文档示例 <a name="document-examples"></a>

DocumentManagement 项目示例的 [RunDocumentsDemo](https://github.com/Azure/azure-documentdb-dotnet/blob/f374cc601f4cf08d11c88f0c3fa7dcefaf7ecfe8/samples/code-samples/DocumentManagement/Program.cs#L97-L102) 方法演示如何执行以下任务。

| 任务 | API 参考 |
| --- | --- |
| [创建文档](https://github.com/Azure/azure-documentdb-dotnet/blob/f374cc601f4cf08d11c88f0c3fa7dcefaf7ecfe8/samples/code-samples/DocumentManagement/Program.cs#L198) |[DocumentClient.CreateDocumentAsync](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.client.documentclient.createdocumentasync.aspx) |
| [按 ID 读取文档](https://github.com/Azure/azure-documentdb-dotnet/blob/f374cc601f4cf08d11c88f0c3fa7dcefaf7ecfe8/samples/code-samples/DocumentManagement/Program.cs#L211) |[DocumentClient.ReadDocumentAsync](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.client.documentclient.readdocumentasync.aspx) |
| [Read all the documents in a collection](https://github.com/Azure/azure-documentdb-dotnet/blob/f374cc601f4cf08d11c88f0c3fa7dcefaf7ecfe8/samples/code-samples/DocumentManagement/Program.cs#L227)（读取集合中的所有文档） |[DocumentClient.ReadDocumentFeedAsync](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.client.documentclient.readdocumentfeedasync.aspx) |
| [查询文档](https://github.com/Azure/azure-documentdb-dotnet/blob/f374cc601f4cf08d11c88f0c3fa7dcefaf7ecfe8/samples/code-samples/DocumentManagement/Program.cs#L248-L251) |[DocumentClient.CreateDocumentQuery](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.linq.documentqueryable.createdocumentquery.aspx) |
| [替换文档](https://github.com/Azure/azure-documentdb-dotnet/blob/f374cc601f4cf08d11c88f0c3fa7dcefaf7ecfe8/samples/code-samples/DocumentManagement/Program.cs#L263) |[DocumentClient.ReplaceDocumentAsync](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.client.documentclient.replacedocumentasync.aspx) |
| [更新插入文档](https://github.com/Azure/azure-documentdb-dotnet/blob/f374cc601f4cf08d11c88f0c3fa7dcefaf7ecfe8/samples/code-samples/DocumentManagement/Program.cs#L300) |[DocumentClient.UpsertDocumentAsync](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.client.documentclient.upsertdocumentasync.aspx) |
| [删除文档](https://github.com/Azure/azure-documentdb-dotnet/blob/f374cc601f4cf08d11c88f0c3fa7dcefaf7ecfe8/samples/code-samples/DocumentManagement/Program.cs#L322) |[DocumentClient.DeleteDocumentAsync](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.client.documentclient.deletedocumentasync.aspx) |
| [使用 .NET 动态对象](https://github.com/Azure/azure-documentdb-dotnet/blob/f374cc601f4cf08d11c88f0c3fa7dcefaf7ecfe8/samples/code-samples/DocumentManagement/Program.cs#L331-L380) |[DocumentClient.CreateDocumentAsync](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.client.documentclient.createdocumentasync.aspx)<br>[DocumentClient.ReadDocumentAsync](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.client.documentclient.readdocumentasync.aspx)<br>[DocumentClient.ReplaceDocumentAsync](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.client.documentclient.replacedocumentasync.aspx) |
| [使用条件 ETag 检查替换文档](https://github.com/Azure/azure-documentdb-dotnet/blob/f2b11dec45a195ddeed333560ebba63055f5ed09/samples/code-samples/DocumentManagement/Program.cs#L398-L440) |[DocumentClient.AccessCondition](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.client.accesscondition.aspx)<br>[Documents.Client.AccessConditionType](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.client.accessconditiontype.aspx) |
| [仅当文档已更改时读取文档](https://github.com/Azure/azure-documentdb-dotnet/blob/f2b11dec45a195ddeed333560ebba63055f5ed09/samples/code-samples/DocumentManagement/Program.cs#L442-L470) |[DocumentClient.AccessCondition](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.client.accesscondition.aspx)<br>[Documents.Client.AccessConditionType](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.client.accessconditiontype.aspx) |

## 索引示例
IndexManagement 项目示例的 [RunIndexDemo](https://github.com/Azure/azure-documentdb-dotnet/blob/ea8c977b9c2f37ddc2894911ec239907ab60e40a/samples/code-samples/IndexManagement/Program.cs#L89-L117) 方法演示如何执行以下任务。

| 任务 | API 参考 |
| --- | --- |
| [从索引中排除文档](https://github.com/Azure/azure-documentdb-dotnet/blob/2e9a48b6a446b47dd6182606c8608d439b88b683/samples/code-samples/IndexManagement/Program.cs#L125-L163) |[IndexingDirective.Exclude](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.indexingdirective.aspx) |
| [使用手动（而非自动）索引](https://github.com/Azure/azure-documentdb-dotnet/blob/2e9a48b6a446b47dd6182606c8608d439b88b683/samples/code-samples/IndexManagement/Program.cs#L171-L209) |[IndexingPolicy.Automatic](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.indexingpolicy.automatic.aspx) |
| [使用延迟（而非一致）索引](https://github.com/Azure/azure-documentdb-dotnet/blob/2e9a48b6a446b47dd6182606c8608d439b88b683/samples/code-samples/IndexManagement/Program.cs#L221-L238) |[IndexingMode.Lazy](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.indexingpolicy.indexingmode.aspx#P:Microsoft.Azure.Documents.IndexingPolicy.IndexingMode) |
| [从索引中排除指定的文档路径](https://github.com/Azure/azure-documentdb-dotnet/blob/2e9a48b6a446b47dd6182606c8608d439b88b683/samples/code-samples/IndexManagement/Program.cs#L248-L297) |[IndexingPolicy.ExcludedPaths](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.indexingpolicy.excludedpaths.aspx) |
| [对哈希索引路径强制执行范围扫描操作](https://github.com/Azure/azure-documentdb-dotnet/blob/2e9a48b6a446b47dd6182606c8608d439b88b683/samples/code-samples/IndexManagement/Program.cs#L305-L340) |[FeedOptions.EnableScanInQuery](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.client.feedoptions.enablescaninquery.aspx) |
| [对字符串使用范围索引](https://github.com/Azure/azure-documentdb-dotnet/blob/2e9a48b6a446b47dd6182606c8608d439b88b683/samples/code-samples/IndexManagement/Program.cs#L342-L405) |[IndexingPolicy.IncludedPaths](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.indexingpolicy.includedpaths.aspx)<br>[RangeIndex](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.rangeindex.aspx) |
| [执行索引转换](https://github.com/Azure/azure-documentdb-dotnet/blob/2e9a48b6a446b47dd6182606c8608d439b88b683/samples/code-samples/IndexManagement/Program.cs#L407-L464) |[ReplaceDocumentCollectionAsync](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.client.documentclient.replacedocumentcollectionasync.aspx) |

有关索引的详细信息，请参阅 [DocumentDB 索引策略](/documentation/articles/documentdb-indexing-policies/)。

## 地理空间示例
地理空间示例文件 [azure-documentdb-dotnet/samples/code-samples/Geospatial/Program.cs](https://github.com/Azure/azure-documentdb-dotnet/blob/master/samples/code-samples/Geospatial/Program.cs) 演示如何执行以下任务。

| 任务 | API 参考 |
| --- | --- |
| [对新集合启用地理空间索引](https://github.com/Azure/azure-documentdb-dotnet/blob/7b09c085817e850d683bc59bd864c2f6b552d275/samples/code-samples/Geospatial/Program.cs#L45-L63) |[IndexingPolicy](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.indexingpolicy.aspx)<br>[IndexKind.Spatial](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.indexkind.aspx)<br>[DataType.Point](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.datatype.aspx) |
| [使用 GeoJSON 点插入文档](https://github.com/Azure/azure-documentdb-dotnet/blob/7b09c085817e850d683bc59bd864c2f6b552d275/samples/code-samples/Geospatial/Program.cs#L116-L126) |[DocumentClient.CreateDocumentAsync](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.client.documentclient.createdocumentasync.aspx)<br>[DataType.Point](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.datatype.aspx) |
| [查找指定距离内的点](https://github.com/Azure/azure-documentdb-dotnet/blob/7b09c085817e850d683bc59bd864c2f6b552d275/samples/code-samples/Geospatial/Program.cs#L152-L194) |[ST\_DISTANCE](/documentation/articles/documentdb-sql-query/#built-in-functions/)<br>\[GeometryOperationExtensions.Distance\]\(https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.spatial.geometryoperationextensions.distance.aspx#M:Microsoft.Azure.Documents.Spatial.GeometryOperationExtensions.Distance(Microsoft.Azure.Documents.Spatial.Geometry,Microsoft.Azure.Documents.Spatial.Geometry\) |
| [查找多边形内的点](https://github.com/Azure/azure-documentdb-dotnet/blob/7b09c085817e850d683bc59bd864c2f6b552d275/samples/code-samples/Geospatial/Program.cs#L196-L221) |[ST\_WITHIN](/documentation/articles/documentdb-sql-query/#built-in-functions/)<br>\[GeometryOperationExtensions.Within\]\(https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.spatial.geometryoperationextensions.within.aspx#M:Microsoft.Azure.Documents.Spatial.GeometryOperationExtensions.Within(Microsoft.Azure.Documents.Spatial.Geometry,Microsoft.Azure.Documents.Spatial.Geometry\) 和<br>[多边形](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.spatial.polygon.aspx) |
| [对现有集合启用地理空间索引](https://github.com/Azure/azure-documentdb-dotnet/blob/7b09c085817e850d683bc59bd864c2f6b552d275/samples/code-samples/Geospatial/Program.cs#L312-L336) |[DocumentClient.ReplaceDocumentCollectionAsync](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.client.documentclient.replacedocumentcollectionasync.aspx)<br>[DocumentCollection.IndexingPolicy](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.documentcollection.indexingpolicy.aspx#P:Microsoft.Azure.Documents.DocumentCollection.IndexingPolicy) |
| [验证点和多边形数据](https://github.com/Azure/azure-documentdb-dotnet/blob/7b09c085817e850d683bc59bd864c2f6b552d275/samples/code-samples/Geospatial/Program.cs#L223-L265) |[ST\_ISVALID](/documentation/articles/documentdb-sql-query/#built-in-functions/)<br>[ST\_ISVALIDDETAILED](/documentation/articles/documentdb-sql-query/#built-in-functions/)<br>[GeometryOperationExtensions.IsValid](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.spatial.geometryoperationextensions.isvalid.aspx)<br>[GeometryOperationExtensions.IsValidDetailed](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.spatial.geometryoperationextensions.isvaliddetailed.aspx) |

有关使用地理空间数据的详细信息，请参阅 [使用 Azure DocumentDB 中的地理空间数据](/documentation/articles/documentdb-geospatial/)。

## 查询示例
查询文档文件 [azure-documentdb-dotnet/samples/code-samples/Queries/Program.cs](https://github.com/Azure/azure-documentdb-dotnet/blob/master/samples/code-samples/Queries/Program.cs) 演示如何通过 SQL 查询语法以及使用查询和 Lambda 的 LINQ 提供程序执行以下各项任务。

| 任务 | API 参考 |
| --- | --- |
| [查询所有文档](https://github.com/Azure/azure-documentdb-dotnet/blob/d17c0ca5be739a359d105cf4112443f65ca2cb72/samples/code-samples/Queries/Program.cs#L122-L138) |[DocumentQueryable.CreateDocumentQuery](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.linq.documentqueryable.createdocumentquery.aspx) |
| [使用 == 查询等式](https://github.com/Azure/azure-documentdb-dotnet/blob/d17c0ca5be739a359d105cf4112443f65ca2cb72/samples/code-samples/Queries/Program.cs#L251-L268) |[DocumentQueryable.CreateDocumentQuery](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.linq.documentqueryable.createdocumentquery.aspx) |
| [使用 != 和 NOT 查询不等式](https://github.com/Azure/azure-documentdb-dotnet/blob/d17c0ca5be739a359d105cf4112443f65ca2cb72/samples/code-samples/Queries/Program.cs#L270-L276) |[DocumentQueryable.CreateDocumentQuery](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.linq.documentqueryable.createdocumentquery.aspx) |
| [使用 \>、\<、\>=、\<= 等范围运算符进行查询](https://github.com/Azure/azure-documentdb-dotnet/blob/d17c0ca5be739a359d105cf4112443f65ca2cb72/samples/code-samples/Queries/Program.cs#L305-L325) |[DocumentQueryable.CreateDocumentQuery](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.linq.documentqueryable.createdocumentquery.aspx) |
| [使用范围运算符对字符串进行查询](https://github.com/Azure/azure-documentdb-dotnet/blob/d17c0ca5be739a359d105cf4112443f65ca2cb72/samples/code-samples/Queries/Program.cs#L337-L346) |[DocumentQueryable.CreateDocumentQuery](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.linq.documentqueryable.createdocumentquery.aspx) |
| [使用 Order By 进行查询](https://github.com/Azure/azure-documentdb-dotnet/blob/d17c0ca5be739a359d105cf4112443f65ca2cb72/samples/code-samples/Queries/Program.cs#L370-L392) |[DocumentQueryable.CreateDocumentQuery](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.linq.documentqueryable.createdocumentquery.aspx) |
| [使用子文档](https://github.com/Azure/azure-documentdb-dotnet/blob/d17c0ca5be739a359d105cf4112443f65ca2cb72/samples/code-samples/Queries/Program.cs#L394-L419) |[DocumentQueryable.CreateDocumentQuery](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.linq.documentqueryable.createdocumentquery.aspx) |
| [使用文档内联接进行查询](https://github.com/Azure/azure-documentdb-dotnet/blob/d17c0ca5be739a359d105cf4112443f65ca2cb72/samples/code-samples/Queries/Program.cs#L421-L435) |[DocumentQueryable.CreateDocumentQuery](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.linq.documentqueryable.createdocumentquery.aspx) |
| [使用字符串、数学和数组运算符进行查询](https://github.com/Azure/azure-documentdb-dotnet/blob/d17c0ca5be739a359d105cf4112443f65ca2cb72/samples/code-samples/Queries/Program.cs#L527-L552) |[DocumentQueryable.CreateDocumentQuery](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.linq.documentqueryable.createdocumentquery.aspx) |
| [通过使用 SqlQuerySpec 的参数化 SQL 进行查询](https://github.com/Azure/azure-documentdb-dotnet/blob/d17c0ca5be739a359d105cf4112443f65ca2cb72/samples/code-samples/Queries/Program.cs#L140-L174) |[DocumentQueryable.CreateDocumentQuery](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.linq.documentqueryable.createdocumentquery.aspx)<br>[SqlQuerySpec](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.sqlqueryspec.aspx) |
| [使用显式分页进行查询](https://github.com/Azure/azure-documentdb-dotnet/blob/d17c0ca5be739a359d105cf4112443f65ca2cb72/samples/code-samples/Queries/Program.cs#L554-L576) |[DocumentQueryable.CreateDocumentQuery](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.linq.documentqueryable.createdocumentquery.aspx) |
| [并行查询已分区集合](https://github.com/Azure/azure-documentdb-dotnet/blob/master/samples/code-samples/Queries/Program.cs#L664-L734) |[DocumentQueryable.CreateDocumentQuery](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.linq.documentqueryable.createdocumentquery.aspx) |
| [使用 Order by 针对已分区集合进行查询](https://github.com/Azure/azure-documentdb-dotnet/blob/master/samples/code-samples/Queries/Program.cs#L737-L810) |[DocumentQueryable.CreateDocumentQuery](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.linq.documentqueryable.createdocumentquery.aspx) |

有关编写查询的详细信息，请参阅 [DocumentDB 内的 SQL 查询](/documentation/articles/documentdb-sql-query/)。

## 服务器端编程示例  <a name="server-side-programming-examples"></a>
服务器端编程文件 [azure-documentdb-dotnet/samples/code-samples/ServerSideScripts/Program.cs](https://github.com/Azure/azure-documentdb-dotnet/blob/master/samples/code-samples/ServerSideScripts/Program.cs) 演示如何执行以下任务。

| 任务 | API 参考 |
| --- | --- |
| [创建存储过程](https://github.com/Azure/azure-documentdb-net/blob/d17c0ca5be739a359d105cf4112443f65ca2cb72/samples/code-samples/ServerSideScripts/Program.cs#L112) |[DocumentClient.CreateStoredProcedureAsync](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.client.documentclient.createstoredprocedureasync.aspx) |
| [执行存储过程](https://github.com/Azure/azure-documentdb-net/blob/d17c0ca5be739a359d105cf4112443f65ca2cb72/samples/code-samples/ServerSideScripts/Program.cs#L127) |[DocumentClient.ExecuteStoredProcedureAsync](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.client.documentclient.executestoredprocedureasync.aspx) |
| [读取存储过程的文档源](https://github.com/Azure/azure-documentdb-net/blob/d17c0ca5be739a359d105cf4112443f65ca2cb72/samples/code-samples/ServerSideScripts/Program.cs#L194) |[DocumentClient.ReadDocumentFeedAsync](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.client.documentclient.readdocumentfeedasync.aspx) |
| [使用 Order by 创建存储过程](https://github.com/Azure/azure-documentdb-net/blob/d17c0ca5be739a359d105cf4112443f65ca2cb72/samples/code-samples/ServerSideScripts/Program.cs#L219) |[DocumentClient.CreateStoredProcedureAsync](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.client.documentclient.createstoredprocedureasync.aspx) |
| [创建预触发器](https://github.com/Azure/azure-documentdb-net/blob/d17c0ca5be739a359d105cf4112443f65ca2cb72/samples/code-samples/ServerSideScripts/Program.cs#L264) |[DocumentClient.CreateTriggerAsync](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.client.documentclient.createtriggerasync.aspx) |
| [创建后触发器](https://github.com/Azure/azure-documentdb-net/blob/d17c0ca5be739a359d105cf4112443f65ca2cb72/samples/code-samples/ServerSideScripts/Program.cs#L329) |[DocumentClient.CreateTriggerAsync](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.client.documentclient.createtriggerasync.aspx) |
| [创建用户定义函数 \(UDF\)](https://github.com/Azure/azure-documentdb-net/blob/d17c0ca5be739a359d105cf4112443f65ca2cb72/samples/code-samples/ServerSideScripts/Program.cs#L389) |[DocumentClient.CreateUserDefinedFunctionAsync](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.client.documentclient.createuserdefinedfunctionasync.aspx) |

有关服务器端编程的详细信息，请参阅 [DocumentDB 服务器端编程：存储过程、数据库触发器和 UDF](/documentation/articles/documentdb-programming/)。

## 用户管理示例
用户管理文件 [azure-documentdb-dotnet/samples/code-samples/UserManagement/Program.cs](https://github.com/Azure/azure-documentdb-dotnet/blob/master/samples/code-samples/UserManagement/Program.cs) 演示如何执行以下任务。

| 任务 | API 参考 |
| --- | --- |
| [创建用户](https://github.com/Azure/azure-documentdb-net/blob/d17c0ca5be739a359d105cf4112443f65ca2cb72/samples/code-samples/UserManagement/Program.cs#L81) |[DocumentClient.CreateUserAsync](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.client.documentclient.createuserasync.aspx) |
| [对集合或文档设置权限](https://github.com/Azure/azure-documentdb-net/blob/d17c0ca5be739a359d105cf4112443f65ca2cb72/samples/code-samples/UserManagement/Program.cs#L85) |[DocumentClient.CreatePermissionAsync](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.client.documentclient.createpermissionasync.aspx) |
| [获取用户权限列表](https://github.com/Azure/azure-documentdb-net/blob/d17c0ca5be739a359d105cf4112443f65ca2cb72/samples/code-samples/UserManagement/Program.cs#L218) |[DocumentClient.ReadUserAsync](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.client.documentclient.readuserasync.aspx)<br>[DocumentClient.ReadPermissionFeedAsync](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.client.documentclient.readpermissionfeedasync.aspx) |

<!---HONumber=Mooncake_0220_2017-->
<!--Update_Description: wording update-->