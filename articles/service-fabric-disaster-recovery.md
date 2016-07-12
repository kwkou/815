<properties
   pageTitle="Azure Service Fabric 灾难恢复 | Azure"
   description="Azure Service Fabric 提供所需的功能用于应对各种灾难。本文介绍可能发生的灾难类型，以及如何应对这些灾难。"
   services="service-fabric"
   documentationCenter=".net"
   authors="seanmck"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="05/25/2016"
   wacn.date="07/04/2016"/>

# Azure Service Fabric 中的灾难恢复

提供高可用性云应用程序的关键是确保它可以从各种故障中生存下来，包括那些完全不受控制的故障。本文说明在潜在灾难环境中 Azure Service Fabric 群集的物理布局，并提供有关如何应对此类灾难以便限制或消除停机或数据丢失风险的指导。

## Azure 中 Service Fabric 群集的物理布局

若要了解不同类型的故障造成的风险，知道 Azure 中群集的物理布局方式会很有帮助。

在 Azure 中创建 Service Fabric 群集时，需要选择用于托管该群集的区域。然后，Azure 基础结构将在区域中为该群集预配资源，最重要的是请求的虚拟机 (VM) 数目。让我们更深入了解这些 VM 的预配方式和位置。

### 容错域

默认情况下，群集中的 VM 平均分散在名为容错域 (FD) 的逻辑组中，逻辑组根据主机硬件中的潜在故障将计算机分段。具体而言，如果你有两个 VM 位于两个不同的 FD 中，则可以确保它们不会共享相同的电源或网络交换机。因此，影响一个 VM 的局域网或电源故障不会影响到另一个 VM，使 Service Fabric 能够重新平衡群集中无响应计算机的工作负荷。

可以使用 [Service Fabric Explorer](/documentation/articles/service-fabric-visualizing-your-cluster/) 中提供的群集映射将各容错域中群集的布局可视化：

![Service Fabric Explorer 中分散在容错域之间的节点][sfx-cluster-map]

>[AZURE.NOTE] 群集映射中的另一个轴显示升级域，升级域以逻辑方式根据计划的维护活动将节点分组。Azure 中的 Service Fabric 群集始终分布在五个升级域之间。

### 地理分布

目前中国有 2 个 Azure 区域：中国东部和中国北部。根据需求和适当位置的可用性（还有其他因素），单个区域可以包含一个或多个物理数据中心。但请注意，即使在包含多个物理数据中心的区域中，也不能保证群集的 VM 平均分散在这些物理位置。实际上，给定群集的所有 VM 目前都预配在单个物理站点中。

## 应对故障

有多种类型的故障可能会影响群集，而每种类型都有其自身的缓解方式。我们将按照发生故障的可能性大小进行探讨。

### 单个计算机故障

如前所述，单个计算机故障（在 VM 中或在容错域内托管计算机的硬件或软件中）本身不造成风险。Service Fabric 通常可在几秒内检测到故障，并根据群集的状态做出响应。例如，如果节点托管某个分区的主副本，则会从该分区的辅助副本中选择新的主副本。当 Azure 恢复有故障的计算机后，该计算机将自动重新加入群集并再次接管自身的工作负荷。

### 多个并发计算机故障

尽管容错域可大幅减少并发计算机故障的风险，但是多个随机故障始终有可能同时关闭群集中的多台计算机。

一般而言，只要大多数节点保持可用，群集就会继续运行，不过，在容量较低时，有状态副本将打包到少量的一组计算机中，并提供较少的无状态实例用于分散负载。

#### 仲裁丢失

如果有状态服务的分区的大多数副本关闭，则该分区将进入所谓的“仲裁丢失”状态。此时，Service Fabric 将停止允许写入该分区，以确保其状态保持一致且可靠。实际上，我们会选择接受一段不可用的时间，以确保客户端不被告知其数据已保存（但事实并非如此）。请注意，如果选择允许从该有状态服务的辅助副本读取，可以在此状态下继续执行读取操作。分区保持仲裁丢失的状态，直到恢复足量的副本，或群集管理员强迫系统继续使用 [Repair-ServiceFabricPartition API][repair-partition-ps] 为止。在主副本关闭时执行此操作会导致数据丢失。

系统服务也会遭受仲裁丢失，同时对相关服务造成特定影响。例如，命名服务的仲裁丢失将影响名称解析，而故障转移管理器服务的仲裁丢失将阻止创建新服务与故障转移。请注意，我们并*不*建议你像修复自己的服务一样尝试修复系统服务，而最好是单纯地等待，直到关闭的副本恢复为止。

#### 将仲裁丢失的风险降到最低

你可以通过增大服务的目标副本集大小，将仲裁丢失的风险降到最低。根据一次可容许的不可用节点数（仍可用于写入）来考虑所需的副本数会有所帮助，但请记住，除了硬件故障以外，应用程序或群集升级也可能使节点暂时不可用。

请考虑以下示例，其中假设你已将服务的 MinReplicaSetSize 配置为 3，即生产运行服务的建议最小数字。当 TargetReplicaSetSize 为 3 时（一个主副本和两个辅助副本），升级期间的硬件故障（两个副本关闭）将导致仲裁丢失，并且服务将变为只读。或者，如果有 5 个副本，则你可以在升级期间承受两次故障（三个副本关闭），因为剩余的两个副本仍可在最小的副本集中构成仲裁。

### 数据中心服务中断或损坏

在极少数情况下，物理数据中心可能会由于电源或网络连接中断而暂时不可用。在这种情况下，Service Fabric 群集和应用程序同样不可用，但会保留你的数据。对于在 Azure 中运行的群集，你可以在 [Azure 状态页][azure-status-dashboard]上查看有关中断的最新信息。

在整个物理数据中心遭到损坏（这种情况很少见）时，该数据中心托管的所有 Service Fabric 群集及其状态将会丢失。

若要防范这种可能性，请务必定期[将状态备份到](/documentation/articles/service-fabric-reliable-services-backup-restore/)异地冗余存储，并确保已验证你能够还原备份。执行备份的频率取决于恢复点目标 (RPO)。即使你尚未完全实现备份和还原，但还是应该实现 `OnDataLoss` 事件的处理程序，以便在发生该事件时进行记录，如下所示：

```c#
protected virtual Task<bool> OnDataLoss(CancellationToken cancellationToken)
{
  ServiceEventSource.Current.ServiceMessage(this, "OnDataLoss event received.");
  return Task.FromResult(true);
}
```


### 软件故障和其他数据丢失根源

在数据丢失的原因中，服务的代码缺陷、人为操作失误和安全违规比大范围的数据中心故障更为常见。但是，在所有情况下，恢复策略都一样：定期备份所有的有状态服务并运用还原该状态的能力。

## 后续步骤

- 了解如何使用[可测试性框架](/documentation/articles/service-fabric-testability-overview/)模拟各种故障
- 阅读有关灾难恢复和高可用性的其他资源。Microsoft 已发布大量有关这些主题的指导。尽管其中有些文档提到其他产品中使用的特定技术，但也包含了许多可在 Service Fabric 上下文中应用的一般性最佳实践：
 - [可用性核对清单](/documentation/articles/best-practices-availability-checklist/)
 - [执行灾难恢复演练](/documentation/articles/sql-database-disaster-recovery-drills/)



<!-- External links -->

[repair-partition-ps]: https://msdn.microsoft.com/zh-cn/library/mt163522.aspx
[azure-status-dashboard]: /support/service-dashboard/

[dr-ha-guide]: https://msdn.microsoft.com/zh-cn/library/azure/dn251004.aspx


<!-- Images -->

[sfx-cluster-map]: ./media/service-fabric-disaster-recovery/sfx-clustermap.png

<!---HONumber=Mooncake_0627_2016-->