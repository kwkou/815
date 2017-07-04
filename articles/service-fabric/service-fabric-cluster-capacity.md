<properties
    pageTitle="规划 Service Fabric 群集容量 | Azure"
    description="Service Fabric 群集容量规划注意事项。 节点类型、持久性和可靠性层"
    services="service-fabric"
    documentationcenter=".net"
    author="ChackDan"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="4c584f4a-cb1f-400c-b61f-1f797f11c982"
    ms.service="service-fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="03/24/2017"
    wacn.date="05/15/2017"
    ms.author="chackdan"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="59fc44aa0ff6721502cd4cf554a1bae9ce953b15"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="service-fabric-cluster-capacity-planning-considerations"></a>Service Fabric 群集容量规划注意事项
对于任何生产部署，容量规划都是一个重要的步骤。 下面是在规划过程中必须注意的一些事项。

* 群集一开始需要的节点类型数目
* 每个节点类型的属性（大小、是否为主节点、是否面向 Internet、VM 数目，等等）
* 群集的可靠性和持久性特征

让我们简单地了解其中每一项。

## <a name="the-number-of-node-types-your-cluster-needs-to-start-out-with"></a>群集一开始需要的节点类型数目
首先，需要确定要创建的群集将用于什么目的，以及打算要将哪些类型的应用程序部署到此群集中。 如果不清楚群集的用途，则很可能还未准备好进入容量规划流程。

确定群集一开始需要的节点类型数目。  每个节点类型将映射到虚拟机规模集。 然后，每个节点类型可以独立扩展或缩减、打开不同的端口集，并可以有不同的容量指标。 因此，节点类型数目的确定基本上可归结为以下考虑因素：

* 应用程序是否有多个服务，其中是否有任何服务需面向公众或面向 Internet？ 典型的应用程序包含可从客户端接收输入的前端网关服务，以及一个或多个与前端服务通信的后端服务。 因此，在此情况下，必须至少有两个节点类型。
* （构成应用程序的）服务是否有不同的基础结构要求，例如，更多的 RAM 或更高的 CPU 周期？ 例如，假设要部署的应用程序包含前端服务和后端服务。 前端服务可以在容量较小（如 D2 的 VM 大小）且向 Internet 开放了端口的 VM 上运行。  但是，后端服务是计算密集型的服务，需要放在容量较大（D4、D6、D15 等的 VM 大小）且不面向 Internet 的 VM 上运行。
  
  在本示例中，可以决定将所有服务都放在一个节点类型上，但我们建议将它们分别放在群集中的两个节点类型上。  这样，每个节点类型的属性（如 Internet 连接或 VM 大小）都会有所不同。 也可以单独缩放 VM 的数目。  
* 由于无法预见未来，因此，请利用所知道的事实，确定应用程序一开始需要的节点类型数目。 以后，随时可以添加或删除节点类型。 Service Fabric 群集必须至少有一个节点类型。

## <a name="the-properties-of-each-node-type"></a>每个节点类型的属性
**节点类型** 相当于云服务中的角色。 节点类型定义 VM 大小、VM 数目及其属性。 在 Service Fabric 群集中定义的每个节点类型将设置为不同的虚拟机规模集 (VMSS)。 VMSS 是一种 Azure 计算资源，可用于将一组虚拟机作为一个集进行部署和管理。 如果定义为不同的 VMSS，则每个节点类型可以独立扩展或缩减、打开不同的端口集，并可以有不同的容量指标。

有关节点类型与 VMSS 之间的关系、如何 RDP 到某个实例、打开新端口等方面的更多详细信息，请阅读[此文档](/documentation/articles/service-fabric-cluster-nodetypes/)。

群集可以有多个节点类型，但主节点类型（在门户定义的第一个节点类型）必须至少有 5 个 VM 供群集用于生产工作负荷（或至少有 3 个 VM 用于测试群集）。 如果要使用 Resource Manager 模板创建群集，可以在节点类型定义下找到 **is Primary** 属性。 主节点类型是 Service Fabric 系统服务所在的节点类型。  

### <a name="primary-node-type"></a>主节点类型
对于包含多个节点类型的群集，必须选择其中一个作为主节点类型。 下面是主节点类型的特征：

* 主节点类型的 **VM 大小下限**取决于你选择的**持久性层**。 持久性层的默认值为 Bronze。 有关持久性层的定义以及可采用的值的详细信息，请向下滚动。  
* 主节点类型的 **VM 数目下限**取决于你选择的**可靠性层**。 可靠性层的默认值为 Silver。 向下滚动，可查看有关可靠性层的定义以及可采用的值的详细信息。 

 

* Service Fabric 系统服务（例如，群集管理器服务或映像存储服务）放在主节点类型上，因此，群集的可靠性和持久性取决于你为主节点类型选择的可靠性层值与持久性层值。

![显示具有两个节点类型的群集的屏幕截图 ][SystemServices]

### <a name="non-primary-node-type"></a>非主节点类型
包含多个节点类型的群集有一个主节点类型，其余的则是非主节点类型。 下面是非主节点类型的特征：

* 此节点类型的 VM 大小下限取决于选择的持久性层。 持久性层的默认值为 Bronze。 向下滚动，可查看有关持久性层的定义以及可采用的值的详细信息。  
* 此节点类型的 VM 数目下限可以是 1。 但是，应该根据想要在此节点类型中运行的应用程序/服务的副本数目选择此数目。 部署群集之后，节点类型中的 VM 数目可能会增加。

## <a name="the-durability-characteristics-of-the-cluster"></a>群集的持久性特征
持久性层用于向系统指示 VM 对于基本 Azure 基础结构拥有的权限。 在主节点类型中，此权限可让 Service Fabric 暂停影响系统服务及有状态服务的仲裁要求的任何 VM 级别基础结构请求（例如，VM 重启、VM 重置映像或 VM 迁移）。 在非主节点类型中，此权限可让 Service Fabric 暂停影响其中运行的有状态服务的仲裁要求的任何 VM 级别基础结构请求，例如，VM 重启、VM 重置映像、VM 迁移，等等。

此权限表示为以下值：

- Gold - 每个 UD 可持续暂停基础结构作业 2 小时

- Silver - 每个 UD 可持续暂停基础结构作业 30 分钟（当前未启用。启用后，将在所有单核及更高配置的标准 VM 上提供）。

- Bronze - 无权限。这是默认值。

## <a name="the-reliability-characteristics-of-the-cluster"></a>群集的可靠性特征
可靠性层用于设置想要在此群集中的主节点类型上运行的系统服务副本数。 副本数越大，群集中的系统服务越可靠。  

可靠性层可以采用以下值。

* Platinum - 运行包含 9 个目标副本集的系统服务
* Gold - 运行包含 7 个目标副本集的系统服务
* Silver - 运行包含 5 个目标副本集的系统服务
* Bronze - 运行包含 3 个目标副本集的系统服务

> [AZURE.NOTE]
> 选择的可靠性层决定了主节点类型必须具有的最少节点数。 可靠性层与群集大小上限没有关系。 因此，可以在 Bronze 可靠性层运行包含 20 个节点的群集。
> 
> 

 随时可以选择将群集的可靠性从一个层更新为另一个层。 这样做会触发更改系统服务副本集计数所需的群集升级。 等待升级完成，然后对群集做出其他任何更改，如添加节点，等等。可以在 Service Fabric Explorer 中运行 [Get-ServiceFabricClusterUpgrade](https://msdn.microsoft.com/zh-cn/library/mt126012.aspx) 来监视升级进度


## <a name="primary-node-type---capacity-guidance"></a>主节点类型 - 容量指导

下面是有关规划主节点类型的容量的指导

1. **VM 实例数：** 若要在 Azure 中运行任何生产工作负荷，必须指定 Silver 或更高的可靠性层，这就相当于主节点类型的大小至少为 5。
2. **VM SKU：** 主节点类型是运行系统服务的位置，因此，在为它选择 VM SKU 时，必须考虑到计划在其中放置的总峰值负载。 下面这个类比用来说明此处我要表达的意思 - 将主节点类型想象成你的“肺”，它向你的大脑提供氧气，如果大脑不能获得足够的氧气，你的身体将会痛苦。 

群集的容量需求完全由计划在群集中运行的工作负荷决定。因此，我们无法为具体的工作负荷提供量化的指导，但是，我们提供了一个大致的指导来帮助你开始规划容量

对于生产工作负荷 


- 建议的 VM SKU 为标准 D3 或标准 D3_V2，或者相当于一块至少 14 GB 的本地 SSD 的容量。
- 支持使用的最小 VM SKU 为标准 D1 或标准 D1_V2，或者相当于一块至少 14 GB 的本地 SSD 的容量。 
- 生产工作负荷不支持不完整的核心 VM SKU，例如标准 A0。
- 具体而言，出于性能原因，生产工作负荷不支持标准 A1 SKU。


## <a name="non-primary-node-type---capacity-guidance-for-stateful-workloads"></a>非主节点类型 - 有状态工作负荷的容量指导

请阅读下面有关使用 Service Fabric Reliable Collections 或 Reliable Actors 的工作负荷的信息。 阅读有关[编程模型](/documentation/articles/service-fabric-choose-framework/)的更多内容。

1. **VM 实例数：**对于有状态生产工作负荷，建议使用最小值和目标副本计数 5 运行它们。 这意味着，在稳定的状态下，最终在每个容错域和升级域中都将具有一个副本（来自副本集）。 系统服务的整体可靠性层概念事实上只是为系统服务指定此设置的一种方式。

因此，对于生产工作负荷，如果在节点中运行有状态工作负荷，建议使用的最小非主节点类型大小为 5。

2. **VM SKU：**这是运行你的应用程序服务的节点类型，因此，你为其选择的 VM SKU 必须将你计划在每个节点中放置的峰值负荷考虑在内。 群集类型的容量需求是绝对由你计划在群集中运行的工作负荷决定的，因此，我们无法针对你的具体工作负荷为你提供定性指导，但是下面的宽泛指导可帮助你入门。

对于生产工作负荷 

- 建议的 VM SKU 为标准 D3 或标准 D3_V2，或者相当于一块至少 14 GB 的本地 SSD 的容量。
- 支持使用的最小 VM SKU 为标准 D1 或标准 D1_V2，或者相当于一块至少 14 GB 的本地 SSD 的容量。 
- 生产工作负荷不支持不完整的核心 VM SKU，例如标准 A0。
- 具体而言，出于性能原因，生产工作负荷不支持标准 A1 SKU。


## <a name="non-primary-node-type---capacity-guidance-for-stateless-workloads"></a>非主节点类型 - 无状态工作负荷的容量指导

对于无状态工作负荷，请阅读以下内容。

**VM 实例数：**对于无状态生产工作负荷，支持的最小非主节点类型大小为 2。 这允许你运行应用程序的两个无状态实例，并允许你的服务在某个 VM 实例丢失的情况下继续运行。 

> [AZURE.NOTE]
> 当你的群集在低于 5.6 的 Service Fabric 版本上运行时，由于运行时中的一个缺陷（计划在 5.6 中修复），如果将非主节点类型缩减为小于 5，则会导致群集运行状况变为不正常，直到你使用相应的节点名称调用 [Remove-ServiceFabricNodeState cmd](https://docs.microsoft.com/zh-cn/powershell/servicefabric/vlatest/Remove-ServiceFabricNodeState)。 
> 
>

**VM SKU：**这是运行你的应用程序服务的节点类型，因此，你为其选择的 VM SKU 必须将你计划在每个节点中放置的峰值负荷考虑在内。 群集类型的容量需求是绝对由你计划在群集中运行的工作负荷决定的，因此，我们无法针对你的具体工作负荷为你提供定性指导，但是下面的宽泛指导可帮助你入门。

对于生产工作负荷 


- 建议的 VM SKU 为标准 D3、标准 D3_V2 或相当的容量。 
- 支持使用的最小 VM SKU 为标准 D1、标准 D1_V2 或相当的容量。 
- 生产工作负荷不支持不完整的核心 VM SKU，例如标准 A0。
- 具体而言，出于性能原因，生产工作负荷不支持标准 A1 SKU。

<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->

## <a name="next-steps"></a>后续步骤
完成容量规划并设置群集后，请阅读以下文章：

* [Service Fabric 群集安全性](/documentation/articles/service-fabric-cluster-security/)
* [Service Fabric 运行状况模型简介](/documentation/articles/service-fabric-health-introduction/)
* [节点类型与 VMSS 之间的关系](/documentation/articles/service-fabric-cluster-nodetypes/)

<!--Image references-->
[SystemServices]: ./media/service-fabric-cluster-capacity/SystemServices.png
<!--Update_Description:wording update;add anchors to subtitles-->