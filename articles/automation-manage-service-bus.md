<properties
	pageTitle="使用 Azure Automation 管理 Azure Service Bus"
	description="了解如何使用 Azure Automation 服务来管理 Azure Service Bus。"
	services="service-bus, automation"
	documentationCenter=""
	authors="csand-msft"
	manager="eamono"
	editor=""/>

<tags
	ms.service="service-bus"
	ms.workload="core"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="04/13/2015"
	wacn.date="05/15/2015"
	ms.author="csand"/>



# 使用 Azure Automation 管理 Azure Service Bus

本指南将介绍 Azure Automation 服务，以及如何使用它来简化 Azure Service Bus 的管理。

## 什么是 Azure Automation？

[Azure Automation](/home/features/automation/) 是用于通过流程自动化简化云管理的一项 Azure 服务。使用 Azure Automation 可以自动完成那些人工操作、经常重复、长时间运行且易出错的任务，从而改善组织的可靠性、效率和价值生成时间。

Azure Automation 提供了具有高可靠性和高可用性的工作流执行引擎，该引擎可以根据你的需求进行扩展。在 Azure Automation 中，流程可以手动、通过第三方系统或按计划的间隔启动，使任务能够完全根据需求进行。

通过将云管理任务改为由 Azure Automation 自动运行，可以降低运营开销，解放 IT 和开发运营人员，让他们将精力集中在增加企业价值的工作上。


## Azure Automation 如何帮助管理 Azure Service Bus？

可以通过使用 [Service Bus REST API](https://msdn.microsoft.com/zh-CN/library/azure/hh780717.aspx) 在 Azure Automation 中管理 Service Bus。在 Azure Automation 中，你可以编写 PowerShell 工作流脚本来使用 REST API 执行许多 Service Bus 任务。你还可以将 Azure Automation 中的这些 REST API 调用与其他 Azure 服务的 cmdlet 搭配使用，以自动完成跨 Azure 服务和第三方系统的复杂任务。


## 后续步骤

在了解 Azure Automation 以及如何使用它来管理 Azure Service Bus 的基础知识后，请使用以下链接了解有关 Azure Automation 的更多信息。

* 请参阅 Azure Automation [入门教程](/documentation/articles/automation-create-runbook-from-samples)
* 请参阅文章[使用 PowerShell 管理 Service Bus](/documentation/articles/service-bus-powershell-how-to-provision)

<!--HONumber=53-->