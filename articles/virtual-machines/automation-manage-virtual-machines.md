<properties
	pageTitle="使用 Azure Automation 管理 Azure 虚拟机 | Azure"
	description="了解如何使用 Azure Automation 服务来方便管理 Azure 虚拟机。"
	services="virtual-machines-windows, automation"
	documentationCenter=""
	authors="jodoglevy"
	manager="eamono"
	editor=""/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="04/19/2016"
	wacn.date="06/29/2016" />
	



#使用 Azure Automation 管理 Azure 虚拟机

本指南介绍 Azure 自动化服务，以及如何使用它来简化 Azure 虚拟机的管理。


## 什么是 Azure Automation？

[Azure 自动化](/home/features/automation/)是用于通过流程自动化简化云管理的一项 Azure 服务。使用 Azure 自动化可以自动完成那些长时间运行、人工操作、易出错和经常重复的任务，从而改善组织的可靠性、效率和价值生成时间。

Azure 自动化提供高度可靠且高度可用的工作流执行引擎，它可以随着组织的发展，根据你的需求扩展。在 Azure 自动化中，流程可以手动、通过第三方系统或按计划的间隔启动，使任务能够完全根据需求进行。

通过将云管理任务改为由 Azure 自动化自动运行，可以降低运营开销，解放 IT 和开发运营人员，让他们将精力集中在增加企业价值的工作上。


## Azure Automation 如何帮助管理 Azure 虚拟机？

可以使用 [Azure PowerShell](https://msdn.microsoft.com/zn-ch/library/azure/jj156055.aspx) 在 Azure 自动化中管理虚拟机。Azure 自动化包含 Azure PowerShell cmdlet，因此，你可以在该服务中执行所有虚拟机管理任务。你还可以将 Azure 自动化中的这些 cmdlet 与其他 Azure 服务的 cmdlet 搭配使用，以自动完成跨 Azure 服务和第三方系统的复杂任务。


## 后续步骤

在了解 Azure 自动化以及如何使用它来管理 Azure 虚拟机的基础知识后，请详细了解：

- [Azure 自动化概述](/documentation/articles/automation-intro/)
- [我的第一个 runbook](/documentation/articles/automation-first-runbook-textual/)

<!---HONumber=Mooncake_1207_2015-->