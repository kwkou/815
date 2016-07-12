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
   ms.date="05/20/2016"
   wacn.date="07/04/2016"/>

# Service Fabric 服务的放置策略
其他许多不同的规则最终可能让你关心 Service Fabric 群集是否跨越地理距离（例如多个数据中心或 Azure 区域），或者环境是否跨越多个地缘政治控制区域（或者你关心其他一些法律或政策边界问题，或者这种距离确实会造成性能/延迟影响）。其中的大多数规则都能通过节点属性和放置约束来配置，但有一些规则比较复杂。在任何情况下，我们都可提供捷径（例如放置约束），让你根据每个命名服务实例来配置放置策略。

## 指定无效域
InvalidDomain 放置策略可让你指定特定的容错域对此工作负荷是无效的。此策略可确保特定的服务绝对不会在特定的区域中运行（例如，出于地缘政治或公司政策的原因）。可以指定多个无效域

![无效域示例][Image1]

代码：

```csharp
ServicePlacementInvalidDomainPolicyDescription invalidDomain = new ServicePlacementInvalidDomainPolicyDescription();
requiredDomain.DomainName = "fd:/DCEast/"; //regulations prohibit this workload here
serviceDescription.PlacementPolicies.Add(invalidDomain);
```

Powershell：

```posh
New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName –Stateful -MinReplicaSetSize 2 -TargetReplicaSetSize 3 -PartitionSchemeSingleton -PlacementPolicy @("InvalidDomain,fd:/dc9/”)
```
## 指定所需域
所需域放置策略要求所有副本要出现在指定的域中。可以指定多个所需域。

![所需域示例][Image2]

代码：

```csharp
ServicePlacementRequiredDomainPolicyDescription requiredDomain = new ServicePlacementRequiredDomainPolicyDescription();
requiredDomain.DomainName = "fd:/DC01/RK03/BL2";
serviceDescription.PlacementPolicies.Add(requiredDomain);
```

Powershell：

```posh
New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName –Stateful -MinReplicaSetSize 2 -TargetReplicaSetSize 3 -PartitionSchemeSingleton -PlacementPolicy @("RequiredDomain,fd:/DC01/RK03/BL2")
```

## 指定主副本的首选域
首选主域是一个有趣的控件，因为它允许选择容错域，如果能够这样做，主副本就必须放置在该域中。如果一切运行正常，主副本最终将在此域中。如果域或主副本发生故障，或出于某种原因而关闭，则主副本将迁移到其他某个位置。在适当的情况下，它将移回到首选域。当然，此设置仅适用于有状态服务。此策略最常用于跨越地理区域的群集，你将在此群集中使用其他位置实现冗余，但希望主副本放置在特定的位置，以便为转到主副本的操作（写入操作，以及默认情况下的所有读取操作由主副本处理）提供较低的延迟。

![首选主域和故障转移][Image3]

```csharp
ServicePlacementPreferPrimaryDomainPolicyDescription primaryDomain = new ServicePlacementPreferPrimaryDomainPolicyDescription();
primaryDomain.DomainName = "fd:/ChinaEast/";
serviceDescription.PlacementPolicies.Add(invalidDomain);
```

Powershell：

```posh
New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName –Stateful -MinReplicaSetSize 2 -TargetReplicaSetSize 3 -PartitionSchemeSingleton -PlacementPolicy @("PreferredPrimaryDomain,fd:/ChinaEast")
```

## 要求在所有域之间分布副本，并且不允许打包
可以指定的另一个策略是要求始终在可用的域之间分布副本。请注意，在上述示例中，很可能暂时要将多个副本（事实上是一个仲裁）打包到同一数据中心。在此情况下，我们实际上可能没问题（因为主副本并未与大多数副本位于同一个 DC 上），但这意味着我们可能暂时需要遵守一种本应避免的配置。这将取决于当时群集中的其他条件，但仍可能想要通过确保副本始终位于独立域中，来完全避免此情况。请记住，如果选择此选项，你需要选择停机时间/故障，而不是允许服务继续在剩余的节点上运行。RequireDomainDistribution 策略可配置此行为，但请注意，这可能意味着，如果最终需要打包，则我们根本不能放置副本。这意味着在可用域中的节点恢复之前，暂时将以不足够的目标副本数来运行。我们发现，某些客户希望随时具有目标副本数（状态副本），而有些客户则不希望可用性受到考验 – 如果我们将副本打包到容错域，而该域发生故障，则会同时丢失所有这些副本。如果副本数较少，可能会导致仲裁丢失。在域永远不会恢复时（这种情况极少见），这可能导致数据丢失。由于大多数用户运行超过 3 个副本，因此，默认设置是不需要域分发，而是让平衡与故障转移能够正常处理这些情况，即使这意味着域暂时具有多个要打包到其中的副本。

代码：

```csharp
ServicePlacementRequireDomainDistributionPolicyDescription distributeDomain = new ServicePlacementRequireDomainDistributionPolicyDescription();
serviceDescription.PlacementPolicies.Add(distributeDomain);
```

Powershell：

```posh
New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName –Stateful -MinReplicaSetSize 2 -TargetReplicaSetSize 3 -PartitionSchemeSingleton -PlacementPolicy @("RequiredDomainDistribution")
```

现在，是否可针对未跨越地理区域的群集中的服务使用这些配置？ 当然可以！ 但也没有充分的理由 – 除非你实际上正在运行跨越地理区域的群集，否则，特别要避免所需、无效和首选域配置。

## 后续步骤
- 有关可用于配置服务的其他选项的详细信息，请查看 [Learn about configuring Services](/documentation/articles/service-fabric-cluster-resource-manager-configure-services/)（了解如何配置服务）中提供的其他群集资源管理器配置的相关主题

[Image1]: ./media/service-fabric-cluster-resource-manager-advanced-placement-rules-placement-policies/cluster-invalid-placement-domain.png
[Image2]: ./media/service-fabric-cluster-resource-manager-advanced-placement-rules-placement-policies/cluster-required-placement-domain.png
[Image3]: ./media/service-fabric-cluster-resource-manager-advanced-placement-rules-placement-policies/cluster-preferred-primary-domain.png

<!---HONumber=Mooncake_0627_2016-->