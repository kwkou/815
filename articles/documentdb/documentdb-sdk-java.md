<properties
    pageTitle="DocumentDB Java API 和 SDK | Azure"
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
    ms.date="01/03/2017"
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

## DocumentDB Java API 和 SDK
<table>  


<tr><td>**SDK 下载**</td><td><a href="http://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22com.microsoft.azure%22%20AND%20a%3A%22azure-documentdb%22">Maven</a></td></tr>

<tr><td>**API 文档**</td><td><a href="http://azure.github.io/azure-documentdb-java/">Java API 参考文档</a></td></tr>

<tr><td>**参与 SDK**</td><td><a href="https://github.com/Azure/azure-documentdb-java/">GitHub</a></td></tr>

<tr><td>**入门**</td><td><a href="/documentation/articles/documentdb-java-get-started/">Java SDK 入门</a></td></tr>

<tr><td>**Web 应用教程**</td><td><a href="/documentation/articles/documentdb-java-application/">使用 DocumentDB 开发 Web 应用程序</a></td></tr>

<tr><td>**当前受支持的运行时**</td><td><a href="http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html">JDK 7</a></td></tr>
</table>

## 发行说明
### <a name="1.9.4"/>[1\.9.4](http://mvnrepository.com/artifact/com.microsoft.azure/azure-documentdb/1.9.4)
- 使用 ConnectionPolicy.setProxy\(\) API 添加了基于简单客户端实例的代理支持。
- 添加了 DocumentClient.close\(\) API 以正确关闭 DocumentClient 实例。
- 通过从本机程序集（而非网关）派生查询计划，提高直接连接模式下的查询性能。
- 设置 FAIL\_ON\_UNKNOWN\_PROPERTIES = false，使用户无需在其 POJO 中定义 JsonIgnoreProperties。
- 重构了日志记录以使用 SLF4J。
- 修复了一致性读取器中的其他几个 Bug。

### <a name="1.9.3"/>[1\.9.3](http://mvnrepository.com/artifact/com.microsoft.azure/azure-documentdb/1.9.3)
- 修复了连接管理中的 Bug，防止直接连接模式下的连接泄漏。
- 修复了 TOP 查询中可能会引发 NullReferenece 异常的 Bug。
- 通过减少调用内部缓存的网络数提高了性能。
- 在 DocumentClientException 中添加了状态代码、ActivityID 和请求 URI，以更好地进行故障排除。

### <a name="1.9.2"/>[1\.9.2](http://mvnrepository.com/artifact/com.microsoft.azure/azure-documentdb/1.9.2)
- 修复了连接管理中的问题，实现了更好的稳定性。

### <a name="1.9.1"/>[1\.9.1](http://mvnrepository.com/artifact/com.microsoft.azure/azure-documentdb/1.9.1)
- 添加了对有限过期一致性级别的支持。
- 添加了对分区集合的 CRUD 操作的直接连接支持。
- 修复了使用 SQL 查询数据库的一个 bug。
- 修复了会话令牌可能设置不正确的会话缓存中的一个 bug。

### <a name="1.9.0"/>[1\.9.0](http://mvnrepository.com/artifact/com.microsoft.azure/azure-documentdb/1.9.0)
- 添加了对跨分区并行查询的支持。
- 添加了对分区集合的 TOP/ORDER BY 查询支持。
- 添加了非常一致性支持。
- 添加了使用直接连接时对基于名称的请求的支持。
- 修复了 bug，使 ActivityId 在所有请求重试中保持一致。
- 修复了在重新创建同名集合时与会话缓存相关的 bug。
- 为地域隔离的空间查询指定集合索引策略时增加了多边形和 LineString 数据类型。
- 修复了 Java 1.8 文档中的问题。

### <a name="1.8.1"/>[1\.8.1](http://mvnrepository.com/artifact/com.microsoft.azure/azure-documentdb/1.8.1)
- 修复了 PartitionKeyDefinitionMap 中的一个 bug，以便缓存单个分区集合，而不进行额外的提取分区键的请求。
- 修复了一个 bug，以便在提供不正确的分区键值时不重试。

### <a name="1.8.0"/>[1\.8.0](http://mvnrepository.com/artifact/com.microsoft.azure/azure-documentdb/1.8.0)
- 添加了对多区域数据库帐户的支持。
- 添加了对自动重试限制请求的支持，并提供了选项用于自定义最大重试次数和最大重试等待时间。请参阅 RetryOptions 和 ConnectionPolicy.getRetryOptions\(\)。
- 弃用了基于 IPartitionResolver 的自定义分区代码。请使用分区集合来提高存储和吞吐量。

### <a name="1.7.1"/>[1\.7.1](http://mvnrepository.com/artifact/com.microsoft.azure/azure-documentdb/1.7.1)
- 添加了对限制的重试策略支持。

### <a name="1.7.0"/>[1\.7.0](http://mvnrepository.com/artifact/com.microsoft.azure/azure-documentdb/1.7.0)
- 添加了对文档的生存时间 \(TTL\) 支持。

### <a name="1.6.0"/>[1\.6.0](http://mvnrepository.com/artifact/com.microsoft.azure/azure-documentdb/1.6.0)
- 实现了[分区集合](/documentation/articles/documentdb-partition-data/)和[用户定义的性能级别](/documentation/articles/documentdb-performance-levels/)。

### <a name="1.5.1"/>[1\.5.1](http://mvnrepository.com/artifact/com.microsoft.azure/azure-documentdb/1.5.1)
- 修复了 HashPartitionResolver 中的 Bug 以生成 little-endian 格式的哈希值，以便与其他 SDK 保持一致。

### <a name="1.5.0"/>[1\.5.0](http://mvnrepository.com/artifact/com.microsoft.azure/azure-documentdb/1.5.0)
- 添加了哈希和范围分区冲突解决程序以协助跨多个分区对应用程序进行分片。

### <a name="1.4.0"/>[1\.4.0](http://mvnrepository.com/artifact/com.microsoft.azure/azure-documentdb/1.4.0)
- 实现 Upsert。添加了新的 upsertXXX 方法以支持 Upsert 功能。
- 实现基于 ID 的路由。无公共 API 更改，全部均为内部更改。

### <a name="1.3.0"/>1.3.0
- 跳过了发布以使版本号与其他 SDK 相符

### <a name="1.2.0"/>[1\.2.0](http://mvnrepository.com/artifact/com.microsoft.azure/azure-documentdb/1.2.0)
- 支持地理空间索引
- 验证所有资源的 ID 属性。资源的 ID 不能包含 ？、/、\#、\\ 字符或以空格结尾。
- 为 ResourceResponse 添加了新标头“索引转换进度”。

### <a name="1.1.0"/>[1\.1.0](http://mvnrepository.com/artifact/com.microsoft.azure/azure-documentdb/1.1.0)
- 实现 V2 索引策略

### <a name="1.0.0"/>[1\.0.0](http://mvnrepository.com/artifact/com.microsoft.azure/azure-documentdb/1.0.0)
- GA SDK

## 发布和停用日期
Microsoft 至少会在停用 SDK 的 **12 个月**之前发出通知，以便顺利转换到更新的/受支持的版本。

新特性和功能以及优化仅添加到当前 SDK，因此建议始终尽早升级到最新 SDK 版本。

使用已停用的 SDK 对 DocumentDB 发出的任何请求都将被服务拒绝。

> [AZURE.WARNING]
Azure DocumentDB SDK for Java 在 **1.0.0** 版之前的所有版本都将在 **2016 年 2 月 29 日**停用。
> 
> 

<br/>

| 版本 | 发布日期 | 停用日期 |
| --- | --- | --- |
| [1\.9.4](#1.9.4) |2016 年 12 月 24 日 |--- | 
| [1\.9.3](#1.9.3) |2016 年 10 月 30 日 |--- | 
| [1\.9.2](#1.9.2) |2016 年 10 月 28 日 |--- | 
| [1\.9.1](#1.9.1) |2016 年 10 月 26 日 |--- | 
| [1\.9.0](#1.9.0) |2016 年 10 月 3 日|--- | 
| [1\.8.1](#1.8.1) | 2016 年 6 月 30 日 |--- | 
| [1\.8.0](#1.8.0) | 2016 年 6 月 14 日 |--- | 
| [1\.7.1](#1.7.1) | 2016 年 4 月 30 日 |--- | 
| [1\.7.0](#1.7.0) | 2016 年 4 月 27 日 |--- | 
| [1\.6.0](#1.6.0) | 2016 年 3 月 29 日 |--- | 
| [1\.5.1](#1.5.1) | 2015 年 12 月 31 日 |---| 
| [1\.5.0](#1.5.0) | 2015 年 12 月 4 日 |--- | 
| [1\.4.0](#1.4.0) | 2015 年 10 月 5 日 |--- | 
| [1\.3.0](#1.3.0) | 2015 年 10 月 5 日 |--- | 
| [1\.2.0](#1.2.0) | 2015 年 8 月 5 日 |--- | 
| [1\.1.0](#1.1.0) | 2015 年 7 月 9 日 |--- | 
| [1\.0.1](#1.0.1) | 2015 年 5 月 12 日 |--- | 
| [1\.0.0](#1.0.0) | 2015 年 4 月 7 日 |--- | 
| 0.9.5-prelease | 2015 年 3 月 9 日 | 2016 年 2 月 29 日 | 
|0.9.4-prelease | 2015 年 2 月 17 日 | 2016 年 2 月 29 日 | 
|0.9.3-prelease | 2015 年 1 月 13 日 | 2016 年 2 月 29 日 | 
|0.9.2-prelease | 2014 年 12 月 19 日 | 2016 年 2 月 29 日 | 
|0.9.1-prelease | 2014 年 12 月 19 日 | 2016 年 2 月 29 日 | 
|0.9.0-prelease | 2014 年 12 月 10 日 | 2016 年 2 月 29 日 |

## 常见问题
[AZURE.INCLUDE [documentdb sdk 常见问题](../../includes/documentdb-sdk-faq.md)]

## 另请参阅
若要了解有关 DocumentDB 的详细信息，请参阅 [Azure DocumentDB](/home/features/documentdb/) 服务页。

<!---HONumber=Mooncake_0220_2017-->
<!---Update_Description: add more content on "Release notes"  -->