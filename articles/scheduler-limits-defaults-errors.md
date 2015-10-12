<properties
 pageTitle="计划程序的限制、默认值和错误代码"
 description=""
 services="scheduler"
 documentationCenter=".NET"
 authors="krisragh"
 manager="dwrede"
 editor=""/>
<tags
 ms.service="scheduler"
 ms.date="07/28/2015"
 wacn.date="09/16/2015"/>

# 计划程序的限制、默认值和错误代码

## 计划程序配额、限制、默认值和中止值

[AZURE.INCLUDE [scheduler-limits-table](../includes/scheduler-limits-table.md)]

## x-ms-request-id 标头

对计划程序服务的每个请求都会返回一个名为“x-ms-request-id”的响应标头。此响应标头包含一个唯一标识此请求的不透明值。

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
|AuthenticationFailed|禁止访问 (403)|服务器未能对请求进行身份验证。请确认证书有效并与此订阅相关联。|
|ResourceNotFound|未找到 (404)|指定的资源不存在。|
|InternalError|内部服务器错误 (500)|服务器遇到内部错误。请重试请求。|
|OperationTimedOut|内部服务器错误 (500)|无法在允许的时间内完成该操作。|
|ServerBusy|服务不可用 (503)|服务器（或内部组件）当前不可用，无法接收请求。请重试你的请求。|
|SubscriptionDisabled|禁止访问 (403)|订阅处于禁用状态。|
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
 
 [开始在管理门户中使用计划程序](/documentation/articles/scheduler-get-started-portal)
 
 [Azure 计划程序中的计划和计费](/documentation/articles/scheduler-plans-billing)
 
 [如何使用 Azure 计划程序生成复杂的计划和高级重复执行](/documentation/articles/scheduler-advanced-complexity)
 
 [计划程序 REST API 参考](https://msdn.microsoft.com/zh-CN/library/dn528946)   
 
 [计划程序的高可用性和可靠性](/documentation/articles/scheduler-high-availability-reliability)
 
 [计划程序出站身份验证](/documentation/articles/scheduler-outbound-authentication)

 

<!---HONumber=69-->