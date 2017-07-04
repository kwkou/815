<properties
    pageTitle="Service Fabric 群集资源管理器 - 放置策略 | Azure"
    description="概述 Service Fabric 服务的其他放置策略和规则"
    services="service-fabric"
    documentationcenter=".net"
    author="masnider"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="5c2d19c6-dd40-4c4b-abd3-5c5ec0abed38"
    ms.service="Service-Fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="01/05/2017"
    wacn.date="02/20/2017"
    ms.author="masnider" />  


# Service Fabric 服务的放置策略
在极少数情况下，可能需要配置其他许多不同的规则。这些情况可能包括：
* Service Fabric 群集跨越地理区域（例如多个本地数据中心）或跨越 Azure 区域的情况
* 环境跨越多个地缘控制区域的情况，或者在意法律或政策边界问题的其他某些情况
* 由于群集跨越较远距离或通过某些较慢或较不可靠的网络进行通信，因此确实会造成性能/延迟影响。

在这些情况下，给定服务在特定区域始终运行或从不运行可能至关重要。同样，尝试在特定区域放置主要副本以最大限度缩短最终用户延迟可能也很重要。

高级放置策略为：

1. 无效域
2. 所需域
3. 首选域
4. 不允许副本打包

以下大部分控制条件都能通过节点属性和放置约束进行配置，但某些比较复杂。为方便起见，Service Fabric 群集资源管理器提供了这些附加的放置策略。与其他放置约束一样，可以根据每个命名服务实例来配置放置策略，并进行动态更新。

## 指定无效域
InvalidDomain 放置策略允许指定特定的容错域对此工作负荷无效。此策略可确保特定的服务绝对不会在特定的区域中运行（例如，出于地缘政治或公司政策的原因）。可以通过单独的策略指定多个无效域。

<center> 
![无效域示例][Image1] 
</center>

代码：


	ServicePlacementInvalidDomainPolicyDescription invalidDomain = new ServicePlacementInvalidDomainPolicyDescription();
	invalidDomain.DomainName = "fd:/DCEast"; //regulations prohibit this workload here
	serviceDescription.PlacementPolicies.Add(invalidDomain);


Powershell：


	New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName –Stateful -MinReplicaSetSize 2 -TargetReplicaSetSize 3 -PartitionSchemeSingleton -PlacementPolicy @("InvalidDomain,fd:/DCEast”)

## 指定所需域
所需域放置策略要求服务的所有有状态副本或无状态服务实例出现在指定的域中。可以通过单独的策略指定多个所需域。

<center> 
![所需域示例][Image2] 
</center>

代码：

	ServicePlacementRequiredDomainPolicyDescription requiredDomain = new ServicePlacementRequiredDomainPolicyDescription();
	requiredDomain.DomainName = "fd:/DC01/RK03/BL2";
	serviceDescription.PlacementPolicies.Add(requiredDomain);


Powershell：


	New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName –Stateful -MinReplicaSetSize 2 -TargetReplicaSetSize 3 -PartitionSchemeSingleton -PlacementPolicy @("RequiredDomain,fd:/DC01/RK03/BL2")


## 指定主要副本的首选域
首选主域是一个有趣的控制条件，因为它允许选择容错域，如果能够这样做，主要副本就应放置在该域中。如果一切运行正常，主要副本最终位于此域中。如果域或主要副本发生故障，或出于某种原因而关闭，则主要副本会迁移到其他某个位置。如果此新位置不在首选域中，群集资源管理器会尽快将主要副本移回首选域。当然，此设置仅适用于有状态服务。此策略最适合用于跨 Azure 区域或多个数据中心，但希望主要副本放置在特定位置的群集。主要副本保持接近其用户有助于降低延迟，尤其是读取延迟。

<center> 
![首选主域和故障转移][Image3] 
</center>

	ServicePlacementPreferPrimaryDomainPolicyDescription primaryDomain = new ServicePlacementPreferPrimaryDomainPolicyDescription();
	primaryDomain.DomainName = "fd:/ChinaEast/";
	serviceDescription.PlacementPolicies.Add(invalidDomain);


Powershell：


	New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName –Stateful -MinReplicaSetSize 2 -TargetReplicaSetSize 3 -PartitionSchemeSingleton -PlacementPolicy @("PreferredPrimaryDomain,fd:/ChinaEast")


## 要求在所有域之间分布副本，并且不允许打包
副本通常在群集状况良好的情况下跨域分布，但仍然可能有特定分区的副本最终被暂时打包成单个域的情况。例如，假设群集在 3 个容错域（fd:/0、fd:/1 和 fd:/2）中有 9 个节点，服务有 3 个副本。假设 fd:/1 和 fd:/2 中用于这些副本的节点已关闭。现在，群集资源管理器往往更倾向于使用相同容错域中的其他节点。在这种情况下，假设由于容量问题，这些域中的其他任何节点都无效。如果群集资源管理器要为这些副本生成替代项，则必须在 fd:/0 中选择节点。但是，执行此操作又违反了容错域约束。同时，这也会增加关闭或丢失整个副本集的可能性（如果永久丢失 FD 0 的话）。有关约束和约束优先级的其他一般信息，请参阅[此主题](/documentation/articles/service-fabric-cluster-resource-manager-management-integration/#constraint-priorities)。

如果看到类似于 `The Load Balancer has detected a Constraint Violation for this Replica:fabric:/<some service name> Secondary Partition <some partition ID> is violating the Constraint: FaultDomain` 的运行状况警告，即表示已触发此条件或类似的条件。这种情况很少发生，一旦发生通常也是暂时性的，节点恢复后就会停止。如果节点持续关闭，并且群集资源管理器需要生成替代项，说明正确的容错域中还有其他有效节点。

某些工作负荷偏向于维持副本的目标数量，即使这些副本打包成了更少的域。这些工作负荷打赌整个域不会同时发生永久故障，并且通常可以恢复本地状态。其他工作负荷则偏向于提前停机，而不愿承受准确性和数据丢失等风险。由于大多数生产工作负荷运行超过 3 个副本、3 个以上容错域，并且每容错域运行多个有效节点，因此默认设置是不要求域分发。而是让通常的均衡与故障转移处理这些情况，即使这意味着域暂时具有多个要打包到其中的副本。

若要针对给定工作负荷禁用此类打包，可对该服务指定“RequireDomainDistribution”策略。设置此策略后，群集资源管理器可保证在相同的容错域或升级域中，永远不存在两个来自相同分区的副本。

代码：


	ServicePlacementRequireDomainDistributionPolicyDescription distributeDomain = new ServicePlacementRequireDomainDistributionPolicyDescription();
	serviceDescription.PlacementPolicies.Add(distributeDomain);


Powershell：


	New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName –Stateful -MinReplicaSetSize 2 -TargetReplicaSetSize 3 -PartitionSchemeSingleton -PlacementPolicy @("RequiredDomainDistribution")


现在，是否可针对不跨越地理区域的群集中的服务使用这些配置？ 当然可以！ 但也没有充分的理由。除非实际上正在运行跨越地理区域的群集，否则，应避免所需、无效和首选域配置。强制特定工作负荷在单个机架上运行，或者优先使用本地群集上某些段并没有太大意义。不同的硬件配置应分布在各个域，这些域通过普通放置约束和节点属性进行处理。

## 后续步骤
* 若要深入了解可用于配置服务的其他选项，请转到[了解如何配置服务](/documentation/articles/service-fabric-cluster-resource-manager-configure-services/)

[Image1]: ./media/service-fabric-cluster-resource-manager-advanced-placement-rules-placement-policies/cluster-invalid-placement-domain.png
[Image2]: ./media/service-fabric-cluster-resource-manager-advanced-placement-rules-placement-policies/cluster-required-placement-domain.png
[Image3]: ./media/service-fabric-cluster-resource-manager-advanced-placement-rules-placement-policies/cluster-preferred-primary-domain.png

<!---HONumber=Mooncake_0213_2017-->
<!--Update_Description: enrich the introduction for "Service Fabric 服务的放置策略"-->