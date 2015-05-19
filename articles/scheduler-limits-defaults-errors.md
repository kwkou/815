<properties 
 pageTitle="计划程序的限制、默认值和错误代码" 
 description="" 
 services="scheduler" 
 documentationCenter=".NET" 
 authors="krisragh" 
 manager="dwrede" 
 editor=""/>
<tags 
wacn.date="05/15/2015"
 ms.service="scheduler" 
 ms.workload="infrastructure-services" 
 ms.tgt_pltfrm="na" 
 ms.devlang="dotnet" 
 ms.topic="article" 
 ms.date="04/22/2015" 
 ms.author="krisragh"/>
 
# 计划程序的限制、默认值和错误代码

## 计划程序配额、限制、默认值和中止值

下表描述 Azure 计划程序中各主要的配额、限制、默认值和中止值。

|资源|限制说明|
|---|---|
|**作业大小**|最大作业大小为 16K。如果某个作业中的 PUT 或 PATCH 结果大于这些限制，则返回 400 错误请求状态代码。|
|**请求 URL 大小**|请求 URL 的最大大小为 2048 个字符。|
|**聚合标头大小**|最大聚合标头大小为 4096 个字符。|
|**标头计数**|最大标头计数为 50 个标头。|
|**正文大小**|最大正文大小为 8192 个字符。|
|**重复间隔**|最大重复间隔为 18 个月。|
|**距离开始时间的时间**|最大"距离开始时间的时间"为 18 个月。|
|**作业历史记录**|作业历史记录中存储的最大响应正文为 2048 个字节。|
|**频率**|默认的最大频率配额在免费作业集合中为 1 小时，在标准作业集合中为 1 分钟。在作业集合上，最大频率可配置为小于该最大值。作业集合中的所有作业都受到对作业集合设置的该值的限制。如果你尝试创建其频率高于针对作业集合的最大频率的作业，则请求将失败并且具有 409 冲突状态代码。|
|**作业数**|默认的最大作业配额在免费作业集合中为 5 个作业，在标准作业集合中为 50 个作业。在作业集合上，可配置作业的最大数目。作业集合中的所有作业都受到对作业集合设置的该值的限制。如果你尝试创建超过最大作业配额数目的更多作业，则请求将失败并且具有 409 冲突状态代码。|
|**作业历史记录保留期**|作业历史记录最多保留 2 个月。|
|**已完成和出错作业的保留期**|完成的作业和出错的作业保留 60 天。|
|**超时**|HTTP 操作的静态（不可配置）请求超时为 30 秒。对于长时间运行的操作，将遵循 HTTP 异步协议；例如，立即返回 202 但在后台继续工作。|

## x-ms-request-id 标头

对计划程序服务的每个请求都会返回一个名为**x-ms-request-id** 的响应标头。此响应标头包含一个唯一标识此请求的不透明值。

如果请求总是失败，并且你验证了请求格式正确，可以使用此值向 Microsoft 报告错误。请在报告中包含 x-ms-request-id 值，发出请求的大概时间，订阅、云服务、作业集合和/或作业的标识符，以及请求尝试进行的操作类型。

## 计划程序状态和错误代码

除了标准 HTTP 状态代码之外，Azure 计划程序 REST API 还返回扩展的错误代码和错误消息。这些扩展代码不替代标准 HTTP 状态代码，但提供可与标准 HTTP 状态代码一起使用的附加的、可行信息。 

例如，可能有多种原因会导致 HTTP 404 错误，因此，在扩展消息中具有这些附加信息可帮助解决问题。有关 REST API 返回的标准 HTTP 代码的详细信息，请参阅[服务管理状态和错误代码](https://msdn.microsoft.com/zh-CN/library/azure/ee460801.aspx)。服务管理 API 的 REST API 操作返回标准 HTTP 状态代码，[HTTP/1.1 状态代码定义](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)中定义了这些代码。下表介绍服务可能返回的常见错误。

|错误代码|HTTP 状态代码|用户消息|
|----|----|----|
|MissingOrIncorrectVersionHeader|错误的请求 (400)|未指定或未正确指定版本控制标头。|
|InvalidXmlRequest|错误的请求 (400)|请求正文的 XML 无效或未正确指定。|
|MissingOrInvalidRequiredQueryParameter|错误的请求 (400)|没有为此请求指定或未正确指定所需的查询参数。|
|InvalidHttpVerb|错误的请求 (400)|指定的 HTTP 谓词未被服务器识别或对此资源无效。|
|AuthenticationFailed|禁止 (403)|服务器未能验证请求。请确认证书有效并与此订阅相关联。|
|ResourceNotFound|未找到 (404)|指定的资源不存在。|
|InternalError|内部服务器错误 (500)|服务器遇到内部错误。请重试请求。|
|OperationTimedOut|内部服务器错误 (500)|无法在允许的时间内完成操作。|
|ServerBusy|服务不可用 (503)|服务器（或内部组件）当前不可用，无法接收请求。请重试你的请求。|
|SubscriptionDisabled|禁止 (403)|订阅处于已禁用状态。|
|BadRequest|错误的请求 (400)|参数不正确。|
|ConflictError|冲突 (409)|发生了冲突，使操作未能完成。|
|TemporaryRedirect|临时重定向 (307)|请求的对象不可用。可从响应的 Location 字段中获取该对象新位置的临时 URI。可对新 URI 重复执行原始请求。|

API 操作可能还会返回由管理服务定义的其他错误信息。此额外错误信息将返回到响应正文中。错误响应的正文采用下面所示的基本格式。

		<?xml version="1.0" encoding="utf-8"?>  
		<Error>  
			<Code>string-code</Code>  
			<Message>detailed-error-message</Message>  
		</Error>  

## 另请参阅

 [计划程序的概念、术语和实体层次结构](/documentation/articles/scheduler-concepts-terms)
 
 [在管理门户中开始使用计划程序](/documentation/articles/scheduler-get-started-portal)
 
 [Azure 计划程序中的计划和计费方式](/documentation/articles/scheduler-plans-billing)
 
 [如何使用 Azure 计划程序生成复杂的计划和高级循环](/documentation/articles/scheduler-advanced-complexity)
 
 [计划程序 REST API 参考](https://msdn.microsoft.com/zh-CN/library/dn528946)   
 
 [计划程序高可用性和可靠性](/documentation/articles/scheduler-high-availability-reliability)
 
 [计划程序出站身份验证](/documentation/articles/scheduler-outbound-authentication)

 

<!--HONumber=53-->