<properties
    pageTitle="Azure 媒体服务错误代码 | Azure"
    description="本主题概述 Azure 媒体服务错误代码。"
    author="Juliako"
    manager="erikre"
    editor=""
    services="media-services"
    documentationcenter="" />  

<tags
    ms.assetid="d3a62a64-7608-4b17-8667-479b26ba0d6c"
    ms.service="media-services"
    ms.workload="media"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="10/25/2016"
    wacn.date="12/12/2016"
    ms.author="juliako" />  


# Azure 媒体服务错误代码
使用 Azure 媒体服务时，可能会从服务收到因问题而异的 HTTP 错误代码，例如身份验证令牌到期或媒体服务不支持操作。以下是媒体服务可能返回的 **HTTP 错误代码**及其可能原因的列表。

## 400 错误的请求
请求包含无效的信息，并出于以下原因之一被拒绝：

* 指定了不受支持的 API 版本。有关最新版本的信息，请参阅[媒体服务 REST API 开发的设置](/documentation/articles/media-services-rest-how-to-use/)。
* 未指定媒体服务的 API 版本。有关如何指定 API 版本的信息，请参阅[使用媒体服务 REST API 连接到媒体服务](/documentation/articles/media-services-rest-connect-programmatically/)。
  
  > [AZURE.NOTE]如果使用 .NET 或 Java SDK 连接到媒体服务，则无论何时尝试并执行针对媒体服务的操作，都会指定 API 版本。

* 指定了未定义的属性。错误消息中提供属性名称。仅可指定身份是给定实体的成员的那些属性。有关实体及其属性的列表，请参阅 [Azure 媒体服务 REST API 参考](http://msdn.microsoft.com/zh-cn/library/azure/hh973617.aspx)。
* 指定了无效的属性值。错误消息中提供属性名称。请参阅上一个链接，了解有效的属性类型及其值。
* 缺少必需的属性值。
* 指定的 URL 中包含错误的值。
* 尝试更新 WriteOnce 属性。
* 尝试创建输入资产中有未指定的或无法确定的主 AssetFile 的作业。
* 尝试更新 SAS 定位符。只能创建或删除 SAS 定位符。可以更新流式处理定位符。有关详细信息，请参阅[定位符](http://msdn.microsoft.com/zh-cn/library/azure/hh974308.aspx)。
* 提交了不受支持的操作或查询。

## 401 未授权
出于以下原因之一，无法对请求进行身份验证（在可以获得授权前）：

* 缺少身份验证标头。
* 身份验证标头值错误。
  * 令牌已过期。如果直接使用 REST API，请参阅[使用媒体服务 REST API 连接到媒体服务](/documentation/articles/media-services-rest-connect-programmatically/)，了解如何生成新的身份验证令牌。如果使用 .NET 或 Java SDK，请创建用于生成令牌的 CloudMediaContext 或 MediaContract 对象。有关如何操作的详细信息，请参阅[使用用于 .NET 的媒体服务 SDK 连接到媒体服务](/documentation/articles/media-services-dotnet-connect-programmatically/)。
  * 令牌包含无效签名。</li></ul></li></ul>

## 403 禁止访问
出于以下原因之一，不允许该请求：

* 无法找到媒体服务帐户或媒体服务帐户已被删除。
* 媒体服务帐户已被禁用且请求类型不是 HTTP GET。服务操作也会返回 403 响应。
* 身份验证令牌不包含用户的凭据信息：AccountName 和/或 SubscriptionId。有关 Azure 管理门户中媒体服务帐户的此类信息，可在媒体服务 UI 扩展中找到。
* 无法访问资源。
  
  * 尝试使用不可用于媒体服务帐户的 MediaProcessor。
  * 尝试更新媒体服务定义的 JobTemplate。
  * 尝试覆盖某些其他媒体服务帐户的定位符。
  * 尝试覆盖某些其他媒体服务帐户的 ContentKey。
* 由于已达到媒体服务帐户的服务配额，无法创建资源。有关服务配额的详细信息，请参阅[配额和限制](/documentation/articles/media-services-quotas-and-limitations/)。

## 404 未找到
出于以下原因之一，资源不允许该请求：

* 尝试更新不存在的实体。
* 尝试删除不存在的实体。
* 尝试创建与不存在的实体链接的实体。
* 尝试对不存在的实体执行 GET 操作。
* 尝试指定未与媒体服务帐户关联的存储帐户。

## 409 冲突
出于以下原因之一，不允许该请求：

* 资产内有多个 AssetFile 具有指定的名称。
* 尝试在资产内创建第二个主 AssetFile。
* 尝试使用已使用的指定 ID 创建 ContentKey。
* 尝试使用已使用的指定 ID 创建定位符。
* IngestManifest 内有多个 IngestManifestFile 具有指定的名称。
* 尝试将第二个存储加密 ContentKey 链接到存储加密资产。
* 尝试将同一 ContentKey 链接到资产。
* 尝试为资产创建缺少存储容器或不再与资产关联的定位符。
* 尝试为已在使用 5 个定位符的资产创建定位符。（Azure 存储强制实施 1 个存储容器只能拥有 5 个共享访问策略的限制。）
* 将资产的存储帐户链接到 IngestManifestAsset 不同于父 IngestManifest 使用的存储帐户。

## 500 内部服务器错误
在处理请求期间，媒体服务会遇到一些阻止处理继续执行的错误。这可能是以下原因之一造成的：

* 创建资产或作业因媒体服务帐户的服务配额信息暂时不可用而失败。
* 创建资产或 IngestManifest blob 存储容器因帐户的存储帐户信息暂时不可用而失败。
* 其他意外错误。

## 503 服务不可用
服务器当前无法接收请求。导致此错误的可能原因是向服务发出了过多的请求。对于发出过多服务请求的应用程序，媒体服务限制机制将限制其资源使用量。

> [AZURE.NOTE]查看错误消息和错误代码字符串，获取收到 503 错误的原因的更多详细信息。此错误并不始终意味着限制。

可能的状态说明包括：

* “服务器正忙。此类型请求之前的运行时间超过 {0} 秒。”
* “服务器正忙。每秒超过 {0} 个请求时可能被限制。”
* “服务器正忙。每 {1} 秒超过 {0} 个请求时可能被限制。”

若要处理此错误，我们建议使用指数回退重试逻辑。这意味着在连续错误响应的重试之间使用渐进式长等待。有关详细信息，请参阅[临时故障处理应用程序块](https://msdn.microsoft.com/zh-cn/library/hh680905.aspx)。

> [AZURE.NOTE]如果使用[用于 .NET 的 Azure 媒体服务 SDK](https://github.com/Azure/azure-sdk-for-media-services/tree/master)，503 错误的重试逻辑由 SDK 实现。


## 另请参阅
[媒体服务管理错误代码](http://msdn.microsoft.com/zh-cn/library/windowsazure/dn167016.aspx)

<!---HONumber=Mooncake_1205_2016-->