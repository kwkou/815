<properties
	pageTitle="使用 Azure 自动化管理 Azure 密钥保管库 | Azure"
	description="了解如何使用 Azure 自动化服务来管理 Azure 密钥保管库。"
	services="Key-Vault, automation"
	documentationCenter=""
	authors="csand-msft"
	manager="eamono"
	editor=""/>

<tags
	ms.service="key-vault"
	ms.date="04/18/2016"
	wacn.date="06/27/2016"/>



#使用 Azure 自动化管理 Azure 密钥保管库

本指南介绍 Azure 自动化服务，以及如何使用它来简化 Azure 密钥保管库的管理。

## 什么是 Azure 自动化？

[Azure 自动化](/home/features/automation)是用于通过流程自动化和所需的状态配置简化云管理的一项 Azure 服务。使用 Azure 自动化可以自动完成那些人工操作、重复、长时间运行且易出错的任务，从而改善组织的可靠性、效率和价值生成时间。

Azure 自动化提供了具有高可靠性和高可用性的工作流执行引擎，该引擎可以根据你的需求进行扩展。在 Azure 自动化中，流程可以手动、通过第三方系统或按计划的间隔启动，使任务能够完全根据需求进行。

通过将云管理任务改为由 Azure 自动化自动运行，可以降低运营开销，解放 IT 和开发运营人员，让他们将精力集中在增加企业价值的工作上。


## Azure 自动化如何帮助管理 Azure 密钥保管库？

可以使用 [AzureRM 密钥保管库 cmdlet](https://www.powershellgallery.com/packages/AzureRM.KeyVault/1.1.4) 和 [Azure 经典密钥保管库 cmdlet](https://msdn.microsoft.com/zh-cn/library/azure/dn868052.aspx) 在 Azure 自动化中管理密钥保管库。Azure 自动化中自动提供管理经典密钥保管库所需的 Azure 模块，因此你可以将 [AzureRM-KeyVault 模块](https://www.powershellgallery.com/packages/AzureRM.KeyVault/1.1.4)导入 Azure 自动化中，以便在服务中执行多种密钥保管库管理任务。你还可以将 Azure 自动化中的这些 cmdlet 与其他 Azure 服务的 cmdlet 搭配使用，以自动完成跨 Azure 服务和第三方系统的复杂任务。

使用 Azure 密钥保管库 cmdlet 可以与其他任务一起执行以下任务：创建和配置密钥保管库、创建或导入密钥、创建或更新机密、更新密钥属性、获取密钥或机密、删除密钥或机密。

下面是使用 PowerShell 管理密钥保管库的一些示例：
* [Azure Key Vault - Step by Step（Azure 密钥保管库 - 分步指南）](https://blogs.technet.microsoft.com/kv/2015/06/02/azure-key-vault-step-by-step)
* [Setting Up and Configuring an Azure Key Vault（设置和配置 Azure 密钥保管库）](https://www.simple-talk.com/cloud/platform-as-a-service/setting-up-and-configuring-an-azure-key-vault)


## 后续步骤

在了解 Azure 自动化 以及如何使用它来管理 Azure 密钥保管库的基础知识后，请使用以下链接了解有关 Azure 自动化的更多信息。

* 请参阅 [Azure 密钥保管库 PowerShell 脚本](https://gallery.technet.microsoft.com/scriptcenter/Azure-Key-Vault-Powershell-1349b091)。

<!---HONumber=Mooncake_0620_2016-->
