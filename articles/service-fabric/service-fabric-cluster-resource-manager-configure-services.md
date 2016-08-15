<properties
   pageTitle="使用 Service Fabric 群集资源管理器配置服务 | Azure"
   description="通过指定指标、放置约束和其他放置策略来描述 Service Fabric 服务。"
   services="service-fabric"
   documentationCenter=".net"
   authors="masnider"
   manager="timlt"
   editor=""/>

<tags
   ms.service="Service-Fabric"
   ms.date="05/20/2016"
   wacn.date="07/04/2016"/>


# 配置 Service Fabric 服务的群集资源管理器设置
Service Fabric 群集资源管理器可让你非常精细地控制用于管控每个命名服务的规则。每个服务实例都可以指定应如何在群集中进行分配的特定规则，并且可定义一组它想要报告的指标，包括它们对于该服务的重要程度。配置服务的过程通常划分为三个不同的任务：

1. 配置放置约束
2. 配置指标
3. 配置高级放置规则（不太常见）

让我们依次讨论每项任务：

## 放置约束
放置约束可用来控制服务实际可在群集中的哪些节点上运行。通常看到特定的命名服务实例或受限于在特定类型节点上运行的类型的所有服务，但放置约束是可扩展的 - 可以根据节点类型定义任意的属性组合，然后在创建服务时，利用约束来选择它们。放置约束还会在服务生存期内动态更新，让你响应群集中的更改。有关放置约束以及如何对其进行配置的详细信息，请参阅[此文](/documentation/articles/service-fabric-cluster-resource-manager-cluster-description/#placement-constraints-and-node-properties)

## 度量值
指标是此服务应在其上获取平衡的资源列表，包括该服务的每个副本或实例默认消耗该资源的数量相关信息。指标还包括一个权重，表示该指标对于服务的重要程度，以便于权衡利弊。

## 其他放置规则
还有其他类型的放置规则，主要用于地理分散的群集或其他较少见的方案。这些规则通过关联或策略来配置。尽管许多方案不使用这些规则，但出于完整性考虑，我们还是介绍了这些规则。

## 后续步骤
- 指标是 Service Fabric 群集资源管理器在群集中管理消耗和容量的方式。若要详细了解指标及其配置方式，请查看[此文](/documentation/articles/service-fabric-cluster-resource-manager-metrics/)
- 相关性是可以针对服务配置的一种模式。它并不常用，但如果需要，可以参阅[此处](/documentation/articles/service-fabric-cluster-resource-manager-advanced-placement-rules-affinity/)
- 可以在服务上配置许多不同的放置规则以处理其他方案。可以在[此处](/documentation/articles/service-fabric-cluster-resource-manager-advanced-placement-rules-placement-policies/)了解这些不同的放置策略
- [获取有关 Service Fabric 群集资源管理器的简介](/documentation/articles/service-fabric-cluster-resource-manager-introduction/)以帮助自己入门
- 若要了解群集资源管理器如何管理和平衡群集中的负载，请查看关于[平衡负载](/documentation/articles/service-fabric-cluster-resource-manager-balancing/)的文章
- 群集资源管理器提供许多用于描述群集的选项。若要详细了解这些选项，请查看有关这篇[描述 Service Fabric 群集](/documentation/articles/service-fabric-cluster-resource-manager-cluster-description/)的文章

<!---HONumber=Mooncake_0627_2016-->