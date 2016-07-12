<properties 
	pageTitle="DocumentDB Node.js SDK | Azure" 
	description="了解有关 Node.js SDK 的全部信息，包括发布日期、停用日期和 DocumentDB Node.js SDK 各版本之间所做的更改。" 
	services="documentdb" 
	documentationCenter="nodejs" 
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

##DocumentDB Node.js SDK

<table>
<tr><td>**下载**</td><td>[NPM](https://www.npmjs.com/package/documentdb)</td></tr>
<tr><td>**参与**</td><td>[GitHub](https://github.com/Azure/azure-documentdb-node/tree/master/source)</td></tr>
<tr><td>**文档**</td><td>[Node.js SDK 参考文档](http://azure.github.io/azure-documentdb-node/)</td></tr>
<tr><td>**示例**</td><td>[Node.js 代码示例](https://github.com/Azure/azure-documentdb-node/tree/master/samples)</td></tr>
<tr><td>**入门**</td><td>[Node.js SDK 入门](/documentation/articles/documentdb-nodejs-get-started/)</td></tr>
<tr><td>**当前受支持的平台**</td><td>[Node.js v0.10](https://nodejs.org/en/blog/release/v0.10.0/)<br/>[Node.js v0.12](https://nodejs.org/en/blog/release/v0.12.0/)<br/>[Node.js v4.2.0](https://nodejs.org/en/blog/release/v4.2.0/)</td></tr>
</table></br>

##发行说明

###<a name="1.8.0"/>1.8.0</a>

  - 添加了对多区域数据库帐户的支持。

###<a name="1.7.0"/>1.7.0</a>

- 在文档中添加了对生存时间 (TTL) 的支持。

###<a name="1.6.0"/>1.6.0</a>
- 已实现[分区集合](/documentation/articles/documentdb-partition-data/)和[用户定义的性能级别](/documentation/articles/documentdb-performance-levels/)。 

###<a name="1.5.6"/>1.5.6</a>

- 修复了 RangePartitionResolver.resolveForRead Bug，其会由于结果的错误拼接，它不返回链接

###<a name="1.5.5"/>1.5.5</a>

- 修复了 hashParitionResolver resolveForRead()：当没有提供分区键的时候返回所有已注册的链接

###<a name="1.5.4"/>1.5.4</a>

- 修复问题 [#100](https://github.com/Azure/azure-documentdb-node/issues/100) - 专用 HTTPS 代理：避免修改全局代理。对所有 lib 的请求均使用专用代理。

###<a name="1.5.3"/>1.5.3</a>

- 修复问题 [#81](https://github.com/Azure/azure-documentdb-node/issues/81) - 正确处理媒体 ID 中的短划线。

###<a name="1.5.2"/>1.5.2</a>

- 修复问题 [#95](https://github.com/Azure/azure-documentdb-node/issues/95) - EventEmitter 侦听器泄漏警告

###<a name="1.5.1"/>1.5.1</a>

- 修复问题 [#92](https://github.com/Azure/azure-documentdb-node/issues/90) - 对区分大小写的系统，将文件夹 Hash 重命名为 hash

### <a name="1.5.0"/>1.5.0</a>

- 通过添加哈希和范围分区冲突解决程序来实现分片支持

### <a name="1.4.0"/>1.4.0</a>

- 实现 Upsert。 DocumentClient 上的新 upsertXXX 方法。 

### <a name="1.3.0"/>1.3.0</a>

- 跳过以使版本号与其他 SDK 符合

### <a name="1.2.2"/>1.2.2</a>

- 将 Q promises 封装库移到新的代码库
- 更新 npm 注册的程序包文件

### <a name="1.2.1"/>1.2.1</a>

- 实现基于 ID 的路由
- 修复问题 [#49](https://github.com/Azure/azure-documentdb-node/issues/49) - 当前属性与方法 current() 发生冲突

### <a name="1.2.0"/>1.2.0</a>

- 添加对地理空间索引的支持。
- 验证所有资源的 ID 属性。资源的 ID 不能包含 ？、/、#、&#47;&#47;、字符或以空格结尾。 
- 将新标头“索引转换进度”添加到 ResourceResponse。

### <a name="1.1.0"/>1.1.0</a>

- 实现 V2 索引策略

### <a name="1.0.3"/>1.0.3</a>

- 问题 [#40](https://github.com/Azure/azure-documentdb-node/issues/40) - 已在核心和 promise SDK 中实现 eslint 和 grunt 配置

### <a name="1.0.2"/>1.0.2</a>

- 问题 [#45](https://github.com/Azure/azure-documentdb-node/issues/45) - 封装库移除有错误的标头。

### <a name="1.0.1"/>1.0.1</a>

- 通过添加 readConflicts、readConflictAsync 和 queryConflicts 实现了查询冲突的功能
- 更新的 API 文档
- 问题 [#41](https://github.com/Azure/azure-documentdb-node/issues/41) - client.createDocumentAsync 错误  

### <a name="1.0.0"/>1.0.0</a>

- GA SDK

## 发布和停用日期
Microsoft 将在停用一款 SDK 之前至少 **12 个月**发出通知，以便顺利过渡到更新的/受支持的版本。

新特性和功能以及优化仅添加到当前 SDK，因此建议你始终尽早升级到最新 SDK 版本。

使用已停用的 SDK 对 DocumentDB 发出的任何请求都将被服务拒绝。

> [AZURE.WARNING]
**1.0.0** 版之前的 Azure DocumentDB SDK for Node.js 的所有版本都将在 **2016 年 2 月 29 日**停用。

<br/>

| 版本 | 发布日期 | 停用日期 
| ---	  | ---	         | ---
| [1\.8.0](#1.8.0) | 2016 年 6 月 14 日 |---
| [1\.7.0](#1.7.0) | 2016 年 4 月 26 日 |---
| [1\.6.0](#1.6.0) | 2016 年 3 月 29 日 |---
| [1\.5.6](#1.5.6) | 2016 年 3 月 8 日 |---
| [1\.5.5](#1.5.5) | 2016 年 2 月 2 日 |---
| [1\.5.4](#1.5.4) | 2016 年 2 月 1 日 |---
| [1\.5.2](#1.5.2) | 2016 年 1 月 26 日 |---
| [1\.5.2](#1.5.2) | 2016 年 1 月 22 日 |---
| [1\.5.1](#1.5.1) | 2016 年 1 月 4 日 |---
| [1\.5.0](#1.5.0) | 2015 年 12 月 31 日 |---
| [1\.4.0](#1.4.0) | 2015 年 10 月 6 日 |---
| [1\.3.0](#1.3.0) | 2015 年 10 月 6 日 |---
| [1\.2.2](#1.2.2) | 2015 年 9 月 10 日 |---
| [1\.2.1](#1.2.1) | 2015 年 8 月 15 日 |---
| [1\.2.0](#1.2.0) | 2015 年 8 月 5 日 |---
| [1\.1.0](#1.1.0) | 2015 年 7 月 9 日 |---
| [1\.0.3](#1.0.3) | 2015 年 6 月 4 日，|---
| [1\.0.2](#1.0.2) | 2015 年 5 月 23 日 |---
| [1\.0.1](#1.0.1) | 2015 年 5 月 15 日 |---
| [1\.0.0](#1.0.0) | 2015 年 4 月 8 日 |---
|0.9.4-prelease | 2015 年 4 月 6 日 | 2016 年 2 月 29 日
|0.9.3-prelease | 2015 年 1 月 14 日 | 2016 年 2 月 29 日
|0.9.2-prelease | 2014 年 12 月 18 日 | 2016 年 2 月 29 日
|0.9.1-prelease | 2014 年 8 月 22 日 | 2016 年 2 月 29 日
|0.9.0-prelease | 2014 年 8 月 21 日 | 2016 年 2 月 29日


## 常见问题
[AZURE.INCLUDE [documentdb sdk 常见问题](../includes/documentdb-sdk-faq.md)]

## 另请参阅

要了解有关 DocumentDB 的详细信息，请参阅 [Azure DocumentDB](https://azure.microsoft.com/services/documentdb/) 服务页。

<!---HONumber=Mooncake_0627_2016-->