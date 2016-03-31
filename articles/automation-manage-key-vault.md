<properties
	pageTitle="使用 Azure 自动化管理 Azure 密钥保管库 | Azure"
	description="了解如何使用 Azure Automation 服务来管理 Azure 密钥保管库。"
	services="Key-Vault, automation"
	documentationCenter=""
	authors="csand-msft"
	manager="eamono"
	editor=""/>

<tags
	ms.service="key-vault"
	ms.date="09/22/2015"
	wacn.date="12/28/2015"/>



#使用 Azure Automation 管理 Azure 密钥保管库

本指南介绍 Azure Automation 服务，以及如何使用它来简化 Azure 密钥保管库的管理。

## 什么是 Azure Automation？

[Azure 自动化](/home/features/automation/)是用于通过流程自动化简化云管理的一项 Azure 服务。使用 Azure Automation 可以自动完成那些人工操作、经常重复、长时间运行且易出错的任务，从而改善组织的可靠性、效率和价值生成时间。

Azure Automation 提供了具有高可靠性和高可用性的工作流执行引擎，该引擎可以根据你的需求进行扩展。在 Azure Automation 中，流程可以手动、通过第三方系统或按计划的间隔启动，使任务能够完全根据需求进行。

通过将云管理任务改为由 Azure Automation 自动运行，可以降低运营开销，解放 IT 和开发运营人员，让他们将精力集中在增加企业价值的工作上。


## Azure Automation 如何帮助管理 Azure 密钥保管库？

可以使用 [Azure PowerShell 工具](https://msdn.microsoft.com/zh-cn/library/azure/jj156055.aspx)中提供的 [Azure 密钥保管库 cmdlet](https://msdn.microsoft.com/zh-cn/library/azure/dn868052.aspx) 在 Azure 自动化中管理密钥保管库。Azure Automation 现成地提供了这些 cmdlet，因此，你可以在该服务中执行许多的密钥保管库管理任务。你还可以将 Azure Automation 中的这些 cmdlet 与其他 Azure 服务的 cmdlet 搭配使用，以自动完成跨 Azure 服务和第三方系统的复杂任务。

使用 Azure 密钥保管库 cmdlet 可以执行以下任务：创建或导入密钥、创建或更新机密、更新密钥属性、获取密钥或机密，或者删除密钥或机密。


## 后续步骤

在了解 Azure Automation 以及如何使用它来管理 Azure 密钥保管库的基础知识后，请使用以下链接了解有关 Azure Automation 的更多信息。

* 请参阅 Azure 自动化[入门教程](/documentation/articles/automation-create-runbook-from-samples)。
* 请参阅 [Azure 密钥保管库 PowerShell 脚本](https://gallery.technet.microsoft.com/scriptcenter/Azure-Key-Vault-Powershell-1349b091)。

<!---HONumber=Mooncake_1207_2015-->