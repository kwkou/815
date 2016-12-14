<properties
    pageTitle="用于 .NET 的媒体服务 SDK 中的重试逻辑 | Azure"
    description="本主题概述了用于 .NET 的媒体服务 SDK 中的重试逻辑。"
    author="Juliako"
    manager="erikre"
    editor=""
    services="media-services"
    documentationcenter="" />  

<tags
    ms.assetid="527b61a6-c862-4bd8-bcbc-b9aea1ffdee3"
    ms.service="media-services"
    ms.workload="media"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="10/25/2016"
    wacn.date="12/12/2016"
    ms.author="juliako" />  


# 用于 .NET 的媒体服务 SDK 中的重试逻辑
使用 Azure 服务时，可能会发生暂时性故障。如果发生暂时性故障，在大多数情况下，重试几次后操作即可成功。用于 .NET 的媒体服务 SDK 可实现重试逻辑，用于处理与异常和错误相关的暂时性故障，引起这些异常和错误的原因包括 Web 请求、执行查询、保存更改和存储操作。默认情况下，再次引发应用程序异常之前，用于 .NET 的媒体服务 SDK 将重试 4 次。应用程序中的代码稍后必须正确处理此异常。

 下面是 Web 请求、存储、查询和 SaveChanges 策略的简要原则：

* 存储策略可用于 Blob 存储操作（上传或下载资产文件）。
* Web 请求策略可用于常规 Web 请求，例如，用于获取身份验证令牌和解析用户群集终结点。
* 查询策略可用于从 REST 查询实体，例如，mediaContext.Assets.Where(…)。
* SaveChanges 策略可用于在服务内执行任何更改数据的操作，例如，创建更新实体的实体、为操作调用服务函数。
  
  本主题列出了用于 .NET 的媒体服务 SDK 的重试逻辑可处理的异常类型和错误代码。

## 异常类型
下表介绍了用于 .NET 的媒体服务 SDK 对于可能导致暂时性故障的操作可处理和不可处理的异常。

| 异常 | Web 请求 | 存储 | 查询 | SaveChanges |
| --- | --- | --- | --- | --- |
| WebException<br/>有关详细信息，请参阅 [WebException 状态代码](/documentation/articles/media-services-retry-logic-in-dotnet-sdk/#WebExceptionStatus)部分。 |是 |是 |是 |是 |
| DataServiceClientException<br/> 有关详细信息，请参阅 [HTTP 错误状态代码](/documentation/articles/media-services-retry-logic-in-dotnet-sdk/#HTTPStatusCode)。 |否 |是 |是 |是 |
| DataServiceQueryException<br/> 有关详细信息，请参阅 [HTTP 错误状态代码](/documentation/articles/media-services-retry-logic-in-dotnet-sdk/#HTTPStatusCode)。 |否 |是 |是 |是 |
| DataServiceRequestException<br/> 有关详细信息，请参阅 [HTTP 错误状态代码](/documentation/articles/media-services-retry-logic-in-dotnet-sdk/#HTTPStatusCode)。 |否 |是 |是 |是 |
| DataServiceTransportException |否 |否 |是 |是 |
| TimeoutException |是 |是 |是 |否 |
| SocketException |是 |是 |是 |是 |
| StorageException |否 |是 |否 |否 |
| IOException |否 |是 |否 |否 |

### <a name="WebExceptionStatus"></a>WebException 状态代码
下表介绍了针对其实现重试逻辑的 WebException 错误代码。[WebExceptionStatus](http://msdn.microsoft.com/zh-cn/library/system.net.webexceptionstatus.aspx) 枚举定义状态代码。

| 状态 | Web 请求 | 存储 | 查询 | SaveChanges |
| --- | --- | --- | --- | --- |
| ConnectFailure |是 |是 |是 |是 |
| NameResolutionFailure |是 |是 |是 |是 |
| ProxyNameResolutionFailure |是 |是 |是 |是 |
| SendFailure |是 |是 |是 |是 |
| PipelineFailure |是 |是 |是 |否 |
| ConnectionClosed |是 |是 |是 |否 |
| KeepAliveFailure |是 |是 |是 |否 |
| UnknownError |是 |是 |是 |否 |
| ReceiveFailure |是 |是 |是 |否 |
| RequestCanceled |是 |是 |是 |否 |
| 超时 |是 |是 |是 |否 |
| ProtocolError <br/>ProtocolError 的重试受控于 HTTP 状态代码处理。有关详细信息，请参阅 [HTTP 错误状态代码](/documentation/articles/media-services-retry-logic-in-dotnet-sdk/#HTTPStatusCode)。 |是 |是 |是 |是 |

### <a name="HTTPStatusCode"></a>HTTP 错误状态代码
如果 Query 和 SaveChanges 操作引发 DataServiceClientException、DataServiceQueryException 或 DataServiceQueryException，则 StatusCode 属性中返回 HTTP 错误状态代码。下表介绍了为哪些错误代码实现了重试逻辑。

| 状态 | Web 请求 | 存储 | 查询 | SaveChanges |
| --- | --- | --- | --- | --- |
| 401 |否 |是 |否 |否 |
| 403 |否 |是<br/>处理等待时间较长的重试。 |否 |否 |
| 408 |是 |是 |是 |是 |
| 429 |是 |是 |是 |是 |
| 500 |是 |是 |是 |否 |
| 502 |是 |是 |是 |否 |
| 503 |是 |是 |是 |是 |
| 504 |是 |是 |是 |否 |

如果要查看用于 .NET 的媒体服务 SDK 的重试逻辑的实际实现，请参阅 [azure-sdk-for-media-services](https://github.com/Azure/azure-sdk-for-media-services/tree/dev/src/net/Client/TransientFaultHandling)。

<!---HONumber=Mooncake_1205_2016-->