<properties
   pageTitle="Service Fabric 群集资源管理器中的限制 | Azure"
   description="了解如何配置 Service Fabric 群集资源管理器提供的限制。"
   services="service-fabric"
   documentationCenter=".net"
   authors="masnider"
   manager="timlt"
   editor=""/>

<tags
   ms.service="Service-Fabric"
   ms.date="05/20/2016"
   wacn.date="07/04/2016"/>


# 限制 Service Fabric 群集资源管理器的行为
有时，即使你已正确配置资源管理器，但群集还是有可能会中断（许多节点或容错域故障、正在升级、正在应用更新时等）。资源管理器将尽力修复一切，但像这种时候，你可能要考虑设置一个停止机制，让群集本身有机会能够稳定下来（将节点恢复正常、网络可以自行恢复正常、部署已更正的位）。为了帮助实现这些目的，资源管理器提供了多个限制机制。请注意，这些都是“重量级”工具，通常不应使用，除非已进行一些谨慎的数学计算来解决实际上可在群集中完成的并行工作量，以及经常需要响应这些种类的未经规划且大型的重新配置事件（也称为“非常糟糕的日子”）。

一般而言，我们建议通过其他选项来避免非常糟糕的日子（例如，一般代码更新，以及避免过度排程要启动的群集），而不是为群集设置限制，以防止它在尝试修复其本身时使用资源）。也就是说，可能判断有些情况（直到解决它们为止）需要在适当的地方拥有多个限制，即使这意味着群集需要较长的时间才能稳定也一样。

##配置限制
下面是默认包含的限制：

-	GlobalMovementThrottleThreshold – 控制一段时间内群集中移动的总数（已定义为 GlobalMovementThrottleCountingInterval，以秒为单位的值）
-	MovementPerPartitionThrottleThreshold – 控制一段时间内针对任何服务分区区的移动总数（MovementPerPartitionThrottleCountingInterval，以秒为单位的值）

``` xml
<Section Name="PlacementAndLoadBalancing">
     <Parameter Name="GlobalMovementThrottleThreshold" Value="1000" />
     <Parameter Name="GlobalMovementThrottleCountingInterval" Value="600" />
     <Parameter Name="MovementPerPartitionThrottleThreshold" Value="50" />
     <Parameter Name="MovementPerPartitionThrottleCountingInterval" Value="600" />
</Section>
```

请注意，我们在大多数时间看到的是客户使用这些限制，这是因为他们已经处于资源受限的环境中（例如，连到个别节点的有限网络带宽），这意味着此类操作失败，或者无论如何速度都变慢，并且客户已充分了解可能需要延长群集到达稳定状态所需花费的时间量，包括了解在为其设置限制时，它们最终以较低的整体可靠性来运行。

## 后续步骤
- 若要了解群集资源管理器如何管理和平衡群集中的负载，请查看关于[平衡负载](/documentation/articles/service-fabric-cluster-resource-manager-balancing/)的文章
- 群集资源管理器提供许多用于描述群集的选项。若要详细了解这些选项，请查看有关这篇[描述 Service Fabric 群集](/documentation/articles/service-fabric-cluster-resource-manager-cluster-description/)的文章

<!---HONumber=Mooncake_0627_2016-->