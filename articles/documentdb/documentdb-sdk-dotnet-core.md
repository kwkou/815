<properties
    pageTitle="Azure DocumentDB .NET Core API、SDK 和资源 | Azure"
    description="了解有关 .NET Core API 和 SDK 的全部信息，包括发布日期、停用日期和 DocumentDB.NET Core SDK 各版本之间的更改。"
    services="documentdb"
    documentationcenter=".net"
    author="rnagpal"
    manager="jhubbard"
    editor="cgronlun"
    translationtype="Human Translation" />
    
<tags
    ms.assetid="f899b314-26ac-4ddb-86b2-bfdf05c2abf2"
    ms.service="documentdb"
    ms.workload="data-services"
    ms.tgt_pltfrm="na"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.date="03/20/2017"
    wacn.date="04/17/2017"
    ms.author="rnagpal"
    ms.custom="H1Hack27Feb2017"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="f2ac24f9d43733c3565e4590aa536a63f214b1a9"
    ms.lasthandoff="04/07/2017" />

# <a name="documentdb-net-core-sdk-release-notes-and-resources"></a>DocumentDB .NET Core SDK：发行说明和资源
> [AZURE.SELECTOR]
- [.NET](/documentation/articles/documentdb-sdk-dotnet/)
- [.NET Core](/documentation/articles/documentdb-sdk-dotnet-core/)
- [Node.js](/documentation/articles/documentdb-sdk-node/)
- [Java](/documentation/articles/documentdb-sdk-java/)
- [Python](/documentation/articles/documentdb-sdk-python/)
- [REST](https://docs.microsoft.com/zh-cn/rest/api/documentdb/)
- [REST 资源提供程序](https://docs.microsoft.com/zh-cn/rest/api/documentdbresourceprovider/)
- [SQL](https://msdn.microsoft.com/zh-cn/library/azure/dn782250.aspx)

## DocumentDB .NET Core API 和 SDK
<table>  


<tr><td>**SDK 下载**</td><td><a href="https://www.nuget.org/packages/Microsoft.Azure.DocumentDB.Core/">NuGet</a></td></tr>

<tr><td>**API 文档**</td><td><a href="https://msdn.microsoft.com/zh-cn/library/azure/dn948556.aspx">.NET API 参考文档</a></td></tr>

<tr><td>**示例**</td><td><a href="/documentation/articles/documentdb-dotnet-samples/">.NET 代码示例</a></td></tr>

<tr><td>**入门**</td><td><a href="/documentation/articles/documentdb-dotnetcore-get-started/">DocumentDB .NET Core SDK 入门</a></td></tr>

<tr><td>**Web 应用教程**</td><td><a href="/documentation/articles/documentdb-dotnet-application/">使用 DocumentDB 开发 Web 应用程序</a></td></tr>

<tr><td>**当前受支持的框架**</td><td><a href="https://www.nuget.org/packages/NETStandard.Library">.NET Standard 1.6</a></td></tr>
</table>

## <a name="release-notes"></a>发行说明

DocumentDB .NET Core SDK 具有与最新版 [DocumentDB.NET SDK](/documentation/articles/documentdb-sdk-dotnet/) 相同的功能。

> [AZURE.NOTE] 
> DocumentDB .NET Core SDK 与通用 Windows 平台 (UWP) 应用尚不兼容。 如果不支持 UWP 应用的 .NET Core SDK 感兴趣，请向 [askdocdb@microsoft.com](mailto:askdocdb@microsoft.com) 发送电子邮件。

### <a name="a-name112112httpswwwnugetorgpackagesmicrosoftazuredocumentdbcore112"></a><a name="1.1.2"/>[1.1.2](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB.Core/1.1.2)

- 修复偶尔导致 WebException 的问题：无法解析远程名称。
- 通过向 ReadDocumentAsync API 添加新重载，添加了对直接读取类型化文档的支持。

### <a name="a-name111111httpswwwnugetorgpackagesmicrosoftazuredocumentdbcore111"></a><a name="1.1.1"/>[1.1.1](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB.Core/1.1.1)

- 添加了对聚合查询（COUNT、MIN、MAX、SUM 和 AVG）的 LINQ 支持。
- 修复了由于使用了事件处理程序而导致的 ConnectionPolicy 对象的内存泄漏问题。
- 修复了使用 ETag 时 UpsertAttachmentAsync 不正常工作的问题。
- 修复了对字符串字段进行排序时跨分区按查询条件排序不正常工作的问题。

### <a name="a-name110110httpswwwnugetorgpackagesmicrosoftazuredocumentdbcore110"></a><a name="1.1.0"/>[1.1.0](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB.Core/1.1.0)

- 添加了对聚合查询（COUNT、MIN、MAX、SUM、AVG）的支持。 请参阅[聚合支持](/documentation/articles/documentdb-sql-query/#Aggregates/)。
- 将分区集合上的最小吞吐量从 10,100 RU/s 降低到 2500 RU/s。

### <a name="a-name100100httpswwwnugetorgpackagesmicrosoftazuredocumentdbcore100"></a><a name="1.0.0"/>[1.0.0](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB.Core/1.0.0)

通过 DocumentDB .NET Core SDK，可构建能在 Windows、Mac 和 Linux 上快速运行的跨平台 [ASP.NET Core](https://www.asp.net/core) 和 [.NET Core](https://www.microsoft.com/net/core#windows) 应用。 DocumentDB .NET Core SDK 的最新版本完全兼容 [Xamarin](https://www.xamarin.com)，可用于生成面向 iOS、Android 和 Mono (Linux) 的应用程序。  

### <a name="a-name010-preview010-previewhttpswwwnugetorgpackagesmicrosoftazuredocumentdbcore010-preview"></a><a name="0.1.0-preview"/>[0.1.0-preview](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB.Core/0.1.0-preview)

通过 DocumentDB .NET Core 预览版 SDK，可构建能在 Windows、Mac 和 Linux 上快速运行的跨平台 [ASP.NET Core](https://www.asp.net/core) 和 [.NET Core](https://www.microsoft.com/net/core#windows) 应用。

DocumentDB .NET Core 预览版 SDK 与最新版 [DocumentDB.NET SDK](/documentation/articles/documentdb-sdk-dotnet/) 功能相同，并支持以下内容：
- 所有[连接模式](/documentation/articles/documentdb-performance-tips/#networking/)：网关模式、Direct TCP 和 Direct HTTP。 
- 所有[一致性级别](/documentation/articles/documentdb-consistency-levels/)：强一致性、会话一致性、有限过期一致性和最终一致性。
- [已分区集合](/documentation/articles/documentdb-partition-data/)。 
- [多区域数据库帐户和异地复制](/documentation/articles/documentdb-distribute-data-globally/)。

若有与此 SDK 相关的问题，请发布到 [StackOverflow](http://stackoverflow.com/questions/tagged/azure-documentdb)，或在 [github 存储库](https://github.com/Azure/azure-documentdb-dotnet/issues)中提出问题。 

## <a name="release--retirement-dates"></a>发布和停用日期

| 版本 | 发布日期 | 停用日期 |
| --- | --- | --- |
| [1.1.2](#1.1.2) |2017 年 3 月 20 日 |--- |
| [1.1.1](#1.1.1) |2017 年 3 月 14 日 |--- |
| [1.1.0](#1.1.0) |2017 年 2 月 16 日 |--- |
| [1.0.0](#1.0.0) |2016 年 12 月 21 日 |--- |
| [0.1.0-preview](#0.1.0-preview) |2016 年 11 月 15 日 |2016 年 12 月 31 日 |

## <a name="see-also"></a>另请参阅
若要了解有关 DocumentDB 的详细信息，请参阅 [Azure DocumentDB](/home/features/documentdb/) 服务页。

<!---Update_Description: wording update -->