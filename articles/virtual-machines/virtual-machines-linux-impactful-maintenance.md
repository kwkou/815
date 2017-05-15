<properties
    pageTitle="Azure 中 Linux VM 的 VM 重启维护 | Azure"
    description="Linux 虚拟机的 VM 重启维护。"
    services="virtual-machines-linux"
    documentationcenter=""
    author=""
    manager="timlt"
    editor=""
    tags="azure-service-management,azure-resource-manager" />
<tags
    ms.assetid="eb4b92d8-be0f-44f6-a6c3-f8f7efab09fe"
    ms.service="virtual-machines-windows"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-linux"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/27/2017"
    wacn.date="05/15/2017"
    ms.author=""
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="c97775b11b0a0f495c44b4a0f67646b5d58df296"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="vm-restarting-maintenance"></a>VM 重启维护

在少数情况下，VM 会由于对底层基础结构的计划内维护而重启。 由于这些计划内维护会影响 Azure 中托管的 VM 的可用性，现在会提供以下各项供你使用：

-   在产生影响前至少 30 天发送的通知。

-   对每个 VM 的维护时段的可见性。

-   设置维护精确时间以影响 VM 的灵活性和控制力。

Azure 中的维护以迭代方式进行计划。 初始迭代范围较小，目的是降低推出新修补程序和功能所涉及的风险。 后续迭代可以跨越多个区域（永远不会来自同一区域对）。 一个 VM 包含在单个维护迭代中。 如果迭代中止，剩余的 VM 将包含在未来的其他迭代中。

计划内维护迭代有两个阶段：强占式维护时段和计划维护时段。

使用**强占式维护时段**可在 VM 上灵活启动维护。 这样，可以确定 VM 受影响的时间、更新顺序和维护每个 VM 的间隔时间。 你可以查询每个 VM 以了解它是否已计划进行维护，并可查看最后启动的维护请求的结果。

**计划的维护时段**是 Azure 为 VM 计划的维护时间。 在这段时间（紧随强占式维护时段之后）内，仍可以查询维护时段，但不能再安排维护。

## <a name="availability-considerations-during-planned-maintenance"></a>计划内维护期间的可用性注意事项 

### <a name="paired-regions"></a>配对区域

每个 Azure 区域与同一地理位置中另一个区域配对。 执行维护时，Azure 将只更新区域对中单个区域的虚拟机实例。 例如，更新中国北部的虚拟机时，Azure 不会同时更新中国东部的任何虚拟机。 后者会安排在其他时间进行，以便在区域之间进行故障转移或负载均衡。

### <a name="single-instance-vms-vs-availability-aet-or-vm-scale-set"></a>单实例 VM 与可用性集或 VM 规模集

部署使用 Azure 中的虚拟机的工作负荷时，可以在可用性集中创建 VM，以向应用程序提供高可用性。 这种配置可确保在发生故障或维护事件期间，至少有一个虚拟机可用。

在可用性集中，个别 VM 可分布在最多 20 个更新域中。 在计划内维护期间，仅一个更新域会在任意指定时间受影响。 在计划内维护期间，受影响更新域的顺序可能不会按原来的顺序保持下去。 对于单实例 VM（不是可用性集的一部分），无法预测或确定同时受影响的 VM 及其数量。

虚拟机规模集是一种 Azure 计算资源，支持将一组相同的 VM 作为单个资源进行部署和管理。
规模集采用更新域的形式向可用性集提供相似的保证。 

有关配置虚拟机以实现高可用性的详细信息，请参阅[*管理 Windows 虚拟机的可用性*](/documentation/articles/virtual-machines-linux-manage-availability/)。

### <a name="scheduled-events"></a>计划事件

使用 Azure 元数据服务可以发现有关 Azure 中托管的虚拟机的信息。 计划事件（已公开的类别之一）显示有关即将发生的事件（例如，重新启动）的信息，使应用程序可以为其做准备并限制中断。

有关计划事件的详细信息，请参阅 [Azure 元数据服务 - 计划事件](/documentation/articles/virtual-machines-scheduled-events/)。

## <a name="maintenance-discovery-and-notifications"></a>维护发现和通知

客户可以在个体 VM 级别看到维护计划。 可以使用 Azure 门户预览、API、PowerShell 或 CLI 查询强占式维护时段和计划内维护时段。 此外，在此过程中，预计将在一个（或多个）VM 受影响时收到通知（电子邮件）。

强占式维护和计划的维护阶段均以某个通知开始。 每个 Azure 订阅预计将收到一条通知。 默认情况下，通知将发送给订阅的管理员和共同管理员。 你还可以为维护通知配置受众。

### <a name="view-the-maintenance-window-in-the-portal"></a>在门户中查看维护时段 

可以使用 Azure 门户预览来查找计划进行维护的 VM。

1.  登录 Azure 门户预览。

2.  单击并打开“虚拟机”边栏选项卡。

3.  单击“列”按钮，以打开可从中进行选择的可用列的列表

4.  选择并添加“维护时段”列。 计划进行维护的 VM 将显示维护时段。 维护完成或中止后，将不再显示维护时段。

### <a name="query-maintenance-details-using-the-azure-api"></a>使用 Azure API 查询维护详细信息

使用[获取 VM 信息 API](https://docs.microsoft.com/zh-cn/rest/api/compute/virtualmachines/virtualmachines-get) 并查找实例视图，以发现单个 VM 的维护详细信息。 响应包括以下元素：

- isCustomerInitiatedMaintenanceAllowed：指示现在是否可以在 VM 上启动强占式重新部署。

- preMaintenanceWindowStartTime：强占式维护时段的开始时间。

- preMaintenanceWindowEndTime：强占式维护时段的结束时间。 过了这段时间，将无法再在此 VM 上启动维护。

- maintenanceWindowStartTime：会影响 VM 的计划的维护时段的开始时间。

- maintenanceWindowEndTime：计划的维护时段的结束时间。

- lastOperationResultCode：最后一次维护-重新部署操作的结果。

- lastOperationMessage：描述最后一次维护-重新部署操作结果的消息。

## <a name="pre-emptive-redeploy"></a>强占式重新部署

通过强占式重新部署操作，可灵活控制将维护应用于 Azure 中的 VM 的时间。 计划内维护从强占式维护时段开始，在此时段内可以确定每个 VM 重启的确切时间。 以下是这样的功能发挥效用的用例：

-   维护需要与最终用户的需求协调一致。

-   Azure 提供的计划的维护时段不够用。
    这有可能是因为该时段发生在一周内最繁忙的时间，或者是它的时间太长。

-   对于多实例或多层应用程序，两个 VM 之间需要留出足够长的时间，或者需要遵循特定的顺序。

当在 VM 上调用强占式重新部署时，它将该 VM 移动到已经更新的节点，然后重新打开它，这可保留所有配置选项和关联的资源。 与此同时，临时磁盘将丢失，并将更新与虚拟网络接口关联的动态 IP 地址。

每个 VM 启用一次强占式重新部署。 如果在此过程中出现错误，将中止操作，VM 不受影响，并将从计划内维护迭代中排除。 系统稍后会与你联系并提供新的计划，并且会重新为你提供机会来安排和决定对各个 VM 的影响顺序。