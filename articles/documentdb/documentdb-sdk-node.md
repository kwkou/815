<properties
    pageTitle="DocumentDB Node.js API 和 SDK | Azure"
    description="了解有关 Node.js API 和 SDK 的全部信息，包括发布日期、停用日期和 DocumentDB Node.js SDK 各版本之间所做的更改。"
    services="documentdb"
    documentationcenter="nodejs"
    author="rnagpal"
    manager="jhubbard"
    editor="cgronlun" />
<tags
    ms.assetid="9d5621fa-0e11-4619-a28b-a19d872bcf37"
    ms.service="documentdb"
    ms.workload="data-services"
    ms.tgt_pltfrm="na"
    ms.devlang="nodejs"
    ms.topic="article"
    ms.date="12/22/2016"
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

## DocumentDB Node.js API 和 SDK
<table>

<tr><td>**下载 SDK**</td><td><a href="https://www.npmjs.com/package/documentdb">NPM</a></td></tr>

<tr><td>**API 文档**</td><td><a href="http://azure.github.io/azure-documentdb-node/DocumentClient.html">Node.js API 参考文档</a></td></tr>

<tr><td>**SDK 安装说明**</td><td><a href="http://azure.github.io/azure-documentdb-node/">安装说明</a></td></tr>

<tr><td>**参与 SDK**</td><td><a href="https://github.com/Azure/azure-documentdb-node/tree/master/source">GitHub</a></td></tr>

<tr><td>**示例**</td><td><a href="/documentation/articles/documentdb-nodejs-samples/">Node.js 代码示例</a></td></tr>

<tr><td>**入门教程**</td><td><a href="/documentation/articles/documentdb-nodejs-get-started/">Node.js SDK 入门</a></td></tr>

<tr><td>**Web 应用教程**</td><td><a href="/documentation/articles/documentdb-nodejs-application/">使用 DocumentDB 构建 Node.js Web 应用程序</a></td></tr>

<tr><td>**当前受支持的平台**</td><td><a href="https://nodejs.org/en/blog/release/v0.10.0/">Node.js v0.10</a><br/><a href="https://nodejs.org/en/blog/release/v0.12.0/">Node.js v0.12</a><br/><a href="https://nodejs.org/en/blog/release/v4.2.0/">Node.js v4.2.0</a></td></tr>
</table>

## 发行说明

### <a name="1.10.1"/>1.10.1</a>
- 使用 SDK 定位模拟器 \(hostname=localhost\) 时禁用 SSL 验证。
- 添加了在存储过程执行期间对启用脚本日志记录的支持。

### <a name="1.10.0"/>1.10.0</a>
- 添加了对跨分区并行查询的支持。
- 添加了对分区集合的 TOP/ORDER BY 查询支持。

### <a name="1.9.0"/>1.9.0</a>
- 添加了对限制请求的重试策略支持。（限制请求收到请求速率太大的异常，错误代码 429。） 默认情况下，遇到错误代码 429 时，DocumentDB 将针对每个请求重试九次，具体取决于响应标头中的 retryAfter 时间。如果想要忽略重试之间由服务器返回的 retryAfter 时间，现在可以对 ConnectionPolicy 对象设置固定的重试间隔时间，并将其作为 RetryOptions 属性的一部分。DocumentDB 现在对每个要中止的请求等待最多 30 秒（不考虑重试计数），并返回错误代码为 429 的响应。还可以在 ConnectionPolicy 对象的 RetryOptions 属性中替代该时间。
- DocumentDB 现在将 x-ms-throttle-retry-count 和 x-ms-throttle-retry-wait-time-ms 作为每个请求的响应标头返回，以表示限制重试计数和重试之间请求所等待的累计时间。
- 已添加 RetryOptions 类，从而公开了 ConnectionPolicy 类上可用于替代某些默认重试选项的 RetryOptions 属性。

### <a name="1.8.0"/>1.8.0</a>
- 添加了对多区域数据库帐户的支持。

### <a name="1.7.0"/>1.7.0</a>
- 添加了对文档生存时间 \(TTL\) 功能的支持。

### <a name="1.6.0"/>1.6.0</a>
- 实现了[分区集合](/documentation/articles/documentdb-partition-data/)和[用户定义的性能级别](/documentation/articles/documentdb-performance-levels/)。

### <a name="1.5.6"/>1.5.6</a>
- 修复了 RangePartitionResolver.resolveForRead Bug，其会由于结果的错误 concat，它不返回链接。

### <a name="1.5.5"/>1.5.5</a>
- 修复了 hashParitionResolver resolveForRead\(\)：没有提供分区键会引发异常，而不是返回所有已注册链接的列表。

### <a name="1.5.4"/>1.5.4</a>
- 修复了问题 [\#100](https://github.com/Azure/azure-documentdb-node/issues/100) - 专用 HTTPS 代理：避免因 DocumentDB 用途而修改全局代理。对所有 lib 的请求均使用专用代理。

### <a name="1.5.3"/>1.5.3</a>
- 修复了问题 [\#81](https://github.com/Azure/azure-documentdb-node/issues/81) - 正确处理媒体 ID 中的短划线。

### <a name="1.5.2"/>1.5.2</a>
- 修复了问题 [\#95](https://github.com/Azure/azure-documentdb-node/issues/95) - EventEmitter 侦听器泄漏警告。

### <a name="1.5.1"/>1.5.1</a>
- 修复了问题 [\#92](https://github.com/Azure/azure-documentdb-node/issues/90) - 对区分大小写的系统，将文件夹 Hash 重命名为 hash。

### <a name="1.5.0"/>1.5.0</a>
- 通过添加哈希和范围分区解析程序来实现分片支持。

### <a name="1.4.0"/>1.4.0</a>
- 实现 Upsert。DocumentClient 上的新 upsertXXX 方法。

### <a name="1.3.0"/>1.3.0</a>
- 跳过以使版本号与其他 SDK 相符。

### <a name="1.2.2"/>1.2.2</a>
- 将 Q 承诺包装拆分到新的存储库。
- 更新到 npm 注册表的程序包文件。

### <a name="1.2.1"/>1.2.1</a>
- 实现基于 ID 的路由。
- 修复了问题 [\#49](https://github.com/Azure/azure-documentdb-node/issues/49) - 当前属性与方法 current\(\) 发生冲突。

### <a name="1.2.0"/>1.2.0</a>
- 添加对地理空间索引的支持。
- 验证所有资源的 ID 属性。资源的 ID 不能包含 ？、/、\#、&#47;&#47;、字符或以空格结尾。
- 为 ResourceResponse 添加了新标头“索引转换进度”。

### <a name="1.1.0"/>1.1.0</a>
- 实现 V2 索引策略。

### <a name="1.0.3"/>1.0.3</a>
- 问题 [\#40](https://github.com/Azure/azure-documentdb-node/issues/40) - 已在核心和承诺 SDK 中实现 eslint 和 grunt 配置。

### <a name="1.0.2"/>1.0.2</a>
- 问题 [\#45](https://github.com/Azure/azure-documentdb-node/issues/45) - 承诺包装器不包括错误的标头。

### <a name="1.0.1"/>1.0.1</a>
- 通过添加 readConflicts、readConflictAsync 和 queryConflicts 实现了查询冲突的功能。
- 更新了 API 文档。
- 问题 [\#41](https://github.com/Azure/azure-documentdb-node/issues/41) - client.createDocumentAsync 错误。

### <a name="1.0.0"/>1.0.0</a>
- GA SDK。

## 发布和停用日期
Microsoft 至少会在停用 SDK 的 **12 个月**之前发出通知，以便顺利转换到更新的/受支持的版本。

新特性和功能以及优化仅添加到当前 SDK，因此建议始终尽早升级到最新 SDK 版本。

使用已停用的 SDK 对 DocumentDB 发出的任何请求都将被服务拒绝。

<br/>  


| 版本 | 发布日期 | 停用日期 |
| --- | --- | --- |
| [1\.10.1](#1.10.1) \|2016 年 12 月 22 日 \|--- \| 
| [1\.10.0](#1.10.0) \|2016 年 10 月 3 日 \|--- \| 
| [1\.9.0](#1.9.0) \|2016 年 7 月 7 日 \|--- \| 
| [1\.8.0](#1.8.0) \|2016 年 6 月 14 日 \|--- \| 
| [1\.7.0](#1.7.0) \|2016 年 4 月 26 日 \|--- \| 
| [1\.6.0](#1.6.0) \|2016 年 3 月 29 日 \|--- \| 
| [1\.5.6](#1.5.6) \|2016 年 3 月 8 日 \|--- \| 
| [1\.5.5](#1.5.5) \|2016 年 2 月 2 日 \|--- \| 
| [1\.5.4](#1.5.4) \|2016 年 2 月 1 日 \|--- \| 
| [1\.5.2](#1.5.2) \|2016 年 1 月 26 日 \|--- \| 
| [1\.5.2](#1.5.2) \|2016 年 1 月 22日 \|--- \| 
| [1\.5.1](#1.5.1) \|2016 年 1 月 4 日 \|--- \| 
| [1\.5.0](#1.5.0) \|2015 年 12 月 31 日 \|--- \| 
| [1\.4.0](#1.4.0) \|2015 年 10 月 6 日 \|--- \| 
| [1\.3.0](#1.3.0) \|2015 年 10 月 6 日 \|--- \| 
| [1\.2.2](#1.2.2) \|2015 年 9 月 10日 \|--- \| 
| [1\.2.1](#1.2.1) \|2015 年 8 月 15日 \|--- \| 
| [1\.2.0](#1.2.0) \|2015 年 8 月 5 日 \|--- \| 
| [1\.1.0](#1.1.0) \|2015 年 7 月 9 日 \|--- \| 
| [1\.0.3](#1.0.3) \|2015 年 6 月 4 日 \|--- \| 
| [1\.0.2](#1.0.2) \|2015 年 5 月 23 日 \|--- \| 
| [1\.0.1](#1.0.1) \|2015年 5 月 15日 \|--- \| 
| [1\.0.0](#1.0.0) \|2015 年 4 月 8 日 \|--- \|

## 常见问题
[AZURE.INCLUDE [documentdb sdk 常见问题](../../includes/documentdb-sdk-faq.md)]

## 另请参阅
若要了解有关 DocumentDB 的详细信息，请参阅 [Azure DocumentDB](/home/features/documentdb/) 服务页。

<!---HONumber=Mooncake_0220_2017-->
<!--Update_Description: wording update-->