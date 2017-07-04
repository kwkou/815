<properties
 pageTitle="计划程序限制和默认值"
 description="计划程序限制和默认值"
 services="scheduler"
 documentationCenter=".NET"
 authors="krisragh"
 manager="dwrede"
 editor=""/>
<tags
 ms.service="scheduler"
 ms.workload="infrastructure-services"
 ms.tgt_pltfrm="na"
 ms.devlang="dotnet"
 ms.topic="article"
 ms.date="08/18/2016"
 wacn.date="02/24/2017"
 ms.author="krisragh"/>

# 计划程序限制和默认值

## 计划程序配额、限制、默认值和中止值

[AZURE.INCLUDE [scheduler-limits-table](../../includes/scheduler-limits-table.md)]

## x-ms-request-id 标头

对计划程序服务的每个请求都会返回一个名为 **x-ms-request-id** 的响应标头。此响应标头包含一个唯一标识此请求的不透明值。

如果请求总是失败，并且你验证了请求格式正确，可以使用此值向 Microsoft 报告错误。在报表中包含 x-ms-request-id 值，发出请求的大概时间，订阅、作业集合和/或作业的标识符，以及请求尝试进行的操作类型。

## 另请参阅


 [计划程序是什么？](/documentation/articles/scheduler-intro/)
 
 [Azure 计划程序的概念、术语和实体层次结构](/documentation/articles/scheduler-concepts-terms/)
 
 [开始在 Azure 门户中使用计划程序](/documentation/articles/scheduler-get-started-portal/)
 
 [Azure 计划程序中的计划和计费](/documentation/articles/scheduler-plans-billing/)
 
 [Azure 计划程序 REST API 参考](https://msdn.microsoft.com/zh-cn/library/mt629143)
 
 [Azure 计划程序 PowerShell cmdlet 参考](/documentation/articles/scheduler-powershell-reference/)

 [Azure 计划程序的高可用性和可靠性](/documentation/articles/scheduler-high-availability-reliability/)
 
 [Azure 计划程序出站身份验证](/documentation/articles/scheduler-outbound-authentication/)

 

<!---HONumber=Mooncake_Quality_Review_0104_2017-->