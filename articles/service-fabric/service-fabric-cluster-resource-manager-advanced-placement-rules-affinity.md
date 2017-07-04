<properties
    pageTitle="Service Fabric 群集资源管理器 - 相关性 | Azure"
    description="概述如何为 Service Fabric 服务配置相关性"
    services="service-fabric"
    documentationcenter=".net"
    author="masnider"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="678073e1-d08d-46c4-a811-826e70aba6c4"
    ms.service="Service-Fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="01/05/2017"
    wacn.date="02/20/2017"
    ms.author="masnider" />  


# 在 Service Fabric 中配置和使用服务相关性
相关性是一个控件，主要用于帮助简化将较大型的单体式应用程序转换到云和微服务领域。也就是说，在某些情况下，它也可以用作提升服务性能的合法优化，尽管这样做可能产生副作用。

假设要将某个较大型应用，或在设计时未将微服务纳入考虑的应用引入 Service Fabric。此类转换很常见。首先将整个应用提升到环境中、打包应用并确保应用可以平稳运行。然后，开始将其分解成不同的、可彼此通信的较小服务。

然后就会看到“糟糕...”。出现“糟糕”通常是由下列原因之一导致的：

1. 单体式应用中的某个组件 X 与组件 Y 之间具有未记录的依赖关系，而用户刚刚将这些组件转换为独立的服务。由于这些服务现在在群集中的不同节点上运行，这种依赖关系已被破坏。
2. 这些对象会通过（本地命名管道 | 共享内存 | 磁盘上的文件）进行通信，但出于性能原因，它们实际需要立即写入共享资源。稍后会移除该硬依赖关系。
3. 一切都没问题，但事实上，这两个组件之间的对话频繁且性能需求很高。将它们移到独立的服务时，整体应用程序性能将受到严重影响，或者延迟会增大。因此，整个应用程序不符合预期。

在这些情况下，我们不想丢失重构工作，也不想回到单体结构。但是，在我们可以重新将组件设计为像服务一样自然运行之前（或在我们可以通过其他某种方式达到性能预期之前），我们需要有位置方面的概念。

怎么办？ 嗯，可以尝试启用相关性。

## 如何配置相关性
若要设置相关性，可以定义两个不同服务之间的相关性关系。可以将相关性想象成在一个服务上“指向”另一个服务，同时假设“这个服务只有在那个服务正在运行时才能运行”。 有时我们将这种相关性称为父子关系（将子级指向父级）。相关性确保将一个服务的副本或实例放置在与另一个服务的副本或实例相同的节点上。


	ServiceCorrelationDescription affinityDescription = new ServiceCorrelationDescription();
	affinityDescription.Scheme = ServiceCorrelationScheme.Affinity;
	affinityDescription.ServiceName = new Uri("fabric:/otherApplication/parentService");
	serviceDescription.Correlations.Add(affinityDescription);
	await fabricClient.ServiceManager.CreateServiceAsync(serviceDescription);


## 不同的相关性选项
相关性通过多种相互关联的架构之一来表示，有两种不同的模式。相关性的最常见模式是所谓的 NonAlignedAffinity 模式。在 NonAlignedAffinity 下，不同服务的副本或实例均放置在同一个节点上。另一种模式是 AlignedAffinity。对齐的相关性仅适用于有状态服务。配置两个有状态服务实现对齐的相关性，可确保这些服务的主要副本与其他服务的主要副本位于相同的节点上。它还能确保这些服务的每个次要副本对位于相同的节点上。也可以针对有状态服务配置 NonAlignedAffinity（但不太常见）。使用 NonAlignedAffinity 时，会将两个有状态服务的不同副本并置在同一个节点上，但不会尝试对齐它们的主要副本或次要副本。

<center> ![相关性模式及其影响][Image1] </center>

### 尽力而为的所需状态
相关性与单体式体系结构之间有一些差异。产生这些差异的原因有许多是因为相关性关系是以尽力而为的方式获取。具有相关性关系的服务是本质上不同的实体，可能失败或者被单独移动。还有一些原因会破坏相关性关系。例如，相关性关系中只有某些服务对象的容量限制能够适用于给定的节点。在这些情况下，即使存在相关性关系，也会因为其他限制而无法强制实施这种关系。如果可以这样做，稍后会自动纠正违规情况。

### 链形与星形
目前，群集资源管理器无法为链形相关性关系建模。这意味着，如果有一个服务是某一个相关性关系中的子级，则该服务不能是另一个相关性关系中的父级。如果想要为这种关系建模，需要有效地将它建模为星形而不是链形。为了从链形转变为星形，最下面的子级会变成第一个子级的父级。根据服务的排列方式，可能需要多次执行此操作。如果没有自然的父级服务，可能需要创建一个作为预留位置。根据用户要求，可能还需要了解[应用程序组](/documentation/articles/service-fabric-cluster-resource-manager-application-groups/)。

<center> ![相关性关系上下文中的链形与星形][Image2] </center>

目前关于相关性关系的另一个要注意的事项是它们是定向的。这意味着，“相关性”规则只强制子级与父级处于相同位置，而不强制父级与子级处于相同位置。例如，假设父级突然故障转移到其他节点。发生这种情况时，群集资源管理器会认为一切都没问题，直到发现子级与父级不在同一位置。由于这些服务具有不同的生命周期，相关系关系不可能完美或立即强制执行。

### 分区支持
对于相关性，要注意的最后一点是，不支持分区父级的相关性关系。我们最终可能会支持这一功能，但目前并不允许。

## 后续步骤
- 有关可用于配置服务的其他选项的详细信息，请查看 [Learn about configuring Services](/documentation/articles/service-fabric-cluster-resource-manager-configure-services/)（了解如何配置服务）中提供的其他群集资源管理器配置的相关主题
- 对于将服务限制于少量计算机，以及尝试聚合服务集合负载的用例，可使用[应用程序组](/documentation/articles/service-fabric-cluster-resource-manager-application-groups/)

[Image1]: ./media/service-fabric-cluster-resource-manager-advanced-placement-rules-affinity/cluster-resrouce-manager-affinity-modes.png
[Image2]: ./media/service-fabric-cluster-resource-manager-advanced-placement-rules-affinity/cluster-resource-manager-chains-vs-stars.png

<!---HONumber=Mooncake_0213_2017-->
<!--Update_Description: add reference to "应用程序组" -->