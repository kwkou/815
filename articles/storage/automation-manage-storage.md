<properties
	pageTitle="使用 Azure 自动化管理 Azure 存储"
	description="了解如何使用 Azure 自动化服务来方便管理 Azure 存储。"
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
	wacn.date="12/19/2016"
	ms.author="jolevy"/>



#使用 Azure 自动化管理 Azure 存储

本指南介绍 Azure 自动化服务，以及如何使用它来简化 Azure 存储 Blob、表和队列的管理。


## 什么是 Azure 自动化？

[Azure 自动化](/home/features/automation/)是一项 Azure 服务，用于通过流程自动化简化云管理。使用 Azure 自动化可以自动完成那些长时间运行、人工操作、易出错和经常重复的任务，从而改善组织的可靠性、效率和价值生成时间。

Azure 自动化提供高度可靠且高度可用的工作流执行引擎，它可以随着组织的发展按需扩展。在 Azure 自动化中，流程可以手动、通过第三方系统或按计划的间隔启动，使任务能够完全根据需求进行。

通过将云管理任务改为由 Azure 自动化自动运行，可以降低运营开销，解放 IT/开发运营人员，让他们将精力集中在增加企业价值的工作上。


## Azure 自动化如何帮助管理 Azure 存储？

可以使用 [Azure PowerShell](https://msdn.microsoft.com/zh-cn/library/azure/jj156055.aspx) 中提供的 PowerShell cmdlet 在 Azure 自动化中管理 Azure 存储。Azure 自动化提供这些现成的 Storage PowerShell cmdlet，因此可以在该服务中执行所有 Blob、表和队列管理任务。还可将 Azure 自动化中的这些 cmdlet 与其他 Azure 服务的 cmdlet 搭配使用，以自动完成跨 Azure 服务和第三方系统的复杂任务。

[Azure 自动化 Runbook 库](https://azure.microsoft.com/blog/2014/10/07/introducing-the-azure-automation-runbook-gallery/)包含产品团队和社区提供的各种 Runbook，可用于开始自动管理 Azure 存储、其他 Azure 服务和第三方系统。库中 Runbook 的功能包括：

 * [使用自动化服务删除寿命达到特定天数的 Azure 存储 Blob](https://gallery.technet.microsoft.com/scriptcenter/Remove-Storage-Blobs-that-aae4b761)
 * [从 Azure 存储下载 Blob](https://gallery.technet.microsoft.com/scriptcenter/a-Blob-from-Azure-Storage-6bc13745)
 * [为单个 Azure VM 或云服务中的所有 VM 备份所有磁盘](https://gallery.technet.microsoft.com/scriptcenter/Backup-all-disks-for-a-ede940d5)


## 后续步骤

既已了解 Azure 自动化以及如何将其用于管理 Azure 存储 Blob、表和队列的基础知识，可使用以下链接了解有关 Azure 自动化的更多信息。

请参阅 Azure 自动化教程[在 Azure 自动化中创建或导入 Runbook](/documentation/articles/automation-creating-importing-runbook/)
 

<!---HONumber=Mooncake_Quality_Review_1202_2016-->