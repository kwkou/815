<properties
   pageTitle="Service Fabric 群集资源管理器 - 移动成本 | Azure"
   description="Service Fabric 服务的移动成本概述"
   services="service-fabric"
   documentationCenter=".net"
   authors="masnider"
   manager="timlt"
   editor=""/>

<tags
   ms.service="Service-Fabric"
   ms.date="05/20/2016"
   wacn.date="07/04/2016"/>

# 影响资源管理器选择的服务移动成本
我们在尝试判断要对群集进行哪些更改和指定解决方案的分数时要纳入考虑的另一个重要因素是达成该解决方案的整体成本。

移动服务实例或副本将 CPU 时间和网络带宽的成本降到最低，而对于有状态服务，则还体现在需要关闭旧副本之前创建状态副本的磁盘上产生空间量的成本。显然你想要将所有和资源管理器一起出现的解决方案的成本降到最低，但也不想忽略可大幅改善群集中资源分配的解决方案。

Service Fabric 资源管理器有两种方式来计算成本和限制它们。首先，当资源管理器正在规划群集的新布局时，它将计算其所进行的每一项单次移动。有一个简单的情况是，如果最终获取两个具备几乎同一整体平衡（分数）的解决方案，则请获取成本最低（移动总数）的那一个。在我们遇到默认或静态负载的相同问题之前，这可以运行得很好 – 在任何复杂的系统中，不太可能所有移动都一样，有些可能昂贵许多。

## 更改副本的移动成本和考虑因素
就像报告负载（群集资源管理器的另一个功能）一样，我们可让服务在任何指定的时间自我报告移动服务所需的成本。

代码

```csharp
this.ServicePartition.ReportMoveCost(MoveCost.Medium);
```

MoveCost 有四个级别：零、低、中和高。再次提醒 – 这些只是相对于彼此，但“零”除外，“零”表示移动副本是免费的，不应算入解决方案的分数中。将移动成本设置为“高”*不*保证副本不移动，只是除非有移动的好理由，否则我们不移动它。

![选择要移动的副本时考虑到移动成本因素][Image1]

移动成本可帮助我们在达成对等的平衡时，查找整体导致最少中断的解决方案。服务的成本概念可相对于许多事项，但计算移动成本时的最常见考虑因素包括：

1.	服务必须移动的状态或数据量
2.	客户端断开连接的成本（因此，移动主副本的成本高于移动辅助副本的成本）
3.	中断某些正在运行操作的成本（某些数据存储级别操作的成本较高，而在某个特计时点之后，不希望丢弃它们（如果不必这样做））。因此，在操作持续期间，可以提高成本以降低服务副本或实例移动的可能性，然后，当操作完成之后，可以让它恢复正常。

## 后续步骤
- 指标是 Service Fabric 群集资源管理器在群集中管理消耗和容量的方式。若要详细了解指标及其配置方式，请查看[此文](/documentation/articles/service-fabric-cluster-resource-manager-metrics/)
- 若要了解群集资源管理器如何管理和平衡群集中的负载，请查看关于[平衡负载](/documentation/articles/service-fabric-cluster-resource-manager-balancing/)的文章

[Image1]: ./media/service-fabric-cluster-resource-manager-movement-cost/service-most-cost-example.png

<!---HONumber=Mooncake_0627_2016-->