<properties
    pageTitle="Service Fabric 群集资源管理器 - 应用程序组 | Azure"
    description="概述 Service Fabric 群集资源管理器中的应用程序组功能"
    services="service-fabric"
    documentationcenter=".net"
    author="masnider"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="4cae2370-77b3-49ce-bf40-030400c4260d"
    ms.service="Service-Fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="01/05/2017"
    wacn.date="02/20/2017"
    ms.author="masnider" />  


# 应用程序组简介
Service Fabric 的群集资源管理器通常通过将负载（通过[指标](/documentation/articles/service-fabric-cluster-resource-manager-metrics/)表示）平均分散到整个群集来管理群集资源。Service Fabric 还管理群集中节点的容量，以及通过[容量](/documentation/articles/service-fabric-cluster-resource-manager-cluster-description/)的概念管理整个群集。指标和容量非常适用于许多种工作负荷，但大量使用不同 Service Fabric 应用程序实例的模式有时还有其他要求。其他要求通常包括：

* 能够在群集中为应用程序实例的服务保留容量
* 能够限制应用程序中服务运行的节点总数
* 定义应用程序实例本身的容量，限制其中服务的数量或资源总消耗量

为了满足这些要求，Service Fabric 群集资源管理器支持应用程序组。

## 管理应用程序容量
应用程序容量可用于限制应用程序跨越的节点数。还可限制单个节点上应用程序服务的指标负载和总指标负载。它还可用于在群集中为应用程序保留资源。

可以在创建新应用程序时为应用程序设置应用程序容量；也可以针对现有应用程序更新容量。

### 限制最大节点数
应用程序容量的最简单用例出现在需要将应用程序实例化限制为特定的最大节点数时。如未指定应用程序容量，Service Fabric 群集资源管理器将根据正常规则（均衡或重组）创建和放置服务。

下图显示了已定义和未定义最大节点数的应用程序实例。不保证哪些服务的哪些副本或实例会放在一起，或使用哪些特定节点。

<center> ![定义最大节点数的应用程序实例][Image1] </center>

在左侧的示例中，应用程序中未设置应用程序容量，并且有三个服务。群集资源管理器已将所有副本分散到 6 个可用节点，以便在群集中实现最佳均衡。在右侧示例中，可以看到限制在 3 个节点上的同一应用程序。

控制此行为的参数称为 MaximumNodes。可在创建应用程序期间设置此参数，或针对已运行的应用程序实例更新此参数。

Powershell


	New-ServiceFabricApplication -ApplicationName fabric:/AppName -ApplicationTypeName AppType1 -ApplicationTypeVersion 1.0.0.0 -MaximumNodes 3
	Update-ServiceFabricApplication –Name fabric:/AppName –MaximumNodes 5


C#


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


## 应用程序指标、负载和容量
应用程序组还允许定义与特定应用程序实例关联的指标，以及这些指标的应用程序容量。据此，可跟踪、保留和限制应用程序实例中服务的资源消耗量。

对于每个应用程序指标，可设置 2 个值：

* **应用程序总容量** - 此设置代表特定指标的应用程序总容量。群集资源管理器不允许在此应用程序实例中创建任何新服务，这会造成总负载超出此值。例如，假设应用程序实例的容量为 10，已有负载为 5。在这种情况下，不允许创建默认总负载为 10 的服务。
* **最大节点容量** - 此设置指定单个节点上应用程序中服务副本的最大总负载。如果节点上的总负载超过此容量，群集资源管理器会尝试将副本移到其他节点，以便遵守容量限制。

## 保留容量
应用程序组的另一个常见用途是确保针对给定应用程序实例保留群集中的资源。即使该应用程序实例在该群集中没有任何服务，或者服务尚未消耗资源，也会发生此情况。让我们看一下工作原理。

### 指定节点和资源保留的最小数量
为应用程序实例保留资源需要指定两个附加参数：*MinimumNodes* 和 *NodeReservationCapacity*

* **MinimumNodes** - 就像指定应用程序内的服务可在其中运行的最大节点数一样，也可以指定最小节点数。此设置可定义资源应该保留的节点数，保证创建应用程序实例时群集中的容量。
* **NodeReservationCapacity** - 可针对应用程序中的每个指标定义 NodeReservationCapacity。此设置定义针对任一节点（放置服务的任何副本或实例）上的应用程序保留的指标负载量。

让我们看看容量保留的示例：

<center> 
![定义保留容量的应用程序实例][Image2] 
</center>

在左侧的示例中，应用程序中未定义任何应用程序容量。群集资源管理器根据一般规则平衡所有内容。

在右侧示例中，假设已创建的应用程序具有以下设置：

* MinimumNodes 设置为 2
* MaximumNodes 设置为 3
* 应用程序指标定义为
  * NodeReservationCapacity 为 20
  * MaximumNodeCapacity 为 50
  * TotalApplicationCapacity 为 100

Service Fabric 为蓝色应用程序在 2 个节点上保留容量，并且不允许群集中其他应用程序实例的服务使用该容量。我们认为此保留应用程序容量是根据该节点和群集中的剩余容量消耗和计算的。

创建的应用程序具有保留容量时，群集资源管理器（为每个指标）保留等于 MinimumNodes * NodeReservationCapacity 的容量。但是，仅当至少 1 个副本放置在该特定节点上时才保留容量。后种保留允许更好地灵活利用资源，因为必要时仅在节点上保留资源。

## 获取应用程序负载信息
对于具有定义的应用程序容量的每个应用程序，可以获取其服务的副本报告的聚合负载的相关信息。

例如，可以使用以下 PowerShell cmdlet 检索负载：


	Get-ServiceFabricApplicationLoad –ApplicationName fabric:/MyApplication1



ApplicationLoad 查询返回针对应用程序所指定应用程序容量的基本信息。此信息包括最小节点数和最大节点数信息，以及应用程序当前使用的节点数。还包括有关每个应用程序负载指标的信息，包括：

* 指标名称：指标的名称。
* 保留容量：在群集中为此应用程序保留的群集容量。
* 应用程序负载：此应用程序的子副本的总负载。
* 应用程序容量：允许的最大应用程序负载值。

## 删除应用程序容量
为应用程序设置应用程序容量参数后，可以使用更新应用程序 API 或 PowerShell cmdlet 来删除这些参数。例如：


	Update-ServiceFabricApplication –Name fabric:/MyApplication1 –RemoveApplicationCapacity



此命令从应用程序删除所有应用程序容量参数。该命令立即生效。此命令完成后，群集资源管理器恢复默认行为进行应用程序管理。可通过 Update-ServiceFabricApplication 再次指定应用程序容量参数。

### 应用程序容量的限制
必须遵守应用程序容量参数的几项限制。如果出现验证错误，系统会拒绝创建或更新应用程序并提示错误，且不会发生任何更改。

* 所有整数参数必须为非负数。
* MinimumNodes 不得大于 MaximumNodes。
* 如果已定义负载指标的容量，则这些容量必须遵守以下规则：
  * 节点保留容量不得大于最大节点容量。例如，不能将节点上指标“CPU”的容量限制为 2 个单位，也不能尝试在每个节点上保留 3 个单位。
  * 如果已指定 MaximumNodes，则 MaximumNodes 和最大节点容量的积不得大于应用程序容量总计。例如，假设将负载指标“CPU”的最大节点容量设置为 8，并将最大节点数设置为 10。在这种情况下，此负载指标的应用程序总容量必须大于 80。

在（客户端）创建应用程序和在（服务器端）更新应用程序期间都会强制实施限制。

## 在哪些情况下不应使用应用程序容量
* 不要尝试使用应用程序组功能将应用程序限制在特定节点子集。换言之，可以指定应用程序最多在 5 个节点上运行，但不能指定群集中特定的 5 个节点。可使用服务放置约束完成将应用程序限制在特定节点。
* 不要尝试使用应用程序容量来确保将相同应用程序的 2 个服务放置在一起。可使用相关性或依据特定要求采用相应的放置约束，确保在相同节点上运行服务。

## 后续步骤
- 有关可用于配置服务的其他选项的详细信息，请查看 [Learn about configuring Services](/documentation/articles/service-fabric-cluster-resource-manager-configure-services/)（了解如何配置服务）中提供的其他群集资源管理器配置的相关主题
- 若要了解群集资源管理器如何管理和均衡群集中的负载，请查看有关[均衡负载](/documentation/articles/service-fabric-cluster-resource-manager-balancing/)的文章
- 参阅 [Service Fabric 群集资源管理器简介](/documentation/articles/service-fabric-cluster-resource-manager-introduction/)，帮助自己入门
- 有关在一般情况下指标的工作原理的详细信息，请阅读 [Service Fabric Load Metrics](/documentation/articles/service-fabric-cluster-resource-manager-metrics/)（Service Fabric 负载指标）
- 群集资源管理器提供许多用于描述群集的选项。若要详细了解这些选项，请查看这篇有关[描述 Service Fabric 群集](/documentation/articles/service-fabric-cluster-resource-manager-cluster-description/)的文章

[Image1]: ./media/service-fabric-cluster-resource-manager-application-groups/application-groups-max-nodes.png
[Image2]: ./media/service-fabric-cluster-resource-manager-application-groups/application-groups-reserved-capacity.png

<!---HONumber=Mooncake_0213_2017-->
<!--Update_Description: enrich introduction for "Service Fabric 指标和容量"-->