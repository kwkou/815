<properties
	pageTitle="创建 Azure Batch 帐户 | Azure"
	description="了解如何在 Azure 门户预览中创建 Azure Batch 帐户，以便在云中运行大规模并行工作负荷"
	services="batch"
	documentationCenter=""
	authors="mmacy"
	manager="timlt"
	editor=""/>  


<tags
	ms.service="batch"
	ms.workload="big-compute"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="get-started-article"
	ms.date="09/21/2016"
	wacn.date="12/30/2016"
	ms.author="marsma"/>  


# 使用 Azure 门户预览创建 Azure Batch 帐户

> [AZURE.SELECTOR]
- [Azure 门户预览](/documentation/articles/batch-account-create-portal/)
- [Batch Management .NET](/documentation/articles/batch-management-dotnet/)

了解如何在 [Azure 门户预览][azure_portal]中创建 Azure Batch 帐户，以及在何处查找重要帐户属性，例如访问密钥和帐户 URL。本文还介绍 Batch 定价，以及如何将 Azure 存储帐户链接到 Batch 帐户，以便可以使用[应用程序包](/documentation/articles/batch-application-packages/)以及[保存作业和任务输出](/documentation/articles/batch-task-output/)。

## 创建批处理帐户

1. 登录到 [Azure 门户预览][azure_portal]。

2. 单击“新建”>“虚拟机”>“Batch 服务”。

	![应用商店中的批处理][marketplace_portal]  


3. “新建 Batch 帐户”边栏选项卡随即显示。请参阅以下 *a* 到 *e* 项，获取每个边栏选项卡元素的说明。

    ![创建批处理帐户][account_portal]  


	a.**帐户名**：Batch 帐户的唯一名称。此名称必须是创建帐户的 Azure 区域内的唯一名称（请参阅下面的“位置”）。它只能包含小写字符、数字，且长度必须为 3-24 个字符。

	b.**订阅**：要在其中创建 Batch 帐户的订阅。如果只有一个订阅，则默认选择此项。

	c.**资源组**：为新 Batch 帐户选择现有的资源组，或选择创建一个新组。

	d.**位置**：要在其中创建 Batch 帐户的 Azure 区域。只有订阅和资源组支持的区域显示为选项。

    e.**存储帐户**（可选）：要与新 Batch 帐户关联（链接）的**常规用途**存储帐户。有关详细信息，请参阅下面的[链接的 Azure 存储帐户](#linked-azure-storage-account)。

4. 单击“创建”创建帐户。

  门户将指示**正在部署**帐户，完成后，“通知”中会显示“部署成功”通知。

## 查看 Batch 帐户属性

创建帐户后，可以打开“Batch 帐户”边栏选项卡，访问其该帐户的设置和属性。可以使用“Batch 帐户”边栏选项卡中左侧的菜单访问所有帐户设置和属性。

![Azure 门户预览中的 Batch 帐户边栏选项卡][account_blade]  


* **Batch 帐户 URL**：使用 [Batch 开发 API](/documentation/articles/batch-technical-overview/#batch-development-apis/) 创建的应用程序需要使用一个帐户 URL 来管理资源以及在帐户中运行作业。Batch 帐户 URL 采用以下格式：

    `https://<account_name>.<region>.batch.azure.com`  


	![门户中的 Batch 帐户 URL][account_url]  


* **访问密钥**：应用程序在 Batch 帐户中处理资源时还需要一个访问密钥。若要查看或重新生成 Batch 帐户的访问密钥，请在“Batch 帐户”边栏选项卡左侧菜单的“搜索”框中输入 `keys`，然后选择“密钥”。

    ![Azure 门户预览中的 Batch 帐户密钥][account_keys]  


## 定价

Batch 帐户只在“免费层”中提供，这意味着无需为 Batch 帐户本身付费。需要支付 Batch 解决方案使用的底层 Azure 计算资源的费用，以及工作负荷运行时其他服务所用资源的费用。例如，需要针对池中的计算节点以及作为任务输入或输出存储在 Azure 存储中的数据付费。如果使用 Batch 的[应用程序包](/documentation/articles/batch-application-packages/)功能，则需支付用于存储应用程序包的 Azure 存储资源费用。有关详细信息，请参阅 [Batch pricing][batch_pricing]（Batch 定价）。

## 链接的 Azure 存储帐户 <a name="linked-azure-storage-account"></a>

如前所述，可以将**常规用途**存储帐户链接到 Batch 帐户（可选）。与 [Batch 文件约定 .NET](/documentation/articles/batch-task-output/)库一样，Batch 的[应用程序包](/documentation/articles/batch-application-packages/)功能在链接的常规用途存储帐户中使用 Blob 存储。这些可选功能可帮助部署 Batch 任务运行的应用程序，以及保存它们生成的数据。

Batch 目前*仅*支持**常规用途**存储帐户类型，如[关于 Azure 存储帐户](/documentation/articles/storage-create-storage-account/)的[创建存储帐户](/documentation/articles/storage-create-storage-account/#create-a-storage-account/)中步骤 5 所述。将某个 Azure 存储帐户链接到 Batch 帐户时，请确保*只*链接**常规用途**存储帐户。

![创建“常规用途”存储帐户][storage_account]  


建议创建 Batch 帐户专用的存储帐户。

>[AZURE.WARNING] 重新生成链接存储帐户的访问密钥时，请多加小心。只能重新生成一个存储帐户密钥，然后单击链接存储帐户边栏选项卡中的“同步密钥”。等待五分钟，让密钥传播到池中的计算节点，然后重新生成并同步其他密钥（如果需要）。如果同时重新生成这两个密钥，计算节点将无法同步任何一个密钥，并且无法访问存储帐户。

  ![重新生成存储帐户密钥][4]  


## Batch 服务的配额和限制

请注意，与 Azure 订阅和其他 Azure 服务一样，Batch 帐户也有特定的[配额和限制](/documentation/articles/batch-quota-limit/)。Batch 帐户的当前配额显示在门户上的帐户“属性”中。

![Azure 门户预览中的 Batch 帐户配额][quotas]  


设计和扩展 Batch 工作负荷时，请记住这些配额。例如，如果池未达到指定的计算节点目标数目，可能已达到 Batch 帐户的核心配额限制。

另请注意， Azure 订阅不受限于单个 Batch 帐户。可以在单个 Batch 帐户中运行多个 Batch 工作负荷，或者在相同订阅、但不同 Azure 区域中的 Batch 帐户之间分散工作负荷。

在 Azure 门户预览中提交免费产品支持请求，即可提高其中的多项配额。有关请求提高配额的详细信息，请参阅 [Quotas and limits for the Azure Batch service](/documentation/articles/batch-quota-limit/)（Azure Batch 服务的配额和限制）。

## 其他 Batch 帐户管理选项

除了使用 Azure 门户预览以外，还可以使用以下工具创建和管理 Batch 帐户：

* [Batch PowerShell cmdlet](/documentation/articles/batch-powershell-cmdlets-get-started/)
* [Azure CLI](/documentation/articles/xplat-cli-install/)
* [Batch Management .NET](/documentation/articles/batch-management-dotnet/)

## 后续步骤

* 若要深入了解 Batch 服务的概念和功能，请参阅 [Azure Batch feature overview](/documentation/articles/batch-api-basics/)（Azure Batch 功能概述）。本文讨论主要 Batch 资源（例如池、计算节点、作业和任务），并提供能够进行大规模计算工作负荷执行的服务功能概述。

* 了解使用 [Batch .NET 客户端库](/documentation/articles/batch-dotnet-get-started/)开发支持 Batch 的应用程序的基本概念。[简介文章](/documentation/articles/batch-dotnet-get-started/)介绍了使用 Batch 服务在多个计算节点上执行工作负荷的可行应用程序，并说明如何使用 Azure 存储进行工作负荷文件暂存和检索。

[api_net]: https://msdn.microsoft.com/zh-cn/library/azure/mt348682.aspx
[api_rest]: https://msdn.microsoft.com/zh-cn/library/azure/Dn820158.aspx

[azure_portal]: https://portal.azure.cn
[batch_pricing]: /pricing/details/batch/

[4]: ./media/batch-account-create-portal/batch_acct_04.png "重新生成存储帐户密钥"
[marketplace_portal]: ./media/batch-account-create-portal/marketplace_batch.PNG
[account_blade]: ./media/batch-account-create-portal/batch_blade.png
[account_portal]: ./media/batch-account-create-portal/batch_acct_portal.png
[account_keys]: ./media/batch-account-create-portal/account_keys.PNG
[account_url]: ./media/batch-account-create-portal/account_url.png
[storage_account]: ./media/batch-account-create-portal/storage_account.png
[quotas]: ./media/batch-account-create-portal/quotas.png

<!---HONumber=Mooncake_1107_2016-->
