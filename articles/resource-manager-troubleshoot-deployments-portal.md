<properties
   pageTitle="使用门户查看部署操作 | Azure"
   description="介绍如何使用 Azure 门户来检测 Resource Manager 部署中的错误。"
   services="azure-resource-manager,virtual-machines"
   documentationCenter=""
   tags="top-support-issue"
   authors="tfitzmac"
   manager="timlt"
   editor="tysonn"/>

<tags
   ms.service="azure-resource-manager"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="vm-multiple"
   ms.workload="infrastructure"
   ms.date="06/15/2016"
   wacn.date="07/18/2016"
   ms.author="tomfitz"/>

# 使用 Azure 门户查看部署操作

> [AZURE.SELECTOR]
- [门户](/documentation/articles/resource-manager-troubleshoot-deployments-portal/)
- [PowerShell](/documentation/articles/resource-manager-troubleshoot-deployments-powershell/)
- [Azure CLI](/documentation/articles/resource-manager-troubleshoot-deployments-cli/)
- [REST API](/documentation/articles/resource-manager-troubleshoot-deployments-rest/)

可以通过 Azure 门户查看部署操作。当你在部署过程中收到错误时，可能最想要查看操作，因此本文将重点介绍如何查看已失败的操作。该门户提供了一个界面让你轻松找到错误并确定可能的解决方法。

[AZURE.INCLUDE [resource-manager-troubleshoot-introduction](../../includes/resource-manager-troubleshoot-introduction.md)]

## 使用部署操作进行故障排除

若要查看部署操作，请使用以下步骤 ：

1. 对于部署中涉及的资源组，请注意最后一个部署的状态。你可以选择此状态以获取更多详细信息。

    ![部署状态](./media/resource-manager-troubleshoot-deployments-portal/deployment-status.png)

2. 你将看到最近的部署历史记录。选择失败的部署。

    ![部署状态](./media/resource-manager-troubleshoot-deployments-portal/select-deployment.png)

3. 选择“失败”**。单击此处查看详细信息**以查看部署失败的原因的说明。在下图中，DNS 记录不是唯一的。

    ![查看失败的部署](./media/resource-manager-troubleshoot-deployments-portal/view-error.png)

    此错误消息应足够让你可以开始进行故障排除。但是，如果你需要有关完成了哪些任务的更多详细信息，可以查看操作，如下面的步骤所示。

4. 可以在“部署”边栏选项卡中查看所有部署操作。选择任何操作，以查看更多详细信息。

    ![查看操作](./media/resource-manager-troubleshoot-deployments-portal/view-operations.png)

    在此示例中，你会看到已成功创建存储帐户、虚拟网络和可用性集。公共 IP 地址失败，未尝试其他资源。

5. 可以通过选择“事件”查看部署的事件。

    ![查看事件](./media/resource-manager-troubleshoot-deployments-portal/view-events.png)

6. 查看部署的所有事件，并选择任何事件以了解更多详细信息。

    ![查看事件](./media/resource-manager-troubleshoot-deployments-portal/see-all-events.png)

## 使用审核日志进行故障排除

[AZURE.INCLUDE [resource-manager-audit-limitations](../../includes/resource-manager-audit-limitations.md)]

若要查看部署相关的错误，请使用以下步骤：

1. 通过选择“审核日志”查看资源组的审核日志。

    ![选择审核日志](./media/resource-manager-troubleshoot-deployments-portal/select-audit-logs.png)

2. 在“审核日志”边栏选项卡中，你将看到订阅中所有资源组的最新操作摘要。其中包括表示操作的时间与状态的图形，以及操作的列表。

    ![显示操作](./media/resource-manager-troubleshoot-deployments-portal/audit-summary.png)

3. 你可以筛选审核日志的视图，以便重点关注特定的条件。选择“审核日志”边栏选项卡顶部的“筛选”。

    ![筛选日志](./media/resource-manager-troubleshoot-deployments-portal/filter-logs.png)

4. 从“筛选”边栏选项卡中选择条件，以将审核日志的视图限制为你要查看的操作。例如，你可以筛选操作，以便只显示资源组的错误。

    ![设置筛选选项](./media/resource-manager-troubleshoot-deployments-portal/set-filter.png)

5. 你可以设置时间跨度以进一步筛选操作。下图将视图筛选为包含特定的 20 分钟时间跨度的信息。

    ![设置时间](./media/resource-manager-troubleshoot-deployments-portal/select-time.png)

6. 你可以在列表中选择任意操作。选择包含你要调查的错误的操作。

    ![选择操作](./media/resource-manager-troubleshoot-deployments-portal/select-operation.png)
  
7. 你将看到该操作的所有事件。请注意摘要中的**相关 ID**。此 ID 用于跟踪相关的事件。与技术支持人员合作排查问题时，此 ID 非常有用。你可以选择任一事件，以查看有关该事件的详细信息。

    ![选择事件](./media/resource-manager-troubleshoot-deployments-portal/select-event.png)

8. 你将看到有关事件的详细信息。请特别注意“属性”，其中提供了有关错误的信息。

    ![显示审核日志详细信息](./media/resource-manager-troubleshoot-deployments-portal/audit-details.png)

下次你查看审核日志时，应用于审核日志的筛选器将会保留，因此可能需要更改这些值才能扩展操作视图。

## 后续步骤

- 有关解决特定部署错误的帮助，请参阅 [Resolve common errors when deploying resources to Azure with Azure Resource Manager（解决使用 Azure Resource Manager 将资源部署到 Azure 时的常见错误）](/documentation/articles/resource-manager-common-deployment-errors/)。
- 若要了解如何使用审核日志来监视其他类型的操作，请参阅 [Audit operations with Resource Manager（使用 Resource Manager 执行审核操作）](/documentation/articles/resource-group-audit/)。
- 若要在执行部署之前验证部署，请参阅 [Deploy a resource group with Azure Resource Manager template（使用 Azure Resource Manager 模板部署资源组）](/documentation/articles/resource-group-template-deploy/)。

<!---HONumber=Mooncake_0711_2016-->
