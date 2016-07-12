<properties 
	pageTitle="DocumentDB .NET SDK | Azure" 
	description="了解有关 .NET SDK 的全部信息，包括发布日期、停用日期和 DocumentDB.NET SDK 各版本之间的更改。" 
	services="documentdb" 
	documentationCenter=".net" 
	authors="aliuy" 
	manager="jhubbard" 
	editor="cgronlun"/>

<tags 
	ms.service="documentdb" 
	ms.date="06/14/2016" 
	wacn.date="06/29/2016"/>

# DocumentDB SDK

> [AZURE.SELECTOR]
- [.NET SDK](/documentation/articles/documentdb-sdk-dotnet/)
- [Node.js SDK](/documentation/articles/documentdb-sdk-node/)
- [Java SDK](/documentation/articles/documentdb-sdk-java/)
- [Python SDK](/documentation/articles/documentdb-sdk-python/)

##DocumentDB .NET SDK

<table>
<tr><td>**下载**</td><td>[NuGet](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB/)</td></tr>
<tr><td>**文档**</td><td>[.NET SDK 参考文档](https://msdn.microsoft.com/library/azure/dn948556.aspx)</td></tr>
<tr><td>**示例**</td><td>[.NET 代码示例](https://github.com/Azure/azure-documentdb-dotnet/tree/master/samples/code-samples)</td></tr>
<tr><td>**入门**</td><td>[DocumentDB .NET SDK 入门](/documentation/articles/documentdb-get-started/)</td></tr>
<tr><td>**当前受支持的框架**</td><td>[Microsoft .NET Framework 4.5](https://www.microsoft.com/download/details.aspx?id=30653)</td></tr>
</table></br>

## 发行说明

### <a name="1.8.0"/>[1\.8.0](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB/1.8.0)
  - 添加了对多区域数据库帐户的支持。
  - 添加了对重试限制请求的支持。用户可以通过配置 ConnectionPolicy.RetryOptions 属性来自定义重试次数和最长等待时间。
  - 添加新的 IDocumentClient 接口，用于定义所有 DocumenClient 属性和方法的签名。在做出此项更改的同时，已将用于创建 IQueryable 和 IOrderedQueryable 的扩展方法更改为 DocumentClient 类本身的方法。
  - 添加了配置选项用于设置给定 DocumentDB 终结点 URI 的 ServicePoint.ConnectionLimit。使用 ConnectionPolicy.MaxConnectionLimit 可以更改默认值（设置为 50）。
  - 已弃用 IPartitionResolver 及其实现。对 IPartitionResolver 的支持现已过时。建议使用分区集合来提高存储和吞吐量。


### <a name="1.7.1"/>[1\.7.1](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB/1.7.1)
  - 向采用 RequestOptions 作为参数的基于 Uri 的 ExecuteStoredProcedureAsync 方法添加了重载。
  
### <a name="1.7.0"/>[1\.7.0](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB/1.7.0)
  - 对文档添加了生存时间 (TTL) 支持。

### <a name="1.6.3"/>[1\.6.3](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB/1.6.3)
  - 修复了用于将其打包为 Azure 云服务解决方案的一部分的 .NET SDK 的 Nuget 包中的 Bug。
  
### <a name="1.6.2"/>[1\.6.2](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB/1.6.2)
  - 实现了[分区集合](/documentation/articles/documentdb-partition-data/)和[用户定义的性能级别](/documentation/articles/documentdb-performance-levels/)。 

### <a name="1.5.3"/>[1\.5.3](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB/1.5.3)
  - **[已修复]** 查询 DocumentDB 终结点引发：System.Net.Http.HttpRequestException：将内容复制到流时出错。

### <a name="1.5.2"/>[1\.5.2](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB/1.5.2)
  - 扩展的 LINQ 支持，包括用于分页、条件表达式和范围比较的新运算符。
    - Take 运算符在 LINQ 中启用 SELECT TOP 行为
    - CompareTo 运算符使能够进行字符串范围比较
    - Conditional (?) 和 coalesce 运算符 (??)
  - **[已修复]** 合并模型投影与 linq 查询中的 Where-In 时出现 ArgumentOutOfRangeException。[#81](https://github.com/Azure/azure-documentdb-dotnet/issues/81)

### <a name="1.5.1"/>[1\.5.1](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB/1.5.1)
 - **[已修复]** 如果 Select 不是最后一个表达式，则 LINQ 提供程序假定没有投影，并且错误地生成 SELECT *。[#58](https://github.com/Azure/azure-documentdb-dotnet/issues/58)

### <a name="1.5.0"/>[1\.5.0](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB/1.5.0)
 - 实现的 Upsert，已添加 UpsertXXXAsync 方法
 - 所有请求的性能改进
 - LINQ 提供程序支持字符串的条件、联合和 CompareTo 方法
 - **[已修复]** LINQ 提供程序 --> 在列表上实现 Contains 方法以生成与 IEnumerable 和 Array 上相同的 SQL
 - **[已修复]** BackoffRetryUtility 再次使用相同的 HttpRequestMessage 而不是在重试时创建一个新的
 - **[已过时]** UriFactory.CreateCollection --> 现在应使用 UriFactory.CreateDocumentCollection
 
### <a name="1.4.1"/>[1\.4.1](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB/1.4.1)
 - **[已修复]** 使用非 en 区域性信息（如 NL-NL 等）时出现本地化问题。 
 
### <a name="1.4.0"/>[1\.4.0](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB/1.4.0)
  - 基于 ID 的路由
    - 用于协助构建基于 ID 的资源链接的新 UriFactory 帮助器
    - DocumentClient 上的新重载以采用 URI
  - 已在用于地理空间的 LINQ 中添加了 IsValid() 和 IsValidDetailed()
  - 增强的LINQ 提供程序支持
    - **数学** - Abs、Acos、Asin、Atan、Ceiling、Cos、Exp、Floor、Log、Log10、Pow、Round、Sign、Sin、Sqrt、Tan、Truncate
    - **字符串** - Concat、Contains、EndsWith、IndexOf、Count、ToLower、TrimStart、Replace、Reverse、TrimEnd、StartsWith、SubString、ToUpper
    - **数组** - Concat、Contains、Count
    - **IN** 运算符

### <a name="1.3.0"/>[1\.3.0](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB/1.3.0)
  - 添加了对修改索引策略的支持
    - DocumentClient 中的新 ReplaceDocumentCollectionAsync 方法
    - ResourceResponse 中的新 IndexTransformationProgress 属性<T>，用于跟踪索引策略更改的百分比进度
    - DocumentCollection.IndexingPolicy 现在是可变的
  - 添加了对空间索引和查询的支持
    - 用于序列化/反序列化空间类型（如点和多边形）的新 Microsoft.Azure.Documents.Spatial 命名空间
    - 用于索引存储在 DocumentDB 中的 GeoJSON 数据的新 SpatialIndex 类
  - **[已修复]**：从 linq 表达式生成的不正确的 SQL 查询 [#38](https://github.com/Azure/azure-documentdb-net/issues/38)

### <a name="1.2.0"/>[1\.2.0](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB/1.2.0)
- 对 Newtonsoft.Json v5.0.7 的依赖关系 
- 更改为支持 Order By
  - LINQ 提供程序支持 OrderBy() 或 OrderByDescending()
  - 支持 Order By 的 IndexingPolicy 
  
		**注意：可能非常重大的更改** 
  
    	如果你的现有代码使用自定义索引策略预配集合，那么你的现有代码将需要更新为支持新的 IndexingPolicy 类。如果你没有任何自定义索引策略，此更改不会影响你。

### <a name="1.1.0"/>[1\.1.0](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB/1.1.0)
- 通过使用新的 HashPartitionResolver 和 RangePartitionResolver 类以及 IPartitionResolver 支持对数据进行分区
- DataContract 序列化
- LINQ 提供程序中的 Guid 支持
- LINQ 中的 UDF 支持

### <a name="1.0.0"/>[1\.0.0](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB/1.0.0)
- GA SDK

> [AZURE.NOTE]
预览版和 GA 版之间的 NuGet 程序包名称有所更改。从 **Microsoft.Azure.Documents.Client** 移动到了 **Microsoft.Azure.DocumentDB** <br/>


### <a name="0.9.x-preview"/>[0\.9.x-preview](https://www.nuget.org/packages/Microsoft.Azure.Documents.Client)
- 预览版 SDK[已过时]

## 发布和停用日期
Microsoft 将在停用一款 SDK 之前至少 **12 个月**发出通知，以便顺利过渡到更新的/受支持的版本。

新特性和功能以及优化仅添加到当前 SDK，因此建议你始终尽早升级到最新 SDK 版本。

使用已停用的 SDK 对 DocumentDB 发出的任何请求都将被服务拒绝。

> [AZURE.WARNING]
**1.0.0** 版之前的 Azure DocumentDB SDK for .NET 的所有版本都将在 **2016 年 2 月 29 日**停用。
 
<br/>
 
| 版本 | 发布日期 | 停用日期 
| ---	  | ---	         | ---
| [1\.8.0](#1.8.0) | 2016 年 6 月 14 日 |---
| [1\.7.1](#1.7.1) | 2016 年 5 月 6 日 |---
| [1\.7.0](#1.7.0) | 2016 年 4 月 26 日 |---
| [1\.6.3](#1.6.3) |2016 年 4 月 8 日 |---
| [1\.6.2](#1.6.2) |2016 年 3 月 29 日 |---
| [1\.5.3](#1.5.3) | 2016 年 2 月 19 日 |---
| [1\.5.2](#1.5.2) | 2015 年 12 月 14 日 |---
| [1\.5.1](#1.5.1) | 2015 年 11 月 23 日 |---
| [1\.5.0](#1.5.0) | 2015 年 10 月 5 日 |---
| [1\.4.1](#1.4.1) | 2015 年 8 月 25 日 |---
| [1\.4.0](#1.4.0) | 2015 年 8 月 13 日 |---
| [1\.3.0](#1.3.0) | 2015 年 8 月 5 日 |---
| [1\.2.0](#1.2.0) | 2015 年 7 月 6 日 |---
| [1\.1.0](#1.1.0) | 2015 年 4 月 30 日 |---
| [1\.0.0](#1.0.0) | 2015 年 4 月 8 日 |---
| [0\.9.3-prelease](#0.9.x-preview) | 2015 年 3 月 12 日 | 2016 年 2 月 29 日
| [0\.9.2-prelease](#0.9.x-preview) | 2015 年 1 月 | 2016 年 2 月 29 日
| [.9.1-prelease](#0.9.x-preview) | 2014 年 10 月 13 日 | 2016 年 2 月 29 日
| [0\.9.0-prelease](#0.9.x-preview) | 2014 年 8 月 21 日 | 2016 年 2 月 29 日

## 常见问题
[AZURE.INCLUDE [documentdb sdk 常见问题](../includes/documentdb-sdk-faq.md)]

## 另请参阅

要了解有关 DocumentDB 的详细信息，请参阅 [Azure DocumentDB](https://azure.microsoft.com/services/documentdb/) 服务页。

<!---HONumber=Mooncake_0627_2016-->