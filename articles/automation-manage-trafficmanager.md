<properties 
 pageTitle="使用 Azure 自动化管理 Azure 流量管理器" 
 description="了解如何使用 Azure 自动化服务来管理 Azure 流量管理器。" 
 services="traffic-manager, automation" 
 documentationCenter="" 
 authors="eamonoreilly" 
 manager="adinah" 
 editor=""/>

<tags 
 ms.service="traffic-manager" 
 ms.date="08/12/2015" 
 wacn.date="10/03/2015"/>


#使用 Azure 自动化管理 Azure 流量管理器

本指南将介绍 Azure 自动化服务，以及如何使用它来简化 Azure 流量管理器的管理。

## 什么是 Azure Automation？

[Azure 自动化](/home/features/automation/)是用于通过流程自动化简化云管理的一项 Azure 服务。使用 Azure 自动化可以自动完成那些长时间运行、人工操作、易出错和经常重复的任务，从而改善组织的可靠性、效率和价值生成时间。

Azure 自动化提供高度可靠且高度可用的工作流执行引擎，它可以随着组织的发展，根据你的需求扩展。在 Azure 自动化中，流程可以手动、通过第三方系统或按计划的间隔启动，使任务能够完全根据需求进行。

通过将云管理任务改为由 Azure 自动化自动运行，可以降低运营开销，解放 IT/开发运营人员，让他们将精力集中在增加企业价值的工作上。


## Azure 自动化如何帮助管理 Azure 流量管理器？

可以使用 [Azure PowerShell 工具](https://msdn.microsoft.com/zh-CN/library/azure/jj156055.aspx)中提供的 PowerShell cmdlet 在 Azure 自动化中管理流量管理器。Azure 自动化可以调用这些 PowerShell cmdlet，因此，你可以在该服务中执行所有流量管理器管理任务。你还可以将 Azure 自动化中的这些 cmdlet 与其他 Azure 服务的 cmdlet 搭配使用，以自动完成跨 Azure 服务和第三方系统的复杂任务。


## 后续步骤

在了解 Azure 自动化以及如何使用它来管理 Azure 流量管理器的基础知识后，请使用以下链接了解有关 Azure 自动化的更多信息。

* 查看 Azure 自动化[入门指南](/documentation/articles/automation-create-runbook-from-samples/)

<!---HONumber=71-->