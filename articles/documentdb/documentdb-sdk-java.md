<properties
    pageTitle="Azure DocumentDB Java API、SDK 和资源 | Microsoft 文档"
    description="了解有关 Java API 和 SDK 的全部信息，包括发布日期、停用日期和 DocumentDB Java SDK 各版本之间所做的更改。"
    services="documentdb"
    documentationcenter="java"
    author="rnagpal"
    manager="jhubbard"
    editor="cgronlun" />
<tags
    ms.assetid="7861cadf-2a05-471a-9925-0fec0599351b"
    ms.service="documentdb"
    ms.workload="data-services"
    ms.tgt_pltfrm="na"
    ms.devlang="java"
    ms.topic="article"
    ms.date="05/10/2017"
    ms.author="khdang"
    ms.custom="H1Hack27Feb2017"
    wacn.date="05/31/2017"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="4a18b6116e37e365e2d4c4e2d144d7588310292e"
    ms.openlocfilehash="88844039e47ff24b141c2dd378ac238f9a2167e2"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/19/2017" />

# <a name="documentdb-java-sdk-release-notes-and-resources"></a>DocumentDB Java SDK：发行说明和资源
> [AZURE.SELECTOR]
- [.NET](/documentation/articles/documentdb-sdk-dotnet/)
- [.NET Core](/documentation/articles/documentdb-sdk-dotnet-core/)
- [Node.js](/documentation/articles/documentdb-sdk-node/)
- [Java](/documentation/articles/documentdb-sdk-java/)
- [Python](/documentation/articles/documentdb-sdk-python/)
- [REST](https://docs.microsoft.com/zh-cn/rest/api/documentdb/)
- [REST 资源提供程序](https://docs.microsoft.com/zh-cn/rest/api/documentdbresourceprovider/)
- [SQL](https://msdn.microsoft.com/zh-cn/library/azure/dn782250.aspx)

<table>


<tr><td>SDK 下载</td><td><a href="http://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22com.microsoft.azure%22%20AND%20a%3A%22azure-documentdb%22">Maven</a></td></tr>

<tr><td>API 文档</td><td><a href="http://azure.github.io/azure-documentdb-java/">Java API 参考文档</a></td></tr>

<tr><td>参与 SDK</td><td><a href="https://github.com/Azure/azure-documentdb-java/">GitHub</a></td></tr>

<tr><td>入门</td><td><a href="/documentation/articles/documentdb-java-get-started/">Java SDK 入门</a></td></tr>

<tr><td>Web 应用教程</td><td><a href="/documentation/articles/documentdb-java-application/">使用 DocumentDB 开发 Web 应用程序</a></td></tr>

<tr><td>当前受支持的运行时</td><td><a href="http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html">JDK 7</a></td></tr>
</table>

## <a name="release-notes"></a>发行说明

### <a name="a-name11101110"></a><a name="1.11.0"></a>1.11.0
- 添加了对每分钟请求单位数 (RU/m) 功能的支持。
- 添加了对称为“ConsistentPrefix”的新一致性级别的支持。
- 修复了以会话模式读取集合时的 bug。

### <a name="a-name11001100"></a><a name="1.10.0"></a>1.10.0
- 启用了对吞吐量低至 2,500 RU/秒并且缩放增量为 100 RU/秒的分区集合的支持。
- 修复了本机程序集中的 bug，该 bug 在某些查询中可能会导致 NullRef 异常。

### <a name="a-name196196"></a><a name="1.9.6"></a>1.9.6
- 修复了查询引擎配置中可能会导致网关模式下查询异常的 Bug。
- 修复了会话容器中的一些 Bug，这些 Bug 可能会在创建集合后立即导致“找不到所有者资源”请求异常。

### <a name="a-name195195"></a><a name="1.9.5"></a>1.9.5
- 添加了对聚合查询（COUNT、MIN、MAX、SUM、AVG）的支持。 请参阅[聚合支持](/documentation/articles/documentdb-sql-query/#Aggregates/)。
- 添加了对更改源的支持。
- 通过 RequestOptions.setPopulateQuotaInfo 添加了对集合配额信息的支持。
- 通过 RequestOptions.setScriptLoggingEnabled 添加了对存储过程脚本日志记录的支持。
- 修复了一个 Bug，该 Bug 导致 DirectHttps 模式的查询在遇到限制失败时可能会挂起。
- 修复了会话一致性模式中的一个 Bug。
- 修复了一个 Bug，该 Bug 可能在请求率很高时导致 HttpContext 中出现 NullReferenceException。
- 改进了 DirectHttps 模式的性能。

### <a name="a-name194194"></a><a name="1.9.4"></a>1.9.4
- 使用 ConnectionPolicy.setProxy() API 添加了基于简单客户端实例的代理支持。
- 添加了 DocumentClient.close() API 以正确关闭 DocumentClient 实例。
- 通过从本机程序集（而非网关）派生查询计划，提高直接连接模式下的查询性能。
- 设置 FAIL_ON_UNKNOWN_PROPERTIES = false，使用户无需在其 POJO 中定义 JsonIgnoreProperties。
- 重构了日志记录以使用 SLF4J。
- 修复了一致性读取器中的其他几个 Bug。

### <a name="a-name193193"></a><a name="1.9.3"></a>1.9.3
- 修复了连接管理中的 bug，防止直接连接模式下的连接泄漏。
- 修复了 TOP 查询中可能会引发 NullReferenece 异常的 Bug。
- 通过减少调用内部缓存的网络数提高了性能。
- 在 DocumentClientException 中添加了状态代码、ActivityID 和请求 URI，以更好地进行故障排除。

### <a name="a-name192192"></a><a name="1.9.2"></a>1.9.2
- 修复了连接管理中的问题，实现了更好的稳定性。

### <a name="a-name191191"></a><a name="1.9.1"></a>1.9.1
- 添加了对 BoundedStaleness 一致性级别的支持。
- 添加了对分区集合的 CRUD 操作的直接连接支持。
- 修复了使用 SQL 查询数据库的一个 bug。
- 修复了会话缓存中会话令牌设置可能不正确的 bug。

### <a name="a-name190190"></a><a name="1.9.0"></a>1.9.0
- 添加了对跨分区并行查询的支持。
- 添加了对分区集合的 TOP/ORDER BY 查询支持。
- 添加了非常一致性支持。
- 添加了使用直接连接时对基于名称的请求的支持。
- 修复了 bug，使 ActivityId 在所有请求重试中保持一致。
- 修复了在重新创建同名集合时与会话缓存相关的 bug。
- 为地域隔离的空间查询指定集合索引策略时增加了多边形和 LineString 数据类型。
- 解决 Java 文档中的 Java 1.8 的问题。

### <a name="a-name181181"></a><a name="1.8.1"></a>1.8.1
- 修复了 PartitionKeyDefinitionMap 中的一个 bug，以便缓存单个分区集合，而不进行额外的提取分区键的请求。
- 修复了一个 bug，以便在提供不正确的分区键值时不重试。

### <a name="a-name180180"></a><a name="1.8.0"></a>1.8.0
- 添加了对多区域数据库帐户的支持。
- 添加了对自动重试限制请求的支持，并提供了选项用于自定义最大重试次数和最大重试等待时间。  请参阅 RetryOptions 和 ConnectionPolicy.getRetryOptions()。
- 弃用了基于 IPartitionResolver 的自定义分区代码。 请使用分区集合来提高存储和吞吐量。

### <a name="a-name171171"></a><a name="1.7.1"></a>1.7.1
- 对限制添加了重试策略支持。  

### <a name="a-name170170"></a><a name="1.7.0"></a>1.7.0
- 对文档添加了生存时间 (TTL) 支持。

### <a name="a-name160160"></a><a name="1.6.0"></a>1.6.0
- 实现了[分区集合](/documentation/articles/documentdb-partition-data/)和[用户定义的性能级别](/documentation/articles/documentdb-performance-levels/)。

### <a name="a-name151151"></a><a name="1.5.1"></a>1.5.1
- 修复了 HashPartitionResolver 中的 Bug 以生成 little-endian 格式的哈希值，以便与其他 SDK 保持一致。

### <a name="a-name150150"></a><a name="1.5.0"></a>1.5.0
- 添加哈希和范围分区冲突解决程序以协助跨多个分区对应用程序进行分片。

### <a name="a-name140140"></a><a name="1.4.0"></a>1.4.0
- 实现 Upsert。 添加了新的 upsertXXX 方法以支持 Upsert 功能。
- 实现基于 ID 的路由。 无公共 API 更改，全部均为内部更改。

### <a name="a-name130130"></a><a name="1.3.0"></a>1.3.0
- 跳过了发布以使版本号与其他 SDK 符合

### <a name="a-name120120"></a><a name="1.2.0"></a>1.2.0
- 支持地理空间索引
- 验证所有资源的 ID 属性。 资源的 ID 不能包含 ？、/、#、\, 字符或以空格结尾。
- 将新标头“索引转换进度”添加到 ResourceResponse。

### <a name="a-name110110"></a><a name="1.1.0"></a>1.1.0
- 实现 V2 索引策略

### <a name="a-name100100"></a><a name="1.0.0"></a>1.0.0
- GA SDK

## <a name="release--retirement-dates"></a>发布和停用日期
Microsoft 至少会在停用 SDK 的 **12 个月**之前发出通知，以便顺利转换到更新的/受支持的版本。

新特性和功能以及优化仅添加到当前 SDK，因此建议你始终尽早升级到最新 SDK 版本。

使用已停用的 SDK 对 DocumentDB 发出的任何请求都将被服务拒绝。

> [AZURE.WARNING]
> Azure DocumentDB SDK for Java 在 **1.0.0** 版之前的所有版本都将在 **2016 年 2 月 29 日**停用。
> 
> 

<br/>

| 版本 | 发布日期 | 停用日期 |
| --- | --- | --- |
| [1.11.0](#1.11.0) |2017 年 5 月 10 日 |--- |
| [1.10.0](#1.10.0) |2017 年 3 月 11 日 |--- |
| [1.9.6](#1.9.6) |2017 年 2 月 21 日 |--- |
| [1.9.5](#1.9.5) |2017 年 1 月 31 日 |--- |
| [1.9.4](#1.9.4) |2016 年 11 月 24 日 |--- |
| [1.9.3](#1.9.3) |2016 年 10 月 30 日 |--- |
| [1.9.2](#1.9.2) |2016 年 10 月 28 日 |--- |
| [1.9.1](#1.9.1) |2016 年 10 月 26 日 |--- |
| [1.9.0](#1.9.0) |2016 年 10 月 3 日 |--- |
| [1.8.1](#1.8.1) |2016 年 6 月 30 日 |--- |
| [1.8.0](#1.8.0) |2016 年 6 月 14 日 |--- |
| [1.7.1](#1.7.1) |2016 年 4 月 30 日 |--- |
| [1.7.0](#1.7.0) |2016 年 4 月 27 日 |--- |
| [1.6.0](#1.6.0) |2016 年 3 月 29 日 |--- |
| [1.5.1](#1.5.1) |2015 年 12 月 31 日 |--- |
| [1.5.0](#1.5.0) |2015 年 12 月 4 日 |--- |
| [1.4.0](#1.4.0) |2015 年 10 月 5 日 |--- |
| [1.3.0](#1.3.0) |2015 年 10 月 5 日 |--- |
| [1.2.0](#1.2.0) |2015 年 8 月 5 日 |--- |
| [1.1.0](#1.1.0) |2015 年 7 月 9 日 |--- |
| 1.0.1  |2015 年 5 月 12 日 |--- |
| [1.0.0](#1.0.0) |2015 年 4 月 7 日 |--- |
| 0.9.5-prelease |2015 年 3 月 9 日 |2016 年 2 月 29 日 |
| 0.9.4-prelease |2015 年 2 月 17 日 |2016 年 2 月 29 日 |
| 0.9.3-prelease |2015 年 1 月 13 日 |2016 年 2 月 29 日 |
| 0.9.2-prelease |2014 年 12 月 19 日 |2016 年 2 月 29 日 |
| 0.9.1-prelease |2014 年 12 月 19 日 |2016 年 2 月 29 日 |
| 0.9.0-prelease |2014 年 12 月 10 日 |2016 年 2 月 29 日 |

## <a name="faq"></a>常见问题
[AZURE.INCLUDE [documentdb-sdk-faq](../../includes/documentdb-sdk-faq.md)]

## <a name="see-also"></a>另请参阅
若要了解有关 DocumentDB 的详细信息，请参阅 [DocumentDB](/home/features/documentdb/) 服务页。

<!---Update_Description: wording update -->