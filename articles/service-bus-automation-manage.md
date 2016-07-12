<properties
	pageTitle="使用 Azure 自动化管理 Azure 服务总线 | Azure"
	description="了解如何使用 Azure 自动化服务来管理 Azure 服务总线。"
	services="service-bus, automation"
	documentationCenter=""
	authors="csand-msft"
	manager="timlt"
	editor=""/>

<tags
	ms.service="service-bus"
	ms.date="04/18/2016"
	wacn.date="06/27/2016"/>



# 使用 Azure 自动化管理 Azure 服务总线

本指南将介绍 Azure 自动化服务，以及如何使用它来简化 Azure 服务总线的管理。

## 什么是 Azure 自动化？

[Azure 自动化](/documentation/services/automation/)是用于通过流程自动化和所需的状态配置简化云管理的一项 Azure 服务。使用 Azure 自动化可以自动完成那些人工操作、重复、长时间运行且易出错的任务，从而改善组织的可靠性、效率和价值生成时间。

Azure 自动化提供了具有高可靠性和高可用性的工作流执行引擎，该引擎可以根据你的需求进行扩展。在 Azure 自动化中，流程可以手动、通过第三方系统或按计划的间隔启动，使任务能够完全根据需求进行。

通过将云管理任务改为由 Azure 自动化自动运行，可以降低运营开销，解放 IT 和开发运营人员，让他们将精力集中在增加企业价值的工作上。


## Azure 自动化如何帮助管理 Azure 服务总线？

可以使用[服务总线 REST API](https://msdn.microsoft.com/zh-cn/library/azure/hh780717.aspx)，通过 Azure 自动化管理服务总线。在 Azure 自动化中，你可以运行 PowerShell 脚本来使用 REST API 执行许多服务总线任务。你还可以将 Azure 自动化中的这些 REST API 调用与其他 Azure 服务的 cmdlet 搭配使用，以自动完成跨 Azure 服务和第三方系统的复杂任务。

下面是使用 PowerShell 管理 Azure 服务总线的一些示例：
* [Custom PowerShell cmdlets to manage Azure Service Bus queues（自定义 PowerShell cmdlet 以管理 Azure 服务总线队列）](https://blogs.technet.microsoft.com/uktechnet/2014/12/04/sample-of-custom-powershell-cmdlets-to-manage-azure-servicebus-queues/)
* [如何使用 PowerShell 脚本创建服务总线队列、主题和订阅](http://blogs.msdn.com/b/paolos/archive/2014/12/02/how-to-create-a-service-bus-queues-topics-and-subscriptions-using-a-powershell-script.aspx)
* [Create Azure Service Bus Namespaces using PowerShell（使用 PowerShell 创建 Azure 服务总线命名空间）](http://buildazure.com/2015/09/24/create-azure-service-bus-namespaces-using-powershell-and-x-plat-cli/)
* [Module with DSCResource for adding configuration nodes to create Azure service bus（具有 DSCResource 的模块，用于添加配置节点以创建 Azure 服务总线）](https://www.powershellgallery.com/packages/AzureServiceBusCreation/1.0)

## 后续步骤

在了解 Azure 自动化 以及如何使用它来管理 Azure 服务总线的基础知识后，请使用以下链接了解有关 Azure 自动化的更多信息。

* 请参阅如何[使用 PowerShell 管理服务总线](/documentation/articles/service-bus-powershell-how-to-provision/)
 

<!---HONumber=Mooncake_0104_2016-->