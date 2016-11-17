<properties
 pageTitle="计划程序 PowerShell Cmdlet 参考"
 description="计划程序 PowerShell Cmdlet 参考"
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
 wacn.date="10/10/2016"
 ms.author="krisragh"/>

# 计划程序 PowerShell Cmdlet 参考

下表介绍 Azure 计划程序中每个主要 cmdlet 的参考页并链接到该页。

若要安装 Azure PowerShell 并将其与 Azure 订阅相关联，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。

有关 [Azure Resource Manager cmdlet](https://msdn.microsoft.com/zh-cn/library/mt125356(v=azure.200).aspx) 的详细信息，请参阅[将 Azure PowerShell 与 Azure Resource Manager 配合使用](/documentation/articles/powershell-azure-resource-manager/)。

Cmdlet|Cmdlet 说明
---|---
[Disable-AzureRmSchedulerJobCollection](https://msdn.microsoft.com/zh-cn/library/mt490133(v=azure.200).aspx) |禁用某一作业集合。 
[Enable-AzureRmSchedulerJobCollection](https://msdn.microsoft.com/zh-cn/library/mt490135(v=azure.200).aspx) |启用某一作业集合。
[Get-AzureRmSchedulerJob](https://msdn.microsoft.com/zh-cn/library/mt490125(v=azure.200).aspx) |获取计划程序作业。
[Get-AzureRmSchedulerJobCollection](https://msdn.microsoft.com/zh-cn/library/mt490132(v=azure.200).aspx) |获取作业集合。
[Get-AzureRmSchedulerJobHistory](https://msdn.microsoft.com/zh-cn/library/mt490126(v=azure.200).aspx) |获取作业历史记录。
[New-AzureRmSchedulerHttpJob](https://msdn.microsoft.com/zh-cn/library/mt490136(v=azure.200).aspx) |创建 HTTP 作业。
[New-AzureRmSchedulerJobCollection](https://msdn.microsoft.com/zh-cn/library/mt490141(v=azure.200).aspx) |创建作业集合。
[New-AzureRmSchedulerServiceBusQueueJob](https://msdn.microsoft.com/zh-cn/library/mt490134(v=azure.200).aspx) |创建服务总线队列作业。
[New-AzureRmSchedulerServiceBusTopicJob](https://msdn.microsoft.com/zh-cn/library/mt490142(v=azure.200).aspx) |创建服务总线主题作业。
[New-AzureRmSchedulerStorageQueueJob](https://msdn.microsoft.com/zh-cn/library/mt490127(v=azure.200).aspx) |创建存储队列作业。 
[Remove-AzureRmSchedulerJob](https://msdn.microsoft.com/zh-cn/library/mt490140(v=azure.200).aspx) |删除计划程序作业。  
[Remove-AzureRmSchedulerJobCollection](https://msdn.microsoft.com/zh-cn/library/mt490131(v=azure.200).aspx) |删除作业集合。 
[Set-AzureRmSchedulerHttpJob](https://msdn.microsoft.com/zh-cn/library/mt490130(v=azure.200).aspx) |修改计划程序 HTTP 作业。
[Set-AzureRmSchedulerJobCollection](https://msdn.microsoft.com/zh-cn/library/mt490129(v=azure.200).aspx) |修改作业集合。 
[Set-AzureRmSchedulerServiceBusQueueJob](https://msdn.microsoft.com/zh-cn/library/mt490143(v=azure.200).aspx) |修改服务总线队列作业。  
[Set-AzureRmSchedulerServiceBusTopicJob](https://msdn.microsoft.com/zh-cn/library/mt490137(v=azure.200).aspx) |修改服务总线主题作业。 
[Set-AzureRmSchedulerStorageQueueJob](https://msdn.microsoft.com/zh-cn/library/mt490128(v=azure.200).aspx) |修改存储队列作业。   

若要获取详细信息，可以运行以下任意 cmdlet：


	Get-Help <cmdlet name> -Detailed


	Get-Help <cmdlet name> -Examples


	Get-Help <cmdlet name> -Full


## 另请参阅


 [计划程序是什么？](/documentation/articles/scheduler-intro/)
 
 [Azure 计划程序的概念、术语和实体层次结构](/documentation/articles/scheduler-concepts-terms/)
 
 [开始在管理门户中使用计划程序](/documentation/articles/scheduler-get-started-portal/)
 
 [Azure 计划程序中的计划和计费](/documentation/articles/scheduler-plans-billing/)
 
 [Azure 计划程序 REST API 参考](https://msdn.microsoft.com/zh-cn/library/mt629143)
 
 [Azure 计划程序的高可用性和高可靠性](/documentation/articles/scheduler-high-availability-reliability/) 
 [Azure 计划程序的限制、默认值和错误代码](/documentation/articles/scheduler-limits-defaults-errors/)

 [Azure 计划程序出站身份验证](/documentation/articles/scheduler-outbound-authentication/)
  

<!---HONumber=Mooncake_0926_2016-->