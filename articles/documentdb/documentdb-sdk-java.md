<properties 
	pageTitle="DocumentDB Java SDK | Azure" 
	description="了解有关 Java SDK 的全部信息，包括发布日期、停用日期和 DocumentDB Java SDK 各版本之间所做的更改。" 
	services="documentdb" 
	documentationCenter="java" 
	authors="aliuy" 
	manager="jhubbard" 
	editor="cgronlun"/>

<tags 
	ms.service="documentdb" 
	ms.date="06/14/2016" 
	wacn.date="07/04/2016"/>

# DocumentDB SDK

> [AZURE.SELECTOR]
- [.NET SDK](/documentation/articles/documentdb-sdk-dotnet/)
- [Node.js SDK](/documentation/articles/documentdb-sdk-node/)
- [Java SDK](/documentation/articles/documentdb-sdk-java/)
- [Python SDK](/documentation/articles/documentdb-sdk-python/)

##DocumentDB Java SDK

<table>
<tr><td>**下载**</td><td>[Maven](http://mvnrepository.com/artifact/com.microsoft.azure/azure-documentdb)</td></tr>
<tr><td>**参与**</td><td>[GitHub](https://github.com/Azure/azure-documentdb-java/)</td></tr>
<tr><td>**文档**</td><td>[Java SDK 参考文档](http://azure.github.io/azure-documentdb-java/)</td></tr>
<tr><td>**入门**</td><td>[Java SDK 入门](/documentation/articles/documentdb-java-application/)</td></tr>
<tr><td>**当前受支持的运行时**</td><td>[JDK 7](http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html)</td></tr>
</table></br>

## 发行说明

### <a name="1.8.0"/>[1\.8.0](http://mvnrepository.com/artifact/com.microsoft.azure/azure-documentdb/1.8.0)
  - 添加了对多区域数据库帐户的支持。
  - 添加对自动重试限制请求的支持，并提供了选项用于自定义最大重试次数和最大重试等待时间。请参阅 RetryOptions 和 ConnectionPolicy.getRetryOptions()。 
  - 已弃用基于 IPartitionResolver 的自定义分区代码。请使用分区集合来提高存储和吞吐量。 

### <a name="1.7.1"/>[1\.7.1](http://mvnrepository.com/artifact/com.microsoft.azure/azure-documentdb/1.7.1)
- 对限制添加了重试策略支持。  

### <a name="1.7.0"/>[1\.7.0](http://mvnrepository.com/artifact/com.microsoft.azure/azure-documentdb/1.7.0)
- 对文档添加了生存时间 (TTL) 支持。 

### <a name="1.6.0"/>[1\.6.0](http://mvnrepository.com/artifact/com.microsoft.azure/azure-documentdb/1.6.0)
- 实现了[分区集合](/documentation/articles/documentdb-partition-data/)和[用户定义的性能级别](/documentation/articles/documentdb-performance-levels/)。 

### <a name="1.5.1"/>[1\.5.1](http://mvnrepository.com/artifact/com.microsoft.azure/azure-documentdb/1.5.1)
- 修复了 HashPartitionResolver 中的 Bug 以生成 little-endian 格式的哈希值，以便与其他 SDK 保持一致。

### <a name="1.5.0"/>[1\.5.0](http://mvnrepository.com/artifact/com.microsoft.azure/azure-documentdb/1.5.0)
- 添加哈希和范围分区冲突解决程序以协助跨多个分区对应用程序进行分片。

### <a name="1.4.0"/>[1\.4.0](http://mvnrepository.com/artifact/com.microsoft.azure/azure-documentdb/1.4.0)
- 实现 Upsert。添加了新的 upsertXXX 方法以支持 Upsert 功能。
- 实现基于 ID 的路由。无公共 API 更改，全部均为内部更改。

### <a name="1.3.0"/>1.3.0
- 跳过了发布以使版本号与其他 SDK 符合

### <a name="1.2.0"/>[1\.2.0](http://mvnrepository.com/artifact/com.microsoft.azure/azure-documentdb/1.2.0)
- 支持地理空间索引
- 验证所有资源的 ID 属性。资源的 ID 不能包含 ？、/、#、\\、字符或以空格结尾。
- 将新标头“索引转换进度”添加到 ResourceResponse。

### <a name="1.1.0"/>[1\.1.0](http://mvnrepository.com/artifact/com.microsoft.azure/azure-documentdb/1.1.0)
- 实现 V2 索引策略

### <a name="1.0.0"/>[1\.0.0](http://mvnrepository.com/artifact/com.microsoft.azure/azure-documentdb/1.0.0)
- GA SDK

## 发布和停用日期
Microsoft 将在停用一款 SDK 之前至少 **12 个月**发出通知，以便顺利过渡到更新的/受支持的版本。

新特性和功能以及优化仅添加到当前 SDK，因此建议你始终尽早升级到最新 SDK 版本。

使用已停用的 SDK 对 DocumentDB 发出的任何请求都将被服务拒绝。

> [AZURE.WARNING]
**1.0.0** 版之前的 Azure DocumentDB SDK for Java 的所有版本都将在 **2016 年 2 月 29 日**停用。

<br/>

| 版本 | 发布日期 | 停用日期 
| ---	  | ---	         | ---
| [1\.8.0](#1.8.0) | 2016 年 6 月 14 日 |---
| [1\.7.1](#1.7.1) | 2016 年 4 月 30 日 |---
| [1\.7.0](#1.7.0) | 2016 年 4 月 27 日 |---
| [1\.6.0](#1.6.0) | 2016 年 3 月 29 日 |---
| [1\.5.1](#1.5.1) | 2015 年 12 月 31 日 |---
| [1\.5.0](#1.5.0) | 2015 年 12 月 4 日 |---
| [1\.4.0](#1.4.0) | 2015 年 10 月 5 日 |---
| [1\.3.0](#1.3.0) | 2015 年 10 月 5 日 |---
| [1\.2.0](#1.2.0) | 2015 年 8 月 5 日 |---
| [1\.1.0](#1.1.0) | 2015 年 7 月 9 日 |---
| [1\.0.1](#1.0.1) | 2015 年 5 月 12 日 |---
| [1\.0.0](#1.0.0) | 2015 年 4 月 7 日 |---
|0.9.5-prelease | 2015 年 3 月 9 日 | 2016 年 2 月 29 日
|0.9.4-prelease | 2015 年 2 月 17 日 | 2016 年 2 月 29 日
|0.9.3-prelease | 2015 年 1 月 13 日 | 2016 年 2 月 29 日
|0.9.2-prelease | 2014 年 12 月 19 日 | 2016 年 2 月 29 日
|0.9.1-prelease | 2014 年 12 月 19 日 | 2016 年 2 月 29 日
|0.9.0-prelease | 2014 年 12 月 10 日 | 2016 年 2 月 29 日

## 常见问题
[AZURE.INCLUDE [documentdb sdk 常见问题](../includes/documentdb-sdk-faq.md)]

## 另请参阅

要了解有关 DocumentDB 的详细信息，请参阅 [Azure DocumentDB](https://azure.microsoft.com/services/documentdb/) 服务页。

<!---HONumber=Mooncake_0627_2016-->