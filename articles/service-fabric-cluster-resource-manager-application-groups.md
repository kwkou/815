<properties
   pageTitle="Service Fabric 群集资源管理器 - 应用程序组 | Azure"
   description="概述 Service Fabric 群集资源管理器中的应用程序组功能"
   services="service-fabric"
   documentationCenter=".net"
   authors="masnider"
   manager="timlt"
   editor=""/>

<tags
   ms.service="Service-Fabric"
   ms.date="05/20/2016"
   wacn.date="07/04/2016"/>

# 应用程序组简介
Service Fabric 的群集资源管理器通常通过将负载（通过指标表示）平均分散到整个群集来管理群集资源。Service Fabric 还管理群集中节点的容量，以及通过容量的概念管理整个群集。这非常适合用于许多不同类型的工作负荷，但大量使用不同 Service Fabric 应用程序实例的模式还有其他要求。其他要求通常包括：

- 能够在某个数量的节点上为应用程序实例的服务保留容量
- 能够限制可运行应用程序中给定服务集的节点总数
- 定义应用程序实例本身的容量，以限制其中服务的资源总消耗量

为了满足这些要求，我们开发了对应用程序组的支持。

## 管理应用程序容量
应用程序容量可用于限制应用程序跨越的节点数，以及单个节点上应用程序实例的总指标负载。它还可用于在群集中为应用程序保留资源。

可以在创建新应用程序时为应用程序设置应用程序容量；也可以针对已创建但未指定应用程序容量的现有应用程序更新容量。

### 限制最大节点数
应用程序容量的最简单用例出现在需要将应用程序实例化限制为特定的最大节点数时。如果未指定应用程序容量，Service Fabric 群集资源管理器将根据正常规则（平衡或重组）实例化副本，这通常表示其服务跨越群集中的可用节点分散，或者是否在任意但较少数目的节点上打开重组。

下图显示了未定义最大节点数的应用程序实例的可能位置，然后显示已设置最大节点数的同一个应用程序。请注意，不保证哪些服务的哪些副本或实例会放在一起。

![定义最大节点数的应用程序实例][Image1]

在左侧的示例中，应用程序中未设置应用程序容量，并且有三个服务。CRM 做了逻辑决策，将所有副本分散到六个可用节点以便在群集中实现最佳平衡。在右侧的示例中，可以看到限制在三个节点上的同一个应用程序，其中 Service Fabric CRM 已在应用程序的服务副本中实现最佳平衡。

控制此行为的参数称为 MaximumNodes。可以在创建应用程序期间设置此参数，或针对已在运行的应用程序实例更新此参数，对于后一种情况，Service Fabric CRM 会将应用程序的服务的副本限制为定义的最大节点数。

Powershell

``` posh
New-ServiceFabricApplication -ApplicationName fabric:/AppName -ApplicationTypeName AppType1 -ApplicationTypeVersion 1.0.0.0 -MaximumNodes 3
Update-ServiceFabricApplication –Name fabric:/AppName –MaximumNodes 5
```

C#

``` csharp
ApplicationDescription ad = new ApplicationDescription();
ad.ApplicationName = new Uri("fabric:/AppName");
ad.ApplicationTypeName = "AppType1";
ad.ApplicationTypeVersion = "1.0.0.0";
ad.MaximumNodes = 3;
fc.ApplicationManager.CreateApplicationAsync(ad);

ApplicationUpdateDescription adUpdate = new ApplicationUpdateDescription(new Uri("fabric:/AppName"));
adUpdate.MaximumNodes = 5;
fc.ApplicationManager.UpdateApplicationAsync(adUpdate);

var appMetric = new ApplicationMetricDescription();
appMetric.Name = "Metric1";
appMetric.TotalApplicationCapacity = 1000;

adUpdate.Metrics.Add(appMetric);
```

## 应用程序指标、负载和容量
应用程序组还允许你定义与特定应用程序实例关联的指标，以及这些指标相关的应用程序的容量。因此，举例来说，你可以根据需要定义任意数量的指标，只要不超过可在其中创建指标的服务数即可

对于每个指标，可将 2 个值设置为描述该应用程序实例的容量：

-	应用程序容量总计 – 代表特定指标的应用程序容量总计。Service Fabric CRM 尝试将此应用程序的服务的指标负载总计限制为指定值；此外，如果应用程序的服务已消耗超过此限制的负载，Service Fabric 群集资源管理器将不允许创建任何新的服务或分区，这会造成总负载超出此限制。
-	最大节点容量 – 指定单个节点上应用程序的服务副本的最大总负载。如果节点上的总负载超过此容量，Service Fabric CRM 会尝试将副本移到其他节点，以便遵守容量限制。

## 保留容量
应用程序组的另一个常见用途是确保针对指定的应用程序实例保留群集中的资源，即使该应用程序实例在该群集中没有服务，或者它们尚未消耗资源。让我们看一下具体的工作原理。

### 指定节点和资源保留的最小数量
为应用程序实例保留资源需要指定两个附加参数：*MinimumNodes* 和 *NodeReservationCapacity*

- MinimumNodes - 就像指定应用程序内的服务可以在上面运行的目标最大节点数一样，你可以指定应用程序应该运行的最小节点数。此设置可有效地定义资源应该保留的最小节点数，在创建应用程序实例时保证群集中的容量。
- NodeReservationCapacity - 可针对应用程序中的每个指标定义 NodeReservationCapacity。此参数定义针对任一节点（放置服务的任何副本或实例）上的应用程序保留的指标负载。

让我们看看容量保留的示例：

![定义保留容量的应用程序实例][Image2]

在左侧的示例中，应用程序中未定义任何应用程序容量。Service Fabric 群集资源管理器将平衡应用程序的子服务副本和实例，以及来自其他服务（在应用程序外部）的副本和实例，以确保群集中的平衡。

在右侧的示例中，让我们假设已创建的应用程序的 MinimumNodes 设置为 2、MaximumNodes 设置为 3，且应用程序指标定义为保留 20、最大节点容量 50 和应用程序容量总计 100，Service Fabric 将为蓝色应用程序在两个节点上保留容量，并且不允许群集中的其他副本使用该容量。我们认为此保留应用程序容量是根据剩余群集容量消耗和计算的。

当创建的应用程序具有保留容量时，群集资源管理器在群集中保留等于 MinimumNodes * NodeReservationCapacity 的容量，但是不在特定节点上保留容量，直到创建和放置应用程序的服务的副本。这可以获得弹性，因为只有在新副本创建时针对它们选择节点。当至少一个副本放置在该特定节点上时保留容量。

## 获取应用程序负载信息
对于具有定义的应用程序容量的每个应用程序，可以获取其服务的副本报告的聚合负载的相关信息。Service Fabric 针对此目的提供 PowerShell 和 Managed API 查询。

例如，可以使用以下 PowerShell cmdlet 检索负载：

``` posh
Get-ServiceFabricApplicationLoad –ApplicationName fabric:/MyApplication1

```

此查询的输出包含已针对应用程序指定的应用程序容量的基本信息，例如最小节点数和最大节点数。另外还提供有关应用程序当前使用的节点数的信息。因此，将会针对每个负载指标提供以下相关信息：
- 指标名称：指标的名称。
-	保留容量：在群集中为此应用程序保留的群集容量。
-	应用程序负载：此应用程序的子副本的总负载。
-	应用程序容量：允许的最大应用程序负载值。

## 删除应用程序容量
为应用程序设置应用程序容量参数后，可以使用更新应用程序 API 或 PowerShell cmdlet 来删除这些参数。例如：

``` posh
Update-ServiceFabricApplication –Name fabric:/MyApplication1 –RemoveApplicationCapacity

```

此命令从应用程序删除所有应用程序容量参数，Service Fabric 群集资源管理器开始将此应用程序视为群集中未定义这些参数的任何其他应用程序。该命令将立即产生效果，群集资源管理器将删除此应用程序的所有应用程序容量参数；再次指定它们需要使用适当的参数调用更新应用程序 API。

## 应用程序容量的限制
必须遵守应用程序容量参数的几项限制。发生验证错误时，创建或更新应用程序的操作将被拒绝并出错。所有整数参数必须为非负数。此外，单个参数的限制如下：

-	MinimumNodes 不得大于 MaximumNodes。
-	如果已定义负载指标的容量，则这些容量必须遵守以下规则：
  - 节点保留容量不得大于最大节点容量。例如，尝试在每个节点上保留 3 个单位时，不能将节点上的指标“CPU”的容量限制为 2 个单位。
  - 如果已指定 MaximumNodes，则 MaximumNodes 和最大节点容量的积不得大于应用程序容量总计。例如，如果将负载指标“CPU”的最大节点容量设置为 8，并将最大节点数设置为 10，则此负载指标的应用程序容量总计必须大于 80。

在（客户端）创建应用程序和在（服务器端）更新应用程序期间都会强制实施限制。在创建期间，这是明显违反要求的一个例子，因为 MaximumNodes 小于 MinimumNodes，在将请求发送到 Service Fabric 群集之前，客户端中的命令就会失败：

``` posh
New-ServiceFabricApplication –Name fabric:/MyApplication1 –MinimumNodes 6 –MaximumNodes 2
```

无效更新的示例如下。如果我们采用现有应用程序并将最大节点数更新为某个值，则会传递更新：

``` posh
Update-ServiceFabricApplication –Name fabric:/MyApplication1 6 –MaximumNodes 2
```

接下来，我们可以尝试更新最小节点数：

``` posh
Update-ServiceFabricApplication –Name fabric:/MyApplication1 6 –MinimumNodes 6
```

客户端不提供有关应用程序的足够上下文，因此允许将更新传递到 Service Fabric 群集。但是，在群集中，Service Fabric 将验证新参数与现有参数，由于最小节点数的值大于最大节点数的值，因此更新操作将会失败。在此情况下，应用程序容量参数将保持不变。

这些限制按顺序实施，使群集资源管理器能够为应用程序的服务副本提供最佳位置。

## 在哪些情况下不应使用应用程序容量

-	不要使用应用程序容量将应用程序限制为特定的节点子集：尽管 Service Fabric 确保对于已指定应用程序容量的每个应用程序遵守最大节点数，用户无法确定它实例化所在的节点。这可以使用服务的放置约束来实现。
-	不要使用应用程序容量来确保相同应用程序的两个服务始终放在一起。这可通过使用服务之间的相关性关系来实现，相关性可以限制为实际应该放在一起的服务。

## 后续步骤
- 有关可用于配置服务的其他选项的详细信息，请查看 [Learn about configuring Services](/documentation/articles/service-fabric-cluster-resource-manager-configure-services/)（了解如何配置服务）中提供的其他群集资源管理器配置的相关主题
- 若要了解群集资源管理器如何管理和平衡群集中的负载，请查看关于[平衡负载](/documentation/articles/service-fabric-cluster-resource-manager-balancing/)的文章
- [获取有关 Service Fabric 群集资源管理器的简介](/documentation/articles/service-fabric-cluster-resource-manager-introduction/)以帮助自己入门
- 有关在一般情况下指标的工作原理的详细信息，请阅读 [Service Fabric Load Metrics](/documentation/articles/service-fabric-cluster-resource-manager-metrics/)（Service Fabric 负载指标）
- 群集资源管理器提供许多用于描述群集的选项。若要详细了解这些选项，请查看有关这篇[描述 Service Fabric 群集](/documentation/articles/service-fabric-cluster-resource-manager-cluster-description/)的文章


[Image1]: ./media/service-fabric-cluster-resource-manager-application-groups/application-groups-max-nodes.png
[Image2]: ./media/service-fabric-cluster-resource-manager-application-groups/application-groups-reserved-capacity.png

<!---HONumber=Mooncake_0627_2016-->