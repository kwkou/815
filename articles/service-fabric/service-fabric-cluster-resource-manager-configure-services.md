<properties
    pageTitle="使用 Service Fabric 群集资源管理器配置服务 | Azure"
    description="通过指定指标、放置约束和其他放置策略描述 Service Fabric 服务。"
    services="service-fabric"
    documentationcenter=".net"
    author="masnider"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="16e135c1-a00a-4c6f-9302-6651a090571a"
    ms.service="Service-Fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="01/05/2017"
    wacn.date="02/20/2017"
    ms.author="masnider" />  


# 配置 Service Fabric 服务的群集资源管理器设置
Service Fabric 群集资源管理器允许精细地控制用于管控每个独立命名服务的规则。每个命名的服务实例都可以针对其在群集中的分配方式指定规则。每个命名的服务还可以定义需要报告的一组指标，包括其对该服务的重要性。配置服务的过程划分为三个不同的任务：

1. 配置放置约束
2. 配置指标
3. 配置高级放置策略和其他规则（不太常见）

## 放置约束
放置约束可用来控制服务实际可在群集中的哪些节点上运行。通常为特定的命名服务实例或受限于在特定类型节点上运行的类型的所有服务。也就是说，放置约束是可扩展的 - 可以根据节点类型定义任意的属性集，然后在创建服务时，利用约束选择它们。放置约束还会在服务生存期内动态更新，让你响应群集中的更改。还可以在群集中动态更新给定节点的属性。有关放置约束以及如何对其进行配置的详细信息，请参阅[此文](/documentation/articles/service-fabric-cluster-resource-manager-cluster-description/#placement-constraints-and-node-properties)

## 度量值
指标是给定的命名服务实例需要的一组资源。服务的指标配置包括：默认情况下，该服务的每个有状态副本或无状态实例消耗多少该资源。指标还包括一个权重，表示该指标对于服务的重要程度，以便于权衡利弊。

## 其他放置规则
还有其他类型的放置规则，用于地理分散的群集或其他较少见的方案。其他放置规则通过相关性或策略进行配置。

## 后续步骤
- 指标是 Service Fabric 群集资源管理器在群集中管理消耗和容量的方式。若要详细了解指标及其配置方式，请查看[此文](/documentation/articles/service-fabric-cluster-resource-manager-metrics/)
- 相关性是可以针对服务配置的一种模式。它并不常用，但如果需要，可以参阅[此处](/documentation/articles/service-fabric-cluster-resource-manager-advanced-placement-rules-affinity/)
- 可以在服务上配置许多不同的放置规则以处理其他方案。可以在[此处](/documentation/articles/service-fabric-cluster-resource-manager-advanced-placement-rules-placement-policies/)了解这些不同的放置策略
- 参阅 [Service Fabric 群集资源管理器简介](/documentation/articles/service-fabric-cluster-resource-manager-introduction/)，帮助自己入门
- 若要了解群集资源管理器如何管理和均衡群集中的负载，请查看有关[均衡负载](/documentation/articles/service-fabric-cluster-resource-manager-balancing/)的文章
- 群集资源管理器提供许多用于描述群集的选项。若要详细了解这些选项，请查看这篇有关[描述 Service Fabric 群集](/documentation/articles/service-fabric-cluster-resource-manager-cluster-description/)的文章

<!---HONumber=Mooncake_0213_2017-->
<!--Update_Description: wording update-->