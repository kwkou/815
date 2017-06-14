<properties
    pageTitle="Azure Batch 服务配额和限制 | Azure"
    description="了解默认的 Azure Batch 配额、限制和约束，以及如何请求提高配额"
    services="batch"
    documentationcenter=""
    author="tamram"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="28998df4-8693-431d-b6ad-974c2f8db5fb"
    ms.service="batch"
    ms.workload="big-compute"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/24/2017"
    wacn.date="05/15/2017"
    ms.author="tamram"
    ms.custom="H1Hack27Feb2017"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="d87d4761c7b83b6f107e12b924281660d6b033f3"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="batch-service-quotas-and-limits"></a>Batch 服务配额和限制

与其他 Azure 服务一样，与 Batch 服务关联的某些资源存在限制。 其中的许多限制是 Azure 在订阅或帐户级别应用的默认配额。 本文将描述这些默认值，以及如何请求提高配额。

设计和增加 Batch 工作负荷时，请记住这些配额。 例如，如果池没有达到指定的计算节点目标数量，那么可能是已达到 Batch 帐户的核心配额限制。

可以在单个批处理帐户中运行多个批处理工作负荷，或者在相同订阅的不同 Azure 区域的批处理帐户之间分散工作负荷。

如果你打算在 Batch 中运行生产工作负荷，可能需要将一个或多个配额提高到默认值以上。 如果需要提高配额，可以免费提出在线 [客户支持请求](#increase-a-quota) 。

> [AZURE.NOTE]
> 配额是一种信用限制，不附带容量保证。 如果你有大规模的容量需求，请联系 Azure 支持。
> 
> 

## <a name="resource-quotas"></a>资源配额
[AZURE.INCLUDE [azure-batch-limits](../../includes/azure-batch-limits.md)]

## <a name="other-limits"></a>其他限制
| **资源** | **最大限制** |
| --- | --- |
| 每个计算节点的[并发任务](/documentation/articles/batch-parallel-node-tasks/)数 |4 x 节点核心数 |
| 每个 Batch 帐户的[应用程序](/documentation/articles/batch-application-packages/)数 |20 |
| 每个应用程序的应用程序包数 |40 |
| 应用程序包大小（每个） |约 195GB<sup>1</sup> |

<sup>1</sup> 最大的块 Blob 大小的 Azure 存储空间限制

## <a name="view-batch-quotas"></a>查看 Batch 配额
可在 [Azure 门户][portal]中查看批处理帐户配额。

1. 在门户中选择“Batch 帐户”，然后选择所需的 Batch 帐户。
2. 在 Batch 帐户的菜单边栏选项卡中选择“属性”
3. “属性”边栏选项卡显示了当前应用于 Batch 帐户的 **配额**
   
    ![Batch 帐户配额][account_quotas]

## <a name="increase-a-quota"></a>提高配额
执行以下步骤，使用 [Azure 门户][portal]来请求提高配额。

1. 在门户仪表板上选择“帮助 + 支持”磁贴，或单击门户右上角的问号 (**?**)。
2. 选择“新建支持请求” > “基本”。
3. 在“基本”边栏选项卡上：
   
    a.在“解决方案资源管理器”中，右键单击项目文件夹下的“引用”文件夹，然后单击“添加引用”。 “问题类型” > “配额”
   
    b.保留“数据库类型”设置，即设置为“共享”。 选择你的订阅。
   
    c. “配额类型” > “Batch”
   
    d.单击“下一步”。 “支持计划” > “配额支持 - 已包括”
   
    单击“下一步”。
4. 在“问题”边栏选项卡上：
   
    a.在“解决方案资源管理器”中，右键单击项目文件夹下的“引用”文件夹，然后单击“添加引用”。 根据[业务影响情况][support_sev]选择“严重性”。
   
    b. 在“详细信息”中，指定想要更改的每个配额、Batch 帐户名和新限制。
   
    单击“下一步”。
5. 在“联系信息”边栏选项卡上：
   
    a.选择“首选联系方法”。
   
    b.保留“数据库类型”设置，即设置为“共享”。 输入并确认所需的联系人详细信息。
   
    单击“创建”提交支持请求。

提交支持请求后，Azure 支持人员将与你取得联系。 请注意，完成该请求最多需要 2 个工作日。

## <a name="related-topics"></a>相关主题
- [使用 Azure 门户创建 Azure Batch 帐户](/documentation/articles/batch-account-create-portal/)
- [Azure Batch 功能概述](/documentation/articles/batch-api-basics/)
- [Azure 订阅和服务限制、配额和约束](/documentation/articles/azure-subscription-service-limits/)

[portal]: https://portal.azure.cn
[portal_classic_increase]: https://azure.microsoft.com/zh-cn/blog/2014/06/04/azure-limits-quotas-increase-requests/
[support_sev]: http://aka.ms/supportseverity

[account_quotas]: ./media/batch-quota-limit/accountquota_portal.PNG

<!---Update_Description: wording update -->