<properties
   pageTitle="Service Fabric 群集资源管理器 - 放置策略 | Azure"
   description="概述 Service Fabric 服务的其他放置策略和规则"
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


# Service Fabric 服务的放置策略
其他许多不同的规则最终可能让用户关心 Service Fabric 群集是否跨越地理距离（例如多个数据中心或 Azure 区域），或者环境是否跨越多个地缘政治控制区域（或者关心其他一些法律或政策边界问题，或者这种距离确实会造成性能/延迟影响）。其中的大多数规则都能通过节点属性和放置约束来配置，但有一些规则比较复杂。为方便起见，我们提供了这些附加的命令。与其他放置约束一样，可以根据每个命名服务实例来设置放置策略。

## 指定无效域
InvalidDomain 放置策略可让你指定特定的容错域对此工作负荷是无效的。此策略可确保特定的服务绝对不会在特定的区域中运行（例如，出于地缘政治或公司政策的原因）。可以通过单独的策略指定多个无效域。

![无效域示例][Image1]  


代码：


	ServicePlacementInvalidDomainPolicyDescription invalidDomain = new ServicePlacementInvalidDomainPolicyDescription();
	invalidDomain.DomainName = "fd:/DCEast"; //regulations prohibit this workload here
	serviceDescription.PlacementPolicies.Add(invalidDomain);


Powershell：


	New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName –Stateful -MinReplicaSetSize 2 -TargetReplicaSetSize 3 -PartitionSchemeSingleton -PlacementPolicy @("InvalidDomain,fd:/DCEast”)

## 指定所需域
所需域放置策略要求服务的所有有状态副本或无状态服务实例出现在指定的域中。可以通过独立的策略指定多个所需域。

![所需域示例][Image2]  


代码：

	ServicePlacementRequiredDomainPolicyDescription requiredDomain = new ServicePlacementRequiredDomainPolicyDescription();
	requiredDomain.DomainName = "fd:/DC01/RK03/BL2";
	serviceDescription.PlacementPolicies.Add(requiredDomain);


Powershell：


	New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName –Stateful -MinReplicaSetSize 2 -TargetReplicaSetSize 3 -PartitionSchemeSingleton -PlacementPolicy @("RequiredDomain,fd:/DC01/RK03/BL2")


## 指定主副本的首选域
首选主域是一个有趣的控件，因为它允许选择容错域，如果能够这样做，主副本就必须放置在该域中。如果一切运行正常，主副本最终将在此域中。如果域或主副本发生故障，或出于某种原因而关闭，则主副本将迁移到其他某个位置。如果此位置不在首选域中，群集资源管理器将在可行的情况下将它移回到首选域。当然，此设置仅适用于有状态服务。此策略最适合用于跨 Azure 区域或多个数据中心的群集。在这种场合下，用户想要使用所有位置实现冗余，但希望主副本放置在特定的位置，以便为转到主副本的操作（写入操作，以及默认情况下的所有读取操作由主副本处理）提供较低的延迟。

![首选主域和故障转移][Image3]  


	ServicePlacementPreferPrimaryDomainPolicyDescription primaryDomain = new ServicePlacementPreferPrimaryDomainPolicyDescription();
	primaryDomain.DomainName = "fd:/ChinaEast/";
	serviceDescription.PlacementPolicies.Add(invalidDomain);


Powershell：


	New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName –Stateful -MinReplicaSetSize 2 -TargetReplicaSetSize 3 -PartitionSchemeSingleton -PlacementPolicy @("PreferredPrimaryDomain,fd:/ChinaEast")


## 要求在所有域之间分布副本，并且不允许打包
可以指定的另一个策略是要求始终在可用的容错域之间分布副本。默认情况下，这会在多数群集状况良好的情况下发生，不过仍然可能有特定分区最终被暂时打包成单个容错域或升级域的例外情况。例如，假设群集在 3 个容错域（0、1 和 2）中有 9 个节点，服务有 3 个副本，容错域 1 和 2 中用于这些副本的节点已关闭，并且由于容量问题，使得这些域中的其他任何节点都无效。如果 Service Fabric 要为这些副本构建替代项，群集资源管理器要求将它们放置在容错域 0 中，但这又违反了容错域约束。同时，这也会增加丢失整个副本集的可能性（如果永久丢失 FD 0 的话）。（有关约束和约束优先级的其他一般信息，请参阅[此主题](/documentation/articles/service-fabric-cluster-resource-manager-management-integration/#constraint-priorities)）

如果看到类似于“负载均衡器检测到此副本的约束违规：fabric:/<某个服务名称> Secondary Partition <某个分区ID> 已违反约束：FaultDomain”，即表示已触发此条件或类似的条件。这种情况通常是暂时性的（节点不会长时间关闭，或者在节点持续关闭并需要构建替代项的情况下，在正确的容错域中还是有其他有效节点），但仍然有某些工作负荷愿意以失去所有副本的风险去交换可用性。可以通过指定“RequireDomainDistribution”策略来实现此目的，该策略可保证在相同的容错域或升级域中，永远不存在两个来自相同分区的副本。

某些工作负荷偏向于随时都有目标数量的副本（状态副本），针对整个域发生故障做出赌注，因为它们知道通常可以恢复本地状态；有些工作负荷则偏向于提前停机，而不愿承受准确性和数据丢失等风险。由于大多数生产工作负荷运行超过 3 个副本，因此，默认设置是不需要域分发，而是让均衡与故障转移能够正常处理这些情况，即使这意味着域暂时具有多个要打包到其中的副本。

代码：


	ServicePlacementRequireDomainDistributionPolicyDescription distributeDomain = new ServicePlacementRequireDomainDistributionPolicyDescription();
	serviceDescription.PlacementPolicies.Add(distributeDomain);


Powershell：


	New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName –Stateful -MinReplicaSetSize 2 -TargetReplicaSetSize 3 -PartitionSchemeSingleton -PlacementPolicy @("RequiredDomainDistribution")


现在，是否可针对未跨越地理区域的群集中的服务使用这些配置？ 当然可以！ 但也没有充分的理由 – 除非实际上正在运行跨越地理区域的群集，否则，特别要避免所需、无效和首选域配置 - 强制特定工作负荷在单个机架上运行，或者优先使用本地群集上某些段并没有太大的意义，除非存在不同类型的硬件或工作负荷段，但这种情况可以通过普通的放置约束来处理。

## 后续步骤
- 有关可用于配置服务的其他选项的详细信息，请查看 [Learn about configuring Services](/documentation/articles/service-fabric-cluster-resource-manager-configure-services/)（了解如何配置服务）中提供的其他群集资源管理器配置的相关主题

[Image1]: ./media/service-fabric-cluster-resource-manager-advanced-placement-rules-placement-policies/cluster-invalid-placement-domain.png
[Image2]: ./media/service-fabric-cluster-resource-manager-advanced-placement-rules-placement-policies/cluster-required-placement-domain.png
[Image3]: ./media/service-fabric-cluster-resource-manager-advanced-placement-rules-placement-policies/cluster-preferred-primary-domain.png

<!---HONumber=Mooncake_1017_2016-->