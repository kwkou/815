<properties
    pageTitle="DocumentDB Python API、SDK 和资源 | Azure"
    description="了解有关 Python API 和 SDK 的全部信息，包括发布日期、停用日期和 DocumentDB Python SDK 各版本之间所做的更改。"
    services="documentdb"
    documentationcenter="python"
    author="rnagpal"
    manager="jhubbard"
    editor="cgronlun" />
<tags
    ms.assetid="3ac344a9-b2fa-4a3f-a4cc-02d287e05469"
    ms.service="documentdb"
    ms.workload="data-services"
    ms.tgt_pltfrm="na"
    ms.devlang="python"
    ms.topic="article"
    ms.date="05/10/2017"
    wacn.date="05/31/2017"
    ms.author="rnagpal"
    ms.custom="H1Hack27Feb2017"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="4a18b6116e37e365e2d4c4e2d144d7588310292e"
    ms.openlocfilehash="56345266de17274b08549b458b367e37fd82078e"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/19/2017" />

# <a name="documentdb-python-sdk-release-notes-and-resources"></a>DocumentDB Python SDK：发行说明和资源
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


<tr><td>下载 SDK</td><td><a href="https://pypi.python.org/pypi/pydocumentdb">PyPI</a></td></tr>

<tr><td>API 文档</td><td><a href="http://azure.github.io/azure-documentdb-python/api/pydocumentdb.html">Python API 参考文档</a></td></tr>

<tr><td>SDK 安装说明</td><td><a href="http://azure.github.io/azure-documentdb-python/">Python SDK 安装说明</a></td></tr>

<tr><td>参与 SDK</td><td><a href="https://github.com/Azure/azure-documentdb-python">GitHub</a></td></tr>

<tr><td>入门</td><td><a href="/documentation/articles/documentdb-python-application/">Python SDK 入门</a></td></tr>

<tr><td>当前受支持的平台</td><td><a href="https://www.python.org/downloads/">Python 2.7</a> 和 <a href="https://www.python.org/downloads/">Python 3.5</a></td></tr>
</table>

## <a name="release-notes"></a>发行说明
### <a name="a-name220220"></a><a name="2.2.0"></a>2.2.0
- 添加了对每分钟请求单位数 (RU/m) 功能的支持。
- 添加了对称为“ConsistentPrefix”的新一致性级别的支持。


### <a name="a-name210210"></a><a name="2.1.0"></a>2.1.0
- 添加了对聚合查询（COUNT、MIN、MAX、SUM、AVG）的支持。
- 添加了在对 DocumentDB 模拟器运行时禁用 SSL 验证的选项。
- 删除了依赖请求模块精确是 2.10.0 的限制。
- 将分区集合上的最小吞吐量从 10,100 RU/s 降低到 2500 RU/s。
- 添加了在存储过程执行期间对启用脚本日志记录的支持。
- 此版本中的 REST API 版本已升级到“2017-01-19”。

### <a name="a-name201201"></a><a name="2.0.1"></a>2.0.1
- 对文档注释进行编辑性更改。

### <a name="a-name200200"></a><a name="2.0.0"></a>2.0.0
- 添加了对 Python 3.5 的支持。
- 添加了使用请求模块创建连接池的支持。
- 添加了会话一致性支持。
- 添加了对分区集合的 TOP/ORDERBY 查询支持。

### <a name="a-name190190"></a><a name="1.9.0"></a>1.9.0
- 添加了对限制请求的重试策略支持。 （限制请求收到请求速率太大的异常，错误代码 429。）默认情况下，遇到错误代码 429 时，DocumentDB 将针对每个请求重试九次，具体取决于响应标头中的 retryAfter 时间。 如果想要忽略重试之间由服务器返回的 retryAfter 时间，现在可以对 ConnectionPolicy 对象设置固定的重试间隔时间，并将其作为 RetryOptions 属性的一部分。 DocumentDB 现在对每个要中止的请求等待最多 30 秒（不考虑重试计数），并返回错误代码为 429 的响应。 还可以在 ConnectionPolicy 对象的 RetryOptions 属性中替代该时间。
- DocumentDB 现在将 x-ms-throttle-retry-count 和 x-ms-throttle-retry-wait-time-ms 作为每个请求的响应标头返回，以表示限制重试计数和重试之间请求所等待的累计时间。
- 删除了 document_client 类上公开的 RetryPolicy 类以及相应的属性 (retry_policy)，改为引入了在 ConnectionPolicy 对象上公开 RetryOptions 属性的 RetryOptions 类，该类可用于覆盖一些默认的重试选项。

### <a name="a-name180180"></a><a name="1.8.0"></a>1.8.0
- 添加了对多区域数据库帐户的支持。

### <a name="a-name170170"></a><a name="1.7.0"></a>1.7.0
- 添加了对文档生存时间 (TTL) 功能的支持。

### <a name="a-name161161"></a><a name="1.6.1"></a>1.6.1
- 与服务器端分区相关的 bug 修复，以允许在 partitionkey 路径中使用特殊字符。

### <a name="a-name160160"></a><a name="1.6.0"></a>1.6.0
- 实现了[分区集合](/documentation/articles/documentdb-partition-data/)和[用户定义的性能级别](/documentation/articles/documentdb-performance-levels/)。 

### <a name="a-name150150"></a><a name="1.5.0"></a>1.5.0
- 添加哈希和范围分区冲突解决程序以协助跨多个分区对应用程序进行分片。

### <a name="a-name142142"></a><a name="1.4.2"></a>1.4.2
- 实现 Upsert。 添加了新的 UpsertXXX 方法以支持 Upsert 功能。
- 实现基于 ID 的路由。 无公共 API 更改，全部均为内部更改。

### <a name="a-name120120"></a><a name="1.2.0"></a>1.2.0
- 支持地理空间索引。
- 验证所有资源的 ID 属性。 资源的 ID 不能包含 ？、/、#、\, 字符或以空格结尾。
- 将新标头“索引转换进度”添加到 ResourceResponse。

### <a name="a-name110110"></a><a name="1.1.0"></a>1.1.0
- 实现 V2 索引策略。

### <a name="a-name101101"></a><a name="1.0.1"></a>1.0.1
- 支持代理连接。

### <a name="a-name100100"></a><a name="1.0.0"></a>1.0.0
- GA SDK。

## <a name="release--retirement-dates"></a>发布和停用日期
Microsoft 至少会在停用 SDK 的 **12 个月**之前发出通知，以便顺利转换到更新的/受支持的版本。

新特性和功能以及优化仅添加到当前 SDK，因此建议你始终尽早升级到最新 SDK 版本。 

使用已停用的 SDK 对 DocumentDB 发出的任何请求都将被服务拒绝。

> [AZURE.WARNING]
> Azure DocumentDB SDK for Python 在 **1.0.0** 版之前的所有版本都将在 **2016 年 2 月 29 日**停用。 
> 
> 

<br/>

| 版本 | 发布日期 | 停用日期 |
| --- | --- | --- |
| [2.2.0](#2.2.0) |2017 年 5 月 10 日 |--- |
| [2.1.0](#2.1.0) |2017 年 5 月 1 日 |--- |
| [2.0.1](#2.0.1) |2016 年 10 月 30 日 |--- |
| [2.0.0](#2.0.0) |2016 年 9 月 29 日 |--- |
| [1.9.0](#1.9.0) |2016 年 7 月 7 日 |--- |
| [1.8.0](#1.8.0) |2016 年 6 月 14 日 |--- |
| [1.7.0](#1.7.0) |2016 年 4 月 26 日 |--- |
| [1.6.1](#1.6.1) |2016 年 4 月 8 日 |--- |
| [1.6.0](#1.6.0) |2016 年 3 月 29 日 |--- |
| [1.5.0](#1.5.0) |2016 年 1 月 3日 |--- |
| [1.4.2](#1.4.2) |2015 年 10 月 6 日 |--- |
| 1.4.1 |2015 年 10 月 6 日 |--- |
| [1.2.0](#1.2.0) |2015 年 8 月 6 日 |--- |
| [1.1.0](#1.1.0) |2015 年 7 月 9 日 |--- |
| [1.0.1](#1.0.1) |2015 年 5 月 25 日 |--- |
| [1.0.0](#1.0.0) |2015 年 4 月 7 日 |--- |
| 0.9.4-prelease |2015 年 1 月 14日 |2016 年 2 月 29 日 |
| 0.9.3-prelease |2014 年 12 月 9 日 |2016 年 2 月 29 日 |
| 0.9.2-prelease |2014 年 11 月 25 日 |2016 年 2 月 29 日 |
| 0.9.1-prelease |2014 年 9 月 23 日 |2016 年 2 月 29 日 |
| 0.9.0-prelease |2014 年 8 月 21 日 |2016 年 2 月 29 日 |

## <a name="faq"></a>常见问题
[AZURE.INCLUDE [documentdb-sdk-faq](../../includes/documentdb-sdk-faq.md)]

## <a name="see-also"></a>另请参阅
若要了解有关 DocumentDB 的详细信息，请参阅 [DocumentDB](/home/features/documentdb/) 服务页。

<!--Update_Description: wording update-->