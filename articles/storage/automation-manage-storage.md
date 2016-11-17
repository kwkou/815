<properties
	pageTitle="使用 Azure 自动化管理 Azure 存储空间"
	description="了解如何使用 Azure 自动化服务来方便管理 Azure 存储空间。"
	services="storage, automation"
	documentationCenter=""
	authors="jodoglevy"
	manager="eamono"
	editor=""/>

<tags
	ms.service="storage"
	ms.workload="storage"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="05/23/2016"
	wacn.date="07/18/2016"
	ms.author="jolevy"/>



#使用 Azure 自动化管理 Azure 存储空间

本指南介绍 Azure 自动化服务，以及如何使用它来简化 Azure 存储 Blob、表和队列的管理。


## 什么是 Azure 自动化？

[Azure 自动化](/home/features/automation/)是通过流程自动化来简化云管理的一项服务。Azure 自动化可将耗时、耗力、易出错、重复性大的任务自动化，更可靠、更有效的帮助企业提升价值。

Azure 自动化提供高度可靠且高度可用的工作流执行引擎，它可以随着组织的发展，根据你的需求扩展。在 Azure 自动化中，流程可以手动、通过第三方系统或按计划的间隔启动，使任务能够完全根据需求进行。

Azure 自动化可自动运行云管理任务，降低运营开销，让 IT/开发运营人员集中精力增加企业价值。


## Azure 自动化如何帮助管理 Azure 存储空间？

[Azure PowerShell](https://msdn.microsoft.com/zh-CN/library/azure/jj156055.aspx)中提供的 Powershell cmdlets 命令可用来管理 Azure 自动化里的 Azure 存储空间。Azure 自动化 现成地提供了这些 Storage PowerShell 命令，因此，你可以在该服务中执行所有 Blob、表和队列管理任务。你还可以将 Azure 自动化 中的这些 命令 与其他 Azure 服务的 命令 搭配使用，自动完成跨 Azure 服务和第三方系统的复杂任务。

[Azure 自动化 Runbook 库](https://azure.microsoft.com/blog/2014/10/07/introducing-the-azure-automation-runbook-gallery/)包含产品团队和社区提供的各种 Runbook，帮助自动管理 Azure 存储空间、其他 Azure 服务和第三方系统。库中 Runbook 的功能包括：

 * [使用自动化服务删除达到特定天数的 Azure 存储空间 Blob](https://gallery.technet.microsoft.com/scriptcenter/Remove-Storage-Blobs-that-aae4b761)
 * [从 Azure 存储空间下载 Blob](https://gallery.technet.microsoft.com/scriptcenter/a-Blob-from-Azure-Storage-6bc13745)
 * [为单个 Azure VM 或云服务中的所有 VM 备份所有磁盘](https://gallery.technet.microsoft.com/scriptcenter/Backup-all-disks-for-a-ede940d5)


## 后续步骤

在了解 Azure 自动化 以及如何使用它来管理 Azure 存储 Blob、表和队列的基础知识后，请使用以下链接了解有关 Azure 自动化的更多信息。

请参阅 Azure 自动化教程[在 Azure 自动化中创建或导入 Runbook](/documentation/articles/automation-creating-importing-runbook/)
 

<!---HONumber=Mooncake_0405_2016-->