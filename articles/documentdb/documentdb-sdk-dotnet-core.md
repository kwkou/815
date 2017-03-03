<properties
    pageTitle="DocumentDB .NET Core API 和 SDK | Azure"
    description="了解有关 .NET Core API 和 SDK 的全部信息，包括发布日期、停用日期和 DocumentDB.NET Core SDK 各版本之间的更改。"
    services="documentdb"
    documentationcenter=".net"
    author="rnagpal"
    manager="jhubbard"
    editor="cgronlun" />
<tags
    ms.assetid="f899b314-26ac-4ddb-86b2-bfdf05c2abf2"
    ms.service="documentdb"
    ms.workload="data-services"
    ms.tgt_pltfrm="na"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.date="01/12/2017"
    wacn.date="02/27/2017"
    ms.author="rnagpal" />

# DocumentDB API 和 SDK
> [AZURE.SELECTOR]
- [.NET](/documentation/articles/documentdb-sdk-dotnet/)
- [.NET Core](/documentation/articles/documentdb-sdk-dotnet-core/)
- [Node.js](/documentation/articles/documentdb-sdk-node/)
- [Java](/documentation/articles/documentdb-sdk-java/)
- [Python](/documentation/articles/documentdb-sdk-python/)
- [REST](https://docs.microsoft.com/zh-cn/rest/api/documentdb/)
- [REST 资源提供程序](https://docs.microsoft.com/rest/api/documentdbresourceprovider/)
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

## 发行说明

DocumentDB .NET Core SDK 与最新版 [DocumentDB.NET SDK](/documentation/articles/documentdb-sdk-dotnet/) 的功能相同。

> [AZURE.NOTE] 
DocumentDB .NET Core SDK 目前与通用 Windows 平台 \(UWP\) 应用不兼容。如需支持 UWP 应用的 .NET Core SDK 预览版，请发送电子邮件至 [askdocdb@microsoft.com](mailto:askdocdb@microsoft.com)。

### <a name="1.0.0"/>[1\.0.0](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB.Core/1.0.0)

借助 DocumentDB .NET Core SDK，可构建能够在 Windows、Mac 和 Linux 上运行的快速、跨平台 [ASP.NET Core](https://www.asp.net/core) 和 [.NET Core](https://www.microsoft.com/net/core#windows) 应用。最新版本的 DocumentDB.NET Core SDK 与 [Xamarin](https://www.xamarin.com) 完全兼容，可用于构建面向 iOS、Android 和 Mono \(Linux\) 的应用程序。

### <a name="0.1.0-preview"/>[0\.1.0-preview](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB.Core/0.1.0-preview)

借助 DocumentDB .NET Core 预览版 SDK，可生成能在 Windows、Mac 和 Linux 上运行的快速、跨平台 [ASP.NET Core](https://www.asp.net/core) 和 [.NET Core](https://www.microsoft.com/net/core#windows) 应用。

DocumentDB .NET Core 预览版 SDK 与最新版 [DocumentDB.NET SDK](/documentation/articles/documentdb-sdk-dotnet/) 功能相同，并支持以下内容：
- 所有[连接模式](/documentation/articles/documentdb-performance-tips/#networking/)：网关模式、Direct TCP 和 Direct HTTPs。
- 所有[一致性级别](/documentation/articles/documentdb-consistency-levels/)：强烈一致性、会话一致性、有限过期一致性和最终一致性。
- [已分区集合](/documentation/articles/documentdb-partition-data/)。
- [多区域数据库帐户和异地复制](/documentation/articles/documentdb-distribute-data-globally/)。

如果遇到与此 SDK 相关的问题，请在 [StackOverflow](http://stackoverflow.com/questions/tagged/azure-documentdb) 中发贴，或者在 [github 存储库](https://github.com/Azure/azure-documentdb-dotnet/issues)中提问。

## 发布和停用日期

| 版本 | 发布日期 | 停用日期 |
| --- | --- | --- |
| [1\.0.0](#1.0.0) \|2016 年 12 月 21 日 \|--- \| 
| [0\.1.0-preview](#0.1.0-preview) \|2016 年 11 月 15 日 \|2016 年 12 月 31 日 \|

## 另请参阅
若要了解有关 DocumentDB 的详细信息，请参阅 [Azure DocumentDB](/home/features/documentdb/) 服务页。

<!---HONumber=Mooncake_0220_2017-->
<!---Update_Description: wording update -->