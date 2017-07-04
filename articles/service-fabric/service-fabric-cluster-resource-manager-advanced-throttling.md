<properties
    pageTitle="Service Fabric 群集资源管理器中的限制 | Azure"
    description="了解如何配置 Service Fabric 群集资源管理器提供的限制。"
    services="service-fabric"
    documentationcenter=".net"
    author="masnider"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="4a44678b-a5aa-4d30-958f-dc4332ebfb63"
    ms.service="Service-Fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="01/05/2017"
    wacn.date="02/20/2017"
    ms.author="masnider" />  


# 限制 Service Fabric 群集资源管理器的行为
即使已正确配置群集资源管理器，群集有时也会中断。例如，可能同时发生节点或容错域故障 - 升级时发生这种情况会怎样？ 群集资源管理器会尽力修复一切，但这可能导致群集中出现变动。限制可帮助提供停止机制，让群集能够使用资源自行稳定下来，即节点恢复正常，网络分区自行修复、部署已更正的位。

为帮助应对此类情况，Service Fabric 群集资源管理器提供多种限制。这些限制十分有效。除非已仔细计算过群集可以并行完成的工作量，否则不应更改这些设置的默认值。

限制的默认值是 Service Fabric 团队从经验中得出的合适默认值。如果需要更改这些默认值，应根据预期的实际负载进行调优。即使意味着群集在主线方案中会需要更长的时间才能稳定下来，用户也可能会决定设置某些限制。

## 配置限制
下面是默认包含的限制：

* GlobalMovementThrottleThreshold - 此设置控制一段时间内群集中移动的总数（定义为 GlobalMovementThrottleCountingInterval，以秒为单位）
* MovementPerPartitionThrottleThreshold - 此设置控制一段时间内针对任何服务分区的移动总数（MovementPerPartitionThrottleCountingInterval，以秒为单位）


	<Section Name="PlacementAndLoadBalancing">
	     <Parameter Name="GlobalMovementThrottleThreshold" Value="1000" />
	     <Parameter Name="GlobalMovementThrottleCountingInterval" Value="600" />
	     <Parameter Name="MovementPerPartitionThrottleThreshold" Value="50" />
	     <Parameter Name="MovementPerPartitionThrottleCountingInterval" Value="600" />
	</Section>

通过用于独立部署的 ClusterConfig.json 或用于 Azure 托管群集的 Template.json：


	"fabricSettings": [
	  {
	    "name": "PlacementAndLoadBalancing",
	    "parameters": [
	      {
	          "name": "GlobalMovementThrottleThreshold",
	          "value": "1000"
	      },
	      {
	          "name": "GlobalMovementThrottleCountingInterval",
	          "value": "600"
	      },
	      {
	          "name": "MovementPerPartitionThrottleThreshold",
	          "value": "50"
	      },
	      {
	          "name": "MovementPerPartitionThrottleCountingInterval",
	          "value": "600"
	      }
	    ]
	  }
	]


大部分时候，客户使用这些限制是因为他们已处于资源受限环境中。此类环境的一些示例包括，每个节点的网络带宽受限，或磁盘因吞吐量限制而无法并行生成多个副本。这些类型的限制意味着，即使没有限制，针对故障触发的操作也无法成功，或会速度缓慢。在这些情况下，客户知道群集达到稳定状态的时间会延长。客户还知道，如果受到限制，他们最终会以较低的整体可靠性运行。

## 后续步骤
- 若要了解群集资源管理器如何管理和均衡群集中的负载，请查看有关[均衡负载](/documentation/articles/service-fabric-cluster-resource-manager-balancing/)的文章
- 群集资源管理器提供许多用于描述群集的选项。若要详细了解这些选项，请查看这篇有关[描述 Service Fabric 群集](/documentation/articles/service-fabric-cluster-resource-manager-cluster-description/)的文章

<!---HONumber=Mooncake_0213_2017-->
<!--Update_Description: add ClusterConfig.json sample script-->