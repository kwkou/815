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
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="08/19/2016"
   wacn.date="10/24/2016"
   ms.author="masnider"/>  



# 限制 Service Fabric 群集资源管理器的行为
即使已正确配置群集资源管理器，但群集有时也会中断。例如，可能同时发生节点或容错域故障 - 升级时如果发生这种情况，该怎么样？ 资源管理器将尽力修复一切，但像这种时候，可能要考虑设置一个停止机制，让群集本身有机会能够稳定下来（将节点恢复正常、网络可以自行恢复正常、部署已更正的位）。为了帮助实现这些目的， Service Fabric 群集资源管理器提供了多个限制机制。请注意，这些限制有相当大的干扰性，通常不应使用，除非已进行一些谨慎的数学计算来解决实际上可在群集中完成的并行工作量，以及经常需要响应这些种类的未经规划且大型的重新配置事件（也称为“非常糟糕的日子”）。

一般而言，我们建议通过其他选项来避免非常糟糕的日子（例如，一般代码更新，以及避免过度排程要启动的群集），而不是为群集设置限制，以防止它在尝试修复其本身时使用资源）。限制确实有默认值，我们也已经通过经验发现默认值适合使用，但是或许应该看一下并根据预期的实际负载来调整它们。尽管最佳实践是不要过度限制或载入群集，但可能确定有些情况（直到解决它们为止）需要在适当的地方拥有多个限制，即使这意味着群集需要较长的时间才能稳定也一样。

##配置限制
下面是默认包含的限制：

-	GlobalMovementThrottleThreshold – 控制一段时间内群集中移动的总数（已定义为 GlobalMovementThrottleCountingInterval，以秒为单位的值）
-	MovementPerPartitionThrottleThreshold – 控制一段时间内针对任何服务分区区的移动总数（MovementPerPartitionThrottleCountingInterval，以秒为单位的值）


	<Section Name="PlacementAndLoadBalancing">
	     <Parameter Name="GlobalMovementThrottleThreshold" Value="1000" />
	     <Parameter Name="GlobalMovementThrottleCountingInterval" Value="600" />
	     <Parameter Name="MovementPerPartitionThrottleThreshold" Value="50" />
	     <Parameter Name="MovementPerPartitionThrottleCountingInterval" Value="600" />
	</Section>


请注意，在大多数情况下，我们看到客户使用这些限制的原因是由于他们已经处于资源受限的环境（例如对个别节点的网络带宽受到限制，或节点上已经置入不符合并行副本构建要求的磁盘），这意味着此类型操作不成功或运行速度很慢。在这些情况下，客户已充分了解可能需要延长群集到达稳定状态所需花费的时间量，包括了解在为其设置限制时，它们最终以较低的整体可靠性来运行。

## 后续步骤
- 若要了解群集资源管理器如何管理和均衡群集中的负载，请查看有关[均衡负载](/documentation/articles/service-fabric-cluster-resource-manager-balancing/)的文章
- 群集资源管理器提供许多用于描述群集的选项。若要详细了解这些选项，请查看这篇有关[描述 Service Fabric 群集](/documentation/articles/service-fabric-cluster-resource-manager-cluster-description/)的文章

<!---HONumber=Mooncake_1017_2016-->