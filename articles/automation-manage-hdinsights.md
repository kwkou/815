<properties
	pageTitle="使用 Azure Automation 管理 Azure HDInsight"
	description="了解如何使用 Azure Automation 服务来管理 Azure HDInsight。"
	services="HDInsight, automation"
	documentationCenter=""
	authors="elcooper"
	manager="eamono"
	editor=""/>

<tags
	ms.service="HDInsight"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="04/13/2015"
	wacn.date="05/15/2015"
	ms.author="elcooper"/>



# 使用 Azure Automation 管理 Azure HDInsight
本指南介绍 Azure Automation 服务，以及如何使用它来简化群集管理和自动化 Azure HDInsight 中的常见任务。

## 什么是 Azure Automation？
[Azure Automation](/home/features/automation/) 是用于通过流程自动化简化云管理的一项 Azure 服务。使用 Azure Automation 可以自动完成那些人工操作、经常重复、长时间运行且易出错的任务，从而改善组织的可靠性、效率和价值生成时间。

Azure Automation 提供了具有高可靠性和高可用性的工作流执行引擎，该引擎可以根据你的需求进行扩展。在 Azure Automation 中，流程可以手动、通过第三方系统或按计划的间隔启动，使任务能够完全根据需求进行。

通过将云管理任务安排为由 Azure Automation 自动运行，可以降低运营开销，解放 IT 和开发运营人员，让他们将精力集中在增加企业价值的工作上。


## Azure Automation 如何帮助管理 Azure HDInsight？

可以使用 [Azure PowerShell 工具](https://msdn.microsoft.com/zh-CN/library/azure/jj156055.aspx) 中提供的 [Azure HDInsight cmdlet](https://msdn.microsoft.com/zh-CN/library/azure/dn479228.aspx) 在 Azure Automation 中管理 HDInsight。Azure Automation 现成地提供了这些 cmdlet，因此，你可以在该服务中执行所有 HDInsight 管理任务。你还可以将 Azure Automation 中的这些 cmdlet 与其他 Azure 服务的 cmdlet 搭配使用，以自动完成跨 Azure 服务和第三方系统的复杂任务。

使用 Azure HDInsight cmdlet 可以自动完成许多任务，例如，在 Linux 或 Windows 上设置 HDInsight 群集、缩放群集、管理群集和提交 MapReduce 作业。这只是你可以在 Azure Automation 中使用 PowerShell 自动完成的众多任务中的一小部分。  


## 后续步骤
在了解 Azure Automation 以及如何使用它来管理 Azure HDInsight 的基础知识后，请使用此链接了解有关 Azure Automation 的更多信息。

* 参阅 Azure Automation [入门教程](/documentation/articles/automation-create-runbook-from-samples)。


<!--HONumber=53-->