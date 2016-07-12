<properties
   pageTitle="Service Fabric 群集资源管理器 - 相关性 | Azure"
   description="概述如何配置 Service Fabric 服务的相关性"
   services="service-fabric"
   documentationCenter=".net"
   authors="masnider"
   manager="timlt"
   editor=""/>

<tags
   ms.service="Service-Fabric"
   ms.date="05/20/2016"
   wacn.date="07/04/2016"/>

# 在 Service Fabric 中配置和使用服务相关性

相关性对于微服务环境而言，乍看之下似乎没有太大意义。相关性是一个控件，主要用于帮助简化将先前已存在且较大型的单体式应用程序转换到云和微服务领域。也就是说，在某些情况下它也可以用作提升服务性能的合法优化，不过这可能会产生一定的副作用。

假设你正在将某个较大型应用，或者在设计时未将微服务纳入考虑的应用引入 Service Fabric。这实际上很常见，我们曾经有多个客户（内部和外部）处于这种局面。首先将整个应用提升到该环境，将它打包并运行。然后，开始将其分解成不同的、可彼此通信的较小服务。

然后就是“惊叹”。“惊叹”通常是由下列原因之一导致的：

1. 单体式应用中的组件 X 原本与我们刚刚转换成服务并在群集间移动的组件 Y 之间具有未记录的依赖性。这种依赖性但已破坏。
2.	这些组件通过（本地命名管道 | 共享内存 | 磁盘上的文件）进行通信，但我确实需要能够独立更新它，以便能够更快地结束任务。稍后我将删除硬依赖性。
3.	一切都没问题，但事实上，这两个组件之间的对话非常频繁且性能敏感，因此当我将其移到独立的服务时，性能将受到严重影响，并且整个应用程序现在都不可用。

在所有这些情况下，我们不想丢失重构的工作，也不想返回单体式，但想尝试一同恢复所有项目，在解决问题之前让它们“觉得”是在本地。

怎么办？ 嗯，可以尝试启用相关性。

## 如何配置相关性
若要设置相关性，可以定义两个不同服务之间的相关性关系。通常可以将此关系想象成在一个服务上“指向”另一个服务，并假设“这个服务只有在那个服务正在运行时才能运行”。 有时我们将这些关系称为父子关系（将子级指向父级）。此关系的作用是确保某个服务的副本或实例与和其生成相关性的服务的副本或实例放置在同一个节点上。

``` csharp
ServiceCorrelationDescription affinityDescription = new ServiceCorrelationDescription();
affinityDescription.Scheme = ServiceCorrelationScheme.Affinity;
affinityDescription.ServiceName = new Uri("fabric:/otherApplication/parentService");
serviceDescription.Correlations.Add(affinityDescription);
await fabricClient.ServiceManager.CreateServiceAsync(serviceDescription);
```

## 不同的相关性选项
相关性通过多种相互关联的架构之一来表示，有两种不同的模式。相关性的最常见模式是所谓的 NonAlignedAffinity 模式。在 NonAlignedAffinity 下，不同服务的副本或实例放在同一个节点上。另一种模式是 AlignedAffinity。对齐的相关性用于有状态服务。将一个有状态服务配置为与另一个有状态服务具有对齐的相关性，可确保这些服务的主副本彼此都放置于相同节点上，并且每个辅助副本的配对也放置于相同节点上。也可以针对有状态服务配置 NonAlignedAffinity（但不太常见）。在这种情况下，你将获得许多与无状态服务相同的行为 – 两个有状态服务的不同副本共置于相同节点上，但不尝试确保主副本和辅助副本都位于相同节点上。

![相关性模式及其影响][Image1]

### 尽力而为的所需状态
相关性与单体式体系结构之间有一些差异。它们几乎全部都可归结为以下事实：相关性关系是尽力而为的 – 例如，由于它们基本上是不同的服务，因此可能单独失败。有其他因素（例如容量限制）可能导致关联化服务的不同副本或实例分隔开来。在这种情况下，即使有相关性关系，但由于存在其他限制，也暂时不会强制实施这种关系。如果以后可以强制实施所有其他约束和相关性，则会自动更正。

### 链形与星形
我们目前无法创建相关性关系的模型链。这意味着，如果有一个服务是某一个相关性关系中的子级，则该服务不能是另一个相关性关系中的父级。如果你想要创建这种关系的模型，则需要有效地将它的模型创建为星形而不是链形，方法是改为将最下层子级的父级设置为“中间”子级的父级。根据服务的排列方式，这可能需要创建“占位符”服务作为多个子项的父项。

![相关性关系上下文中的链形与星形][Image2]

目前关于相关性关系的另一个要注意事项是它们是双向的。这是难以察觉的，但实际上这意味着“相关性”规则只强制子级就是父级的所在之处 – 父级应该突然故障转移到另一个节点（或者任何其他受限的操作，其只强制移动父级），然后在资源管理器注意到子级不是与父级位于相同位置之前，它实际上不觉得有任何不妥之处；关系不立即强制实施。

### 分区支持
对于相关性，要注意的最后一点是，不支持分区父级的相关性关系。我们最终可能会支持这一点，但目前并不允许。

## 后续步骤
- 有关可用于配置服务的其他选项的详细信息，请查看 [Learn about configuring Services](/documentation/articles/service-fabric-cluster-resource-manager-configure-services/)（了解如何配置服务）中提供的其他群集资源管理器配置的相关主题
- 用户尝试使用相关性的许多原因（例如限制服务在少量计算机上运行，以及尝试聚合服务集合的负载）实际上可以通过应用程序组获得更好的支持。请查看 [Application Groups](/documentation/articles/service-fabric-cluster-resource-manager-application-groups/)（应用程序组）

[Image1]: ./media/service-fabric-cluster-resource-manager-advanced-placement-rules-affinity/cluster-resrouce-manager-affinity-modes.png
[Image2]: ./media/service-fabric-cluster-resource-manager-advanced-placement-rules-affinity/cluster-resource-manager-chains-vs-stars.png

<!---HONumber=Mooncake_0627_2016-->