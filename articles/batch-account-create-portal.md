<properties
	pageTitle="创建 Azure Batch 帐户 | Azure"
	description="了解如何在 Azure 门户中创建 Azure Batch 帐户，以便在云中运行大规模并行工作负荷"
	services="batch"
	documentationCenter=""
	authors="dlepow"
	manager="timlt"
	editor=""/>

<tags
	ms.service="batch"
	ms.date="05/12/2016"
	wacn.date="07/15/2016"/>



# 在 Azure 门户中创建和管理 Azure Batch 帐户

> [AZURE.SELECTOR]
- [Azure 门户](/documentation/articles/batch-account-create-portal/)
- [Batch Management .NET](/documentation/articles/batch-management-dotnet/)

[Azure 门户][azure_portal]提供了创建和管理 Azure Batch 帐户所需的工具，你可将这些工具用于大规模的并行工作负荷处理。在本文中，我们将使用经典门户逐步说明如何创建 Batch 帐户，以及讨论 Batch 帐户最重要的几项设置与属性。例如，使用 Batch 开发的应用程序和服务需要帐户的 URL 和访问密钥（在 Azure 门户中可找到这两项）才能与 Batch 服务 API 进行通信。

>[AZURE.NOTE] Azure 新门户很快会上线，它支持 Batch 服务所提供的部分功能，包括创建帐户及管理帐户设置和属性。开发人员可通过 Batch API 获取 Batch 的完整功能（包括创建和运行作业与任务）。

## 在经典门户中创建批处理帐户

1. 登录到 [Azure 门户][azure_portal]。

2. 单击“新建”>“计算”>“Batch 服务”>“快速创建”，输入“账户名称”，选择“区域”。

     ![创建账户][3]

5. 单击“创建账户”以创建帐户。

  门户将指示它“正在创建”帐户，“Batch 帐户”边栏选项卡将在完成时显示。

## 查看 Batch 帐户属性

Batch 帐户边栏选项卡显示帐户的多个属性，并且可让你访问其他设置，例如访问密钥、配额、用户和存储帐户关联。

* **Batch 帐户 URL** -- 当你使用 [Batch REST][api_rest] API 或 [Batch .NET][api_net] 客户端库时，此 URL 可用于访问 Batch 帐户，并遵循以下格式：

 		 `https://<account_name>.<region>.batch.chinacloudapi.cn`

* **访问密钥** -- 若要查看及管理 Batch 帐户的访问密钥，请单击打开“管理访问密钥”。与 Batch 服务 API（例如 [Batch REST][api_rest] 或 [Batch .NET][api_net] 客户端库）通信时，需要有访问密钥。

 ![批处理帐户密钥][account_keys]



## Batch 帐户注意事项

* 你也可以使用 [Batch PowerShell cmdlet](/documentation/articles/batch-powershell-cmdlets-get-started/) 和 [Batch Management .NET](/documentation/articles/batch-management-dotnet/) 库来创建和管理 Batch 帐户。

* 你不需要支付 Batch 帐户本身的费用。你需要支付 Batch 解决方案所用 Azure 计算资源的费用，以及工作负荷运行时其他服务所用资源的费用。例如，你需要支付池中计算节点的费用，而如果使用[应用程序包](/documentation/articles/batch-application-packages)功能，则需支付用于存储应用程序包版本的 Azure 存储空间资源费用。有关详细信息，请参阅 [Batch pricing（Batch 定价）][batch_pricing]。

* 你可以在单个 Batch 帐户中运行多个 Batch 工作负荷，或者在不同 Azure 区域的 Batch 帐户之间分散工作负荷。

* 如果你正在运行多个大规模 Batch 工作负荷，请注意适用于 Azure 订阅和每个 Batch 帐户的特定 [Batch 服务配额与限制](/documentation/articles/batch-quota-limit/)。Batch 帐户的当前配额显示在门户上的帐户属性中。


> [AZURE.IMPORTANT] Batch 目前“仅”支持**常规用途**存储帐户类型，如 [About Azure storage accounts（关于 Azure 存储帐户）](/documentation/articles/storage-create-storage-account/)的 [Create a storage account（创建存储帐户）](/documentation/articles/storage-create-storage-account/#create-a-storage-account)中步骤 5 所述。将某个 Azure 存储帐户链接到你的 Batch 帐户时，“只会”链接**常规用途**的存储帐户。

## 后续步骤

* 若要深入了解 Batch 服务的概念和功能，请参阅 [Azure Batch feature overview（Azure Batch 功能概述）](/documentation/articles/batch-api-basics/)。本文讨论主要 Batch 资源（例如池、计算节点、作业和任务），并提供能够进行大规模计算工作负荷执行的服务功能概述。

* 了解使用 [Batch .NET 客户端库](/documentation/articles/batch-dotnet-get-started/)开发支持 Batch 的应用程序的基本概念。[简介文章](/documentation/articles/batch-dotnet-get-started/)介绍了使用 Batch 服务在多个计算节点上执行工作负荷的可行应用程序，并说明如何使用 Azure 存储空间进行工作负荷文件暂存和检索。

[api_net]: https://msdn.microsoft.com/library/azure/mt348682.aspx
[api_rest]: https://msdn.microsoft.com/library/azure/Dn820158.aspx

[azure_portal]: https://manage.windowsazure.cn/
[batch_pricing]: /pricing/details/batch/

[3]: ./media/batch-account-create-portal/batch_acct_01.png "Azure 门户中的创建 Batch 服务边栏选项卡"
[account_keys]: ./media/batch-account-create-portal/account_key.PNG

<!---HONumber=Mooncake_0530_2016-->