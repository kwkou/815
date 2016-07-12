<properties 
	pageTitle="DocumentDB Python SDK | Azure" 
	description="了解有关 Python SDK 的全部信息，包括发布日期、停用日期和 DocumentDB Python SDK 各版本之间所做的更改。" 
	services="documentdb" 
	documentationCenter="python" 
	authors="aliuy" 
	manager="jhubbard" 
	editor="cgronlun"/>

<tags 
	ms.service="documentdb" 
	ms.date="06/14/2016" 
	wacn.date="06/30/2016"/>

# DocumentDB SDK

> [AZURE.SELECTOR]
- [.NET SDK](/documentation/articles/documentdb-sdk-dotnet/)
- [Node.js SDK](/documentation/articles/documentdb-sdk-node/)
- [Java SDK](/documentation/articles/documentdb-sdk-java/)
- [Python SDK](/documentation/articles/documentdb-sdk-python/)

##DocumentDB Python SDK

<table>
<tr><td>**下载**</td><td>[PyPI](https://pypi.python.org/pypi/pydocumentdb)</td></tr>
<tr><td>**参与**</td><td>[GitHub](https://github.com/Azure/azure-documentdb-python)</td></tr>
<tr><td>**文档**</td><td>[Python SDK 参考文档](http://azure.github.io/azure-documentdb-python/)</td></tr>
<tr><td>**入门**</td><td>[Python SDK 入门](/documentation/articles/documentdb-python-application/)</td></tr>
<tr><td>**当前受支持的平台**</td><td>[Python 2.7](https://www.python.org/download/releases/2.7/)</td></tr>
</table></br>

## 发行说明

### <a name="1.8.0"/>[1\.8.0](https://pypi.python.org/pypi/pydocumentdb/1.8.0)
  - 添加了对多区域数据库帐户的支持。

### <a name="1.7.0"/>[1\.7.0](https://pypi.python.org/pypi/pydocumentdb/1.7.0)
- 在文档中添加了对生存时间 (TTL) 的支持。

### <a name="1.6.1"/>[1\.6.1](https://pypi.python.org/pypi/pydocumentdb/1.6.1)
- 与服务器端分区相关的 bug 修复，以允许在 partitionkey 路径中使用特殊字符。

### <a name="1.6.0"/>[1\.6.0](https://pypi.python.org/pypi/pydocumentdb/1.6.0)
- 已实现[分区集合](/documentation/articles/documentdb-partition-data/)和[用户定义的性能级别](/documentation/articles/documentdb-performance-levels/)。 

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
- 实现 V2 索引策略

### <a name="1.0.1"/>[1\.0.1](https://pypi.python.org/pypi/pydocumentdb/1.0.1)
- 支持代理连接

### <a name="1.0.0"/>[1\.0.0](https://pypi.python.org/pypi/pydocumentdb/1.0.0)
- GA SDK

## 发布和停用日期
Microsoft 将在停用一款 SDK 之前至少 **12 个月**发出通知，以便顺利过渡到更新的/受支持的版本。

新特性和功能以及优化仅添加到当前 SDK，因此建议你始终尽早升级到最新 SDK 版本。

使用已停用的 SDK 对 DocumentDB 发出的任何请求都将被服务拒绝。

> [AZURE.WARNING]
**1.0.0** 版之前的 Azure DocumentDB SDK for Python 的所有版本都将在 **2016 年 2 月 29 日**停用。

<br/>

| 版本 | 发布日期 | 停用日期 
| ---	  | ---	         | ---
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
[AZURE.INCLUDE [documentdb sdk 常见问题](../includes/documentdb-sdk-faq.md)]

## 另请参阅

要了解有关 DocumentDB 的详细信息，请参阅 [Azure DocumentDB](/services/documentdb/) 服务页。

<!---HONumber=Mooncake_0627_2016-->