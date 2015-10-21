<properties
	pageTitle="使用 Azure Automation 管理 Azure 存储空间"
	description="了解如何使用 Azure Automation 服务来方便管理 Azure 存储空间。"
	services="storage, automation"
	documentationCenter=""
	authors="jodoglevy"
	manager="eamono"
	editor=""/>

<tags
	ms.service="storage"
	ms.date="08/11/2015"
	wacn.date="09/18/2015"/>



# 使用 Azure 自动化 管理 Azure 存储空间

本指南介绍 Azure 自动化 服务，以及如何使用它来简化 Azure 存储 Blob、表和队列的管理。


## 什么是 Azure 自动化？

[Azure 自动化](/documentation/services/automation) 是用于通过流程自动化简化云管理的一项 Azure 服务。使用 Azure 自动化 可以自动完成那些长时间运行、人工操作、易出错和经常重复的任务，从而改善组织的可靠性、效率和价值生成时间。

Azure 自动化 提供高度可靠且高度可用的工作流执行引擎，它可以随着组织的发展，根据你的需求扩展。在 Azure 自动化 中，流程可以手动、通过第三方系统或按计划的间隔启动，使任务能够完全根据需求进行。

通过将云管理任务改为由 Azure 自动化 自动运行，可以降低运营开销，解放 IT/开发运营人员，让他们将精力集中在增加企业价值的工作上。 


## Azure 自动化 如何帮助管理 Azure 存储空间？

可以使用 [Azure PowerShell 工具](https://msdn.microsoft.com/zh-CN/library/azure/jj156055.aspx)中提供的 PowerShell 命令 在 Azure 自动化 中管理 Azure 存储空间。Azure 自动化 现成地提供了这些 Storage PowerShell 命令，因此，你可以在该服务中执行所有 Blob、表和队列管理任务。你还可以将 Azure 自动化 中的这些 命令 与其他 Azure 服务的 命令 搭配使用，以自动完成跨 Azure 服务和第三方系统的复杂任务。

[Azure Automation Runbook 库](http://azure.microsoft.com/blog/2014/10/07/introducing-the-azure-automation-runbook-gallery/)包含产品团队和社区提供的各种 Runbook，以帮助你开始自动管理 Azure 存储空间、其他 Azure 服务和第三方系统。库中 Runbook 的功能包括：

 * [删除 X 天以前的 Azure 存储空间 Blob](https://gallery.technet.microsoft.com/scriptcenter/Remove-Storage-Blobs-that-aae4b761)
 * [将 Azure 存储空间中的 Blob 下载到 Azure Automation](https://gallery.technet.microsoft.com/scriptcenter/a-Blob-from-Azure-Storage-6bc13745)
 * [在 Azure 云服务中创建 Azure VM 数据磁盘的副本](https://gallery.technet.microsoft.com/scriptcenter/Make-copies-of-Azure-VM-065a6394)


## 后续步骤

在了解 Azure 自动化 以及如何使用它来管理 Azure 存储 Blob、表和队列的基础知识后，请使用以下链接了解有关 Azure 自动化 的更多信息。

* 查看 Azure 自动化 [入门指南](/documentation/articles/automation-create-runbook-from-samples)
