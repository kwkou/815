<properties
    pageTitle="Service Fabric 群集资源管理器：移动成本 | Azure"
    description="Service Fabric 服务的移动成本概述"
    services="service-fabric"
    documentationcenter=".net"
    author="masnider"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="f022f258-7bc0-4db4-aa85-8c6c8344da32"
    ms.service="Service-Fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="01/05/2017"
    wacn.date="02/20/2017"
    ms.author="masnider" />  


# 影响群集资源管理器选项的服务移动成本
在尝试确定要对群集进行哪些更改时，Service Fabric Cluster Resource Manager 会考虑的一个重要因素是实现该解决方案所需的总体成本。“成本”这一概念是针对能够实现的余额而言。

移动服务实例或副本可将 CPU 时间和网络带宽的成本降到最低。对于有状态服务，则还体现在需要关闭旧副本之前创建状态副本的磁盘上和内存中产生空间量的成本。显然我们想要将所有与 Azure Service Fabric 群集资源管理器一起出现的解决方案的成本降到最低。但同时，也不希望忽略可大幅改善群集中资源分配的解决方案。

群集资源管理器提供两种方式来计算和限制成本，即使同时根据其他目标尝试管理群集也没有问题。首先，当群集资源管理器正在规划群集的新布局时，将会计算其所进行的每次移动。如果生成的两种解决方案的余额（评分）相同，则成本（总移动数）低的解决方案更好。

此策略很适用。但是，对于默认负载或静态负载，在任何复杂的系统中不太可能所有移动都一样。有些移动的成本可能要昂贵得多。

## 更改副本的移动成本和考虑因素
至于报告负载（群集资源管理器的另一功能），服务可以随时动态自我报告移动的成本。

代码：


	this.ServicePartition.ReportMoveCost(MoveCost.Medium);

也可以在创建服务时指定默认的移动成本。

MoveCost 有四个级别：“零”、“低”、“中”和“高”。MoveCost 是相互的，但零除外。“零”移动成本表示移动不会产生成本，不应计入解决方案的分数。将移动成本设置为“高”*并不能* 确保副本始终呆在一个位置。

<center> 
![选择要移动的副本时考虑到移动成本因素][Image1] 
</center>

MoveCost 可帮助我们在达成对等的均衡时，查找整体导致最少中断且最容易实现的解决方案。服务的成本概念可以相对于许多事项。计算移动成本时的最常见因素包括：

* 服务必须移动的状态或数据量。
* 客户端断开连接的成本。移动主副本的成本通常高于移动辅助副本的成本。
* 中断某些进行中操作的成本。某些数据存储级别的操作，或者为了响应客户端调用而执行的操作，成本都很高。在特定的时间点后，除非有必要，否则我们不会停止这些操作。因此，当操作正在进行时，提高该服务对象的移动成本可以降低其移动的可能性。当操作完成之后，可以将成本重新设置为正常。

## 后续步骤
- Service Fabric 群集资源管理器使用指标管理群集中的消耗和容量。有关指标以及如何配置指标的详细信息，请查看[在 Service Fabric 中使用指标管理资源消耗和负载](/documentation/articles/service-fabric-cluster-resource-manager-metrics/)
- 有关群集资源管理器如何在群集中管理负载和均衡负载的详细信息，请查看[均衡 Service Fabric 群集](/documentation/articles/service-fabric-cluster-resource-manager-balancing/)。

[Image1]: ./media/service-fabric-cluster-resource-manager-movement-cost/service-most-cost-example.png

<!---HONumber=Mooncake_0213_2017-->
<!--Update_Description: wording update-->