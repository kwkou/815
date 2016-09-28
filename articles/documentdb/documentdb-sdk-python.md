<properties 
	pageTitle="DocumentDB Python API 和 SDK | Azure" 
	description="了解有关 Python API 和 SDK 的全部信息，包括发布日期、停用日期和 DocumentDB Python SDK 各版本之间所做的更改。" 
	services="documentdb" 
	documentationCenter="python" 
	authors="rnagpal" 
	manager="jhubbard" 
	editor="cgronlun"/>

<tags ms.service="documentdb"
	ms.date="08/09/2016" 
	wacn.date="09/28/2016"/>

# DocumentDB SDK

> [AZURE.SELECTOR]
- [.NET](/documentation/articles/documentdb-sdk-dotnet/)
- [Node.js](/documentation/articles/documentdb-sdk-node/)
- [Java](/documentation/articles/documentdb-sdk-java/)
- [Python](/documentation/articles/documentdb-sdk-python/)
- [REST](https://msdn.microsoft.com/zh-cn/library/azure/dn781481.aspx)
- [SQL](https://msdn.microsoft.com/zh-cn/library/azure/dn782250.aspx)

##DocumentDB Python SDK

<table>
<tr><td>**下载 SDK**</td><td>[PyPI](https://pypi.python.org/pypi/pydocumentdb)</td></tr>
<tr><td>**API 文档**</td><td>[Python API 参考文档](http://azure.github.io/azure-documentdb-python/api/pydocumentdb.html)</td></tr>
<tr><td>**SDK 安装说明**</td><td>[Python SDK 安装说明](http://azure.github.io/azure-documentdb-python/)</td></tr>
<tr><td>**参与 SDK**</td><td>[GitHub](https://github.com/Azure/azure-documentdb-python)</td></tr>
<tr><td>**入门**</td><td>[Python SDK 入门](/documentation/articles/documentdb-python-application/)</td></tr>
<tr><td>**当前受支持的平台**</td><td>[Python 2.7](https://www.python.org/download/releases/2.7/)</td></tr>
</table>

## 发行说明

### <a name="1.9.0"/>[1\.9.0](https://pypi.python.org/pypi/pydocumentdb/1.9.0)
- 对限制添加了重试策略支持。（限制请求收到请求速率太大的异常，错误代码 429。） 默认情况下，遇到错误代码 429 时，DocumentDB 将针对每个请求重试九次，具体取决于响应标头中的 retryAfter 时间。如果想要忽略重试之间由服务器返回的 retryAfter 时间，现在可以对 ConnectionPolicy 对象设置固定的重试间隔时间，并将其作为 RetryOptions 属性的一部分。DocumentDB 现在对每个要中止的请求等待最多 30 秒（不考虑重试计数），并返回错误代码为 429 的响应。还可以在 ConnectionPolicy 对象的 RetryOptions 属性中替代该时间。

- DocumentDB 现在将 x-ms-throttle-retry-count 和 x-ms-throttle-retry-wait-time-ms 作为每个请求的响应标头返回，以表示限制重试计数和重试之间请求所等待的累计时间。

- 已移除 document\_client 类上公开的 RetryPolicy 类以及相应的属性 (retry\_policy)，引入了在 ConnectionPolicy 对象上公开 RetryOptions 属性的 RetryOptions 类，该类可用于覆盖一些默认的重试选项。

### <a name="1.8.0"/>[1\.8.0](https://pypi.python.org/pypi/pydocumentdb/1.8.0)
  - 添加了对多区域数据库帐户的支持。

### <a name="1.7.0"/>[1\.7.0](https://pypi.python.org/pypi/pydocumentdb/1.7.0)
- 在文档中添加了对生存时间 (TTL) 的支持。

### <a name="1.6.1"/>[1\.6.1](https://pypi.python.org/pypi/pydocumentdb/1.6.1)
- 与服务器端分区相关的 bug 修复，以允许在 partitionkey 路径中使用特殊字符。

### <a name="1.6.0"/>[1\.6.0](https://pypi.python.org/pypi/pydocumentdb/1.6.0)
- 实现了[分区集合](/documentation/articles/documentdb-partition-data/)和[用户定义的性能级别](/documentation/articles/documentdb-performance-levels/)。

### <a name="1.5.0"/>[1\.5.0](https://pypi.python.org/pypi/pydocumentdb/1.5.0)
- 添加哈希和范围分区冲突解决程序以协助跨多个分区对应用程序进行分片。

### <a name="1.4.2"/>[1\.4.2](https://pypi.python.org/pypi/pydocumentdb/1.4.2)
- 实现 Upsert。添加了新的 UpsertXXX 方法以支持 Upsert 功能。
- 实现基于 ID 的路由。无公共 API 更改，全部均为内部更改。

### <a name="1.2.0"/>[1\.2.0](https://pypi.python.org/pypi/pydocumentdb/1.2.0)
- 支持地理空间索引。
- 验证所有资源的 ID 属性。资源的 ID 不能包含 ？、/、#、\\、字符或以空格结尾。
- 将新标头“索引转换进度”添加到 ResourceResponse。

### <a name="1.1.0"/>[1\.1.0](https://pypi.python.org/pypi/pydocumentdb/1.1.0)
- 实施 V2 索引策略。

### <a name="1.0.1"/>[1\.0.1](https://pypi.python.org/pypi/pydocumentdb/1.0.1)
- 支持代理连接。

### <a name="1.0.0"/>[1\.0.0](https://pypi.python.org/pypi/pydocumentdb/1.0.0)
- GA SDK。

## 发布和停用日期
Microsoft 至少会在停用 SDK 的 **12 个月**之前发出通知，以便顺利转换到更新的/受支持的版本。

新特性和功能以及优化仅添加到当前 SDK，因此建议你始终尽早升级到最新 SDK 版本。

使用已停用的 SDK 对 DocumentDB 发出的任何请求都将被服务拒绝。

> [AZURE.WARNING]
Azure DocumentDB SDK for Python 在 **1.0.0** 版之前的所有版本都将在 **2016 年 2 月 29 日**停用。

<br/>

| 版本 | 发布日期 | 停用日期 
| ---	  | ---	         | ---
| [1\.9.0](#1.9.0) | 2016 年 7 月 7 日 |---
| [1\.8.0](#1.8.0) | 2016 年 6 月 14 日 |---
| [1\.7.0](#1.7.0) | 2016 年 4 月 26 日 |---
| [1\.6.1](#1.6.1) | 2016 年 4 月 8 日 |---
| [1\.6.0](#1.6.0) | 2016 年 3 月 29 日 |---
| [1\.5.0](#1.5.0) | 2016 年 1 月 3 日 |---
| [1\.4.2](#1.4.2) | 2015 年 10 月 6 日 |---
| [1\.4.1](#1.4.1) | 2015 年 10 月 6 日 |---
| [1\.2.0](#1.2.0) | 2015 年 8 月 6 日 |---
| [1\.1.0](#1.1.0) | 2015 年 7 月 9 日 |---
| [1\.0.1](#1.0.1) | 2015 年 5 月 25 日 |---
| [1\.0.0](#1.0.0) | 2015 年 4 月 7 日 |---
|0.9.4-prelease | 2015 年 1 月 14 日 | 2016 年 2 月 29 日
|0.9.3-prelease | 2014 年 12 月 9 日 | 2016 年 2 月 29 日
|0.9.2-prelease | 2014 年 11 月 25 日 | 2016 年 2 月 29 日
|0.9.1-prelease | 2014 年 9 月 23 日 | 2016 年 2 月 29 日
|0.9.0-prelease | 2014 年 8 月 21 日 | 2016 年 2 月 29日

## 常见问题
[AZURE.INCLUDE [documentdb sdk 常见问题](../../includes/documentdb-sdk-faq.md)]

## 另请参阅

要了解有关 DocumentDB 的详细信息，请参阅 [Azure DocumentDB](/services/documentdb/) 服务页。

<!---HONumber=Mooncake_0919_2016-->