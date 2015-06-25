
<properties 
 pageTitle="使用 Azure Automation 管理 Azure 备份" 
 description="了解如何使用 Azure Automation 服务来管理 Azure 备份。" 
 services="backup, automation" 
 documentationCenter="" 
 authors="eamonoreilly" 
 manager="jwinter" 
 editor=""/>

<tags 
wacn.date="05/15/2015"
 ms.service="backup" 
 ms.workload="storage-backup-recovery" 
 ms.tgt_pltfrm="na" 
 ms.devlang="na" 
 ms.topic="article" 
 ms.date="04/13/2015" 
 ms.author="eamono"/>


# 使用 Azure Automation 管理 Azure 备份

本指南将介绍 Azure Automation 服务，以及如何使用它来简化 Azure 备份的管理。

## 什么是 Azure Automation？

[Azure Automation](/home/features/automation) 是用于通过流程自动化简化云管理的一项 Azure 服务。使用 Azure Automation 可以自动完成那些长时间运行、人工操作、易出错和经常重复的任务，从而改善组织的可靠性、效率和价值生成时间。

Azure Automation 提供高度可靠且高度可用的工作流执行引擎，它可以随着组织的发展，根据你的需求扩展。在 Azure Automation 中，流程可以手动、通过第三方系统或按计划的间隔启动，使任务能够完全根据需求进行。

通过将云管理任务改为由 Azure Automation 自动运行，可以降低运营开销，解放 IT/开发运营人员，让他们将精力集中在增加企业价值的工作上。 


## Azure Automation 如何帮助管理 Azure 备份？

可以使用 [Windows MSOnlineBackup 模块](https://technet.microsoft.com/zh-cn/library/hh770400.aspx)中提供的 PowerShell cmdlet 在 Azure Automation 中管理备份。Azure Automation 可以调用这些 PowerShell cmdlet，因此，你可以在该服务中执行所有备份管理任务。你还可以将 Azure Automation 中的这些 cmdlet 与其他 Azure 服务的 cmdlet 搭配使用，以自动完成跨 Azure 服务和第三方系统的复杂任务。


## 后续步骤

在了解 Azure Automation 以及如何使用它来管理 Azure 备份的基础知识后，请使用以下链接了解有关 Azure Automation 的更多信息。

* 查看 Azure Automation [入门指南](/documentation/articles/automation-create-runbook-from-samples)

<!--HONumber=53-->