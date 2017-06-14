<properties
    pageTitle="在 Azure 门户中创建批处理帐户 | Azure"
    description="了解如何在 Azure 门户中创建 Azure Batch 帐户，以便在云中运行大规模并行工作负荷"
    services="batch"
    documentationcenter=""
    author="tamram"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="3fbae545-245f-4c66-aee2-e25d7d5d36db"
    ms.service="batch"
    ms.workload="big-compute"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.date="03/27/2017"
    ms.author="tamram"
    ms.custom="H1Hack27Feb2017"
    wacn.date="05/15/2017"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="3ff18e6f95d8bbc27348658bc5fce50c3320cf0a"
    ms.openlocfilehash="9f054f55b43d6382fe550400f90e7ea41b4aa38f"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/15/2017" />

# <a name="create-a-batch-account-with-the-azure-portal"></a>使用 Azure 门户创建 Batch 帐户
> [AZURE.SELECTOR]
- [Azure 门户](/documentation/articles/batch-account-create-portal/)
- [Batch Management .NET](/documentation/articles/batch-management-dotnet/)

了解如何在 [Azure 门户][azure_portal]中创建 Azure 批处理帐户，以及如何选择适合计算方案的帐户属性。 了解在何处查找重要的帐户属性，例如访问密钥和帐户 URL。 

有关批处理帐户和方案的背景，请参阅[功能概述](/documentation/articles/batch-api-basics/)。

## <a name="create-a-batch-account"></a>创建批处理帐户
1. 登录到 [Azure 门户][azure_portal]。
2. 单击“新建” > “计算” > “批处理服务”。
   
    ![应用商店中的批处理][marketplace_portal]
3. 将显示“新建 Batch 帐户”  边栏选项卡。 请查看下面的针对每个边栏选项卡元素的说明。
   
    ![创建批处理帐户][account_portal]
   
    a.在“解决方案资源管理器”中，右键单击项目文件夹下的“引用”文件夹，然后单击“添加引用”。 **帐户名称**：所选批处理帐户名称必须在创建帐户的 Azure 区域中唯一（参见下面的“位置”）。 帐户名只能包含小写字符或数字，且长度必须为 3-24 个字符。
   
    b.保留“数据库类型”设置，即设置为“共享”。 **订阅**：要在其中创建批处理帐户的订阅。 如果只有一个订阅，则默认选择此项。
  
    c. **资源组**：为新批处理帐户选择现有的资源组，或选择创建一个新组。
   
    d.单击“下一步”。 **位置**：要在其中创建批处理帐户的 Azure 区域。 只有订阅和资源组支持的区域显示为选项。
   
    e.在“新建 MySQL 数据库”边栏选项卡中，接受法律条款，然后单击“确定”。 **存储帐户**（可选）：与批处理帐户关联的通用 Azure 存储帐户。 建议大多数批处理帐户采用此设置。 如需更多详细信息，请参阅本文后面的[关联的 Azure 存储帐户](#linked-azure-storage-account)。

4. 单击“创建”  以创建帐户。
   
   门户指示部署正在进行。 完成后，将会在“通知”中显示“部署成功”通知。
   


## <a name="view-batch-account-properties"></a>查看 Batch 帐户属性
创建帐户后，即可打开 **Batch 帐户边栏选项卡** 来访问其设置和属性。 可以使用 Batch 帐户边栏选项卡的左侧菜单访问所有帐户设置和属性。

![Azure 门户中的 Batch 帐户边栏选项卡][account_blade]

- **批处理帐户 URL**：通过[批处理 API](/documentation/articles/batch-apis-tools/#batch-development-apis/) 开发应用程序时，需要帐户 URL 才能访问批处理资源。 Batch 帐户 URL 采用以下格式：
  
    `https://<account_name>.<region>.batch.azure.com`

    ![门户中的 Batch 帐户 URL][account_url]

- **访问密钥**（“批处理服务”模式）：从应用程序访问批处理帐户时，若要进行身份验证，需提供帐户访问密钥。 （此设置不适用于“用户订阅”模式，该模式使用 Azure Active Directory 身份验证。）

    若要查看或重新生成批处理帐户的访问密钥，请在“批处理帐户”边栏选项卡左侧菜单的“搜索”框中输入 `keys`，然后选择“密钥”。 
  
    ![Azure 门户中的 Batch 帐户密钥][account_keys]

[AZURE.INCLUDE [batch-pricing-include](../../includes/batch-pricing-include.md)]

## <a name="linked-azure-storage-account"></a>链接的 Azure 存储帐户

可以选择性地将通用 Azure 存储帐户关联到批处理帐户。 与[批处理文件约定 .NET](/documentation/articles/batch-task-output/) 库一样，批处理的[应用程序包](/documentation/articles/batch-application-packages/)功能使用 Azure Blob 存储。 这些可选功能可用于部署批处理任务运行的应用程序，以及保存它们生成的数据。

建议创建批处理帐户专用的新存储帐户。

![创建“常规用途”存储帐户][storage_account]

> [AZURE.NOTE] 
> Azure 批处理目前仅支持常规用途的存储帐户类型。 有关此帐户类型，请参见[关于 Azure 存储帐户](/documentation/articles/storage-create-storage-account/)的步骤 5：[创建存储帐户](/documentation/articles/storage-create-storage-account/#create-a-storage-account/)。
>
>

> [AZURE.WARNING]
> 重新生成链接存储帐户的访问密钥时，请多加小心。 只能重新生成一个存储帐户密钥，然后单击链接的存储帐户边栏选项卡上的“同步密钥”  。 等待五分钟，让密钥传播到池中的计算节点，然后重新生成并同步其他密钥（如有必要）。 如果同时重新生成这两个密钥，计算节点将无法同步任何一个密钥，并且无法访问存储帐户。
> 
> 

![重新生成存储帐户密钥][4]

## <a name="batch-service-quotas-and-limits"></a>Batch 服务配额和限制
请知悉，与 Azure 订阅和其他 Azure 服务一样，Batch 帐户也适用特定 [配额和限制](/documentation/articles/batch-quota-limit/) 。 Batch 帐户的当前配额在门户上的帐户“属性” 中显示。

![Azure 门户中的 Batch 帐户配额][quotas]



此外，其中许多配额只需在 Azure 门户中提交免费产品支持请求即可增加。 有关请求增加配额的详细信息，请参阅 [Azure Batch 服务的配额和限制](/documentation/articles/batch-quota-limit/) 。

## <a name="other-batch-account-management-options"></a>其他 Batch 帐户管理选项
除了使用 Azure 门户以外，还可以使用以下工具创建和管理 Batch 帐户：

- [Batch PowerShell cmdlet](/documentation/articles/batch-powershell-cmdlets-get-started/)
- [Azure CLI](/documentation/articles/batch-cli-get-started/)
- [Batch Management .NET](/documentation/articles/batch-management-dotnet/)

## <a name="next-steps"></a>后续步骤
- 请参阅[批处理功能概述](/documentation/articles/batch-api-basics/)，详细了解处理服务的概念和功能。 本文讨论主要 Batch 资源（例如池、计算节点、作业和任务），并提供能够进行大规模计算工作负荷执行的服务功能概述。
- 了解使用[批处理 .NET 客户端库](/documentation/articles/batch-dotnet-get-started/)或 [Python](/documentation/articles/batch-python-tutorial/) 开发支持批处理的应用程序的基本概念。 这些简介文章介绍了使用批处理服务在多个计算节点上执行工作负荷的可行应用程序，并说明了如何使用 Azure 存储进行工作负荷文件暂存和检索。

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
[subscription_access]: ./media/batch-account-create-portal/subscription_iam.png
[add_permission]: ./media/batch-account-create-portal/add_permission.png
[account_portal_byos]: ./media/batch-account-create-portal/batch_acct_portal_byos.png

<!---Update_Description: wording update -->