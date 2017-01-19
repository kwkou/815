<properties
   pageTitle="资源平衡器群集描述 | Azure"
   description="通过在群集资源管理器中指定容错域、升级域、节点属性和节点容量来描述 Service Fabric 群集。"
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
   wacn.date="01/17/2017"
   ms.author="masnider"/>  


# 描述 Service Fabric 群集
Service Fabric 群集资源管理器提供多种机制用于描述群集。在运行时，群集资源管理器使用此信息来确保群集中运行的服务的高可用性，同时确保适当使用群集中的资源。

## 关键概念
群集资源管理器支持多种用于描述群集的功能：

- 容错域
- 升级域
- 节点属性
- 节点容量

## 容错域
容错域是协调故障的任何区域。单一计算机就是容错域（它本身可能出于各种原因而发生故障，包括电源故障、驱动器故障、NIC 固件错误，等等）。连接到同一以太网交换机的许多计算机位于同一个容错域中，连接到单一电源的计算机也是如此。由于这些组件在性质上是重叠的，容错域原本就有层次性，在 Service Fabric 中以 URI 的形式表示。

如果已设置自己的群集，则需要考虑故障的所有不同区域，以确保容错域已正确设置，使 Service Fabric 知道可以安全放置服务的位置。所谓的“安全”实际上是指智能 – 我们不想要将服务放置在造成服务停机的容错域中（例如上面所列的任何组件发生故障）。在 Azure 环境中，我们利用环境提供的容错域信息来自动正确配置群集中的节点。在下图（图 7）中，我们为所有实体着色以便使容错域成为简单的示例，并列出生成的所有不同容错域。此示例列出了数据中心 (DC)、机架 (R) 和刀片服务器 (B)。可以想象，如果每个刀片服务器包含多个虚拟机，则容错域层次结构中可能有另一个层。

![通过故障域组织的节点][Image1]  


 在运行时，Service Fabric 群集资源管理器会考虑群集中的容错域，并尝试分散指定服务的副本，以便将它们全都放在不同的容错域中。在任何一个容错域（位于层次结构中的任何一个级别）发生故障时，此过程有助于确保该服务的可用性不遭到破坏。

 Service Fabric 群集资源管理器实际上不考虑层次结构中有多少层，不过因为它会尝试确保丢失层次结构的任何一部分并不影响在其上运行的群集或服务，通常最好是在容错域的每一个深度级别有相同的计算机数目。这可以防止层次结构的某一个部分在一天结束时包含比其他部分还要多的服务。

 以容错域“树”不平衡的方式设置群集会使群集资源管理器难以找出最佳的副本分配，具体而言，丢失特定的域可能会严重影响群集的可用性 – 群集资源管理器很难决定是要通过放置服务在该“繁重”域中有效使用计算机，还是要放置服务以使域的丢失不会造成问题。

 下图显示了两个不同的示例群集布局，其中一个群集的节点完善地分散到容错域，而另一个群集中的某个容错域有多出许多的节点。请注意，在 Azure 中，系统将为你做出哪个节点位于哪个容错和升级域的选择，因此你应该永远不会看到这种失衡。不过，如果你曾经在本地或另一个环境中维护自己的群集，则需要注意一些事项。

 ![两种不同的群集布局][Image2]

## 升级域
升级域是另一项功能，可帮助 Service Fabric 资源管理器了解群集的布局，使它可以针对故障事先做出计划。升级域定义区域（实际上是节点集），在升级期间的同一时间停机。

升级域非常类似于容错域，但有几个重要的差异。首先，升级域通常由策略定义；容错域由协调故障区域严格定义（因此通常是环境的硬件布局）。但是，对于升级域，你需要确定所需的数量。另一个差异在于（截至目前为止），升级域不是分层的 – 更像是简单的标记。

下图显示了一种虚构的设置，其中有条带化到三个容错域中的三个升级域。该图还显示了有状态服务的三个不同副本的一种可能的放置方式。请注意，它们全都在不同的容错和升级域中。这意味着我们可能会在服务升级中途丢失容错域，并且群集中仍然有一个正在运行的代码和数据副本。根据你的需要，这种情况可能已足够良好，不过你可能发现此副本是旧的（因为 Service Fabric 使用基于仲裁的复制）。若要真正能够幸免于发生两次故障，你需要更多的副本（最少五个）。

![包含容错域和升级域的布局][Image3]

使用大量升级域既有利也有弊 – 优点是升级的每个步骤更细微，因此会给较少的节点或服务造成影响。这会减少每次需要移动的服务数，降低系统中的流动，并在整体上提高可靠性（因为受到任何问题影响的服务更少）。使用大量升级域的缺点是，Service Fabric 在升级域升级时将验证每个升级域的运行状况，并确保升级域状况良好再转到下一个升级域。此项检查的目的是确保服务能够稳定，在继续升级之前验证其运行状况以便检测任何问题。这种折衷是可接受的，因为可以防止错误的更改一次性影响过多的服务。

升级域太少会产生负面影响 – 当每个升级域关闭并进行升级时，大部分的整体容量将不可用。例如，如果你只有三个升级域，则一次只能采用整体服务或群集容量的 1/3。这并非理想的结果，因为你必须在其余的群集中拥有足够的容量才能应对工作负荷，这意味着在正常情况下，这些节点比增加 COGS 时的负荷更少。

环境中容错或升级域的总数没有实际限制，对于它们如何重叠也没有约束。我们看到的通用结构采用 1:1（其中每个唯一容错域映射到其本身的升级域）的、每个节点一个升级域（物理或虚拟 OS 实例）的“条带化”或“矩阵”模型，其中的容错域和升级域构成了沿对角线分布的计算机矩阵。

![故障域和升级域布局][Image4]

至于要选择哪种分配并没有最佳答案，每种做法各有利弊。例如，1FD:1UD 模型相当容易设置，而每个节点 1 个 UD 的模型是最类似于以前人们用于管理少量计算机的模型（其中每个计算机可独立关闭）。

最常见的模型（我们用于托管 Azure Service Fabric 群集的模型）是 FD/UD 矩阵，其中 FD 和 UD 构成一个表，节点沿着对角线开始放置。最后的结构是稀疏还是紧凑取决于相比于 FD 和 UD 数目的节点总数（换而言之，对于明显较大的群集，几乎所有项目最终看起来就像是密集矩阵模式，如“图 10”的右下角选项所示）。

## 容错域与升级域约束及最终行为
群集资源管理器将要在容错域与升级域之间保持服务的均衡视为约束。可以在[此文](/documentation/articles/service-fabric-cluster-resource-manager-management-integration/)中详细了解约束。容错域和升级域约束的定义如下：“针对给定服务分区，两个域之间的副本数目差异应该永远不*大于一*”。 这实际上意味着，对于给定的服务而言，特定的移动或排列方式在群集中可能无效，因为这样做会违反容错域和升级域的约束。

让我们看一个示例。假设有一个 6 节点群集，其中配置了 5 个容错域和 5 个升级域。

| |FD0 |FD1 |FD2 |FD3 |FD4 |
|-------|:-----:|:-----:|:-----:|:-----:|:-----:|
| UD0 |N1 | | | | |
| UD1 |N6 |N2 | | | |
| UD2 | | |N3 | | |
| UD3 | | | |N4 | |
| UD4 | | | | |N5 |

假设要创建一个 TargetReplicaSetSize 为 5 的服务。副本驻留在 N1-N5 上。事实上，N6 永远不被使用。为什么？ 看看当前的布局以及选择 N6 时所发生情况之间的差异，然后考虑 FD 和 UD 约束的定义之间的关系。

下面是获得的布局，以及每个容错域和升级域的副本总数。


| |FD0 |FD1 |FD2 |FD3 |FD4 |UDTotal|
|-------|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|
| UD0 |R1 | | | | |1 |
| UD1 | |R2 | | | |1 |
| UD2 | | |R3 | | |1 |
| UD3 | | | |R4 | |1 |
| UD4 | | | | |R5 |1 |
|FDTotal|1 |1 |1 |1 |1 |- |

可以看到，此布局在每个容错域与升级域的节点数目上是均衡的，在每个容错域和升级域的副本数目上也一样是均衡的。每个域都拥有相同数量的节点，以及相同数量的副本。

现在让我们看看，如果不使用 N2，而是改用 N6 会发生什么情况。副本将如何分布？ 看起来应该类似于这样：

| |FD0 |FD1 |FD2 |FD3 |FD4 |UDTotal|
|-------|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|
| UD0 |R1 | | | | |1 |
| UD1 |R5 | | | | |1 |
| UD2 | | |R2 | | |1 |
| UD3 | | | |R3 | |1 |
| UD4 | | | | |R4 |1 |
|FDTotal|2 |0 |1 |1 |1 |- |

这违反了容错域约束的定义，因为 FD0 具有 2 个副本，而 FD1 有 0 个，总差为 2，因此群集资源管理器不允许这种排列方式。同样，如果选择 N2-6，则会得到：

| |FD0 |FD1 |FD2 |FD3 |FD4 |UDTotal|
|-------|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|
| UD0 | | | | | |0 |
| UD1 |R5 |R1 | | | |2 |
| UD2 | | |R2 | | |1 |
| UD3 | | | |R3 | |1 |
| UD4 | | | | |R4 |1 |
|FDTotal|1 |1 |1 |1 |1 |- |

尽管从容错域的角度看是均衡的，但违反了升级域的约束（因为 UD0 有 0 个副本，而 UD1 有 2 个），因此这也是无效的。

## 配置容错域和升级域
定义容错域和升级域是在 Azure 托管的 Service Fabric 部署中自动完成的；Service Fabric 只从 Azure 中选择环境信息。在 Azure 中容错域和升级域信息看起来像是“单一级别”，但实际上是从 Azure Stack 的较低级别封装信息，并且仅从用户的角度显示逻辑容错域和升级域。

如果你正在运行自己的群集（或只是想要在开发计算机上尝试运行特定拓扑），则需要自行提供容错域和升级域信息。在此示例中，我们定义了一个 9 节点本地开发群集，该群集跨三个“数据中心”（每个数据中心有三个机架）；三个升级域跨这三个数据中心条带化。在群集清单模板中，该布局类似于：

ClusterManifest.xml


	  <Infrastructure>
	    <!-- IsScaleMin indicates that this cluster runs on one-box /one single server -->
	    <WindowsServer IsScaleMin="true">
	      <NodeList>
	        <Node NodeName="Node01" IPAddressOrFQDN="localhost" NodeTypeRef="NodeType01" FaultDomain="fd:/DC01/Rack01" UpgradeDomain="UpgradeDomain1" IsSeedNode="true" />
	        <Node NodeName="Node02" IPAddressOrFQDN="localhost" NodeTypeRef="NodeType02" FaultDomain="fd:/DC01/Rack02" UpgradeDomain="UpgradeDomain2" IsSeedNode="true" />
	        <Node NodeName="Node03" IPAddressOrFQDN="localhost" NodeTypeRef="NodeType03" FaultDomain="fd:/DC01/Rack03" UpgradeDomain="UpgradeDomain3" IsSeedNode="true" />
	        <Node NodeName="Node04" IPAddressOrFQDN="localhost" NodeTypeRef="NodeType04" FaultDomain="fd:/DC02/Rack01" UpgradeDomain="UpgradeDomain1" IsSeedNode="true" />
	        <Node NodeName="Node05" IPAddressOrFQDN="localhost" NodeTypeRef="NodeType05" FaultDomain="fd:/DC02/Rack02" UpgradeDomain="UpgradeDomain2" IsSeedNode="true" />
	        <Node NodeName="Node06" IPAddressOrFQDN="localhost" NodeTypeRef="NodeType06" FaultDomain="fd:/DC02/Rack03" UpgradeDomain="UpgradeDomain3" IsSeedNode="true" />
	        <Node NodeName="Node07" IPAddressOrFQDN="localhost" NodeTypeRef="NodeType07" FaultDomain="fd:/DC03/Rack01" UpgradeDomain="UpgradeDomain1" IsSeedNode="true" />
	        <Node NodeName="Node08" IPAddressOrFQDN="localhost" NodeTypeRef="NodeType08" FaultDomain="fd:/DC03/Rack02" UpgradeDomain="UpgradeDomain2" IsSeedNode="true" />
	        <Node NodeName="Node09" IPAddressOrFQDN="localhost" NodeTypeRef="NodeType09" FaultDomain="fd:/DC03/Rack03" UpgradeDomain="UpgradeDomain3" IsSeedNode="true" />
	      </NodeList>
	    </WindowsServer>
	  </Infrastructure>

> [AZURE.NOTE] 在 Azure 部署中，由 Azure 分配容错域和升级域。因此，Azure 基础结构选项中节点和角色的定义不包含容错域或升级域信息。

##<a name="placement-constraints-and-node-properties"></a> 放置约束和节点属性
有时（事实上是大多数情况下），需要确保只在群集中的特定节点或一组特定节点上运行某些工作负荷。例如，某些工作负荷可能需要 GPU 或 SSD，而有些则不用。一个有说服力的示例就是几乎在每个 n 层体系结构中，特定计算机充当应用程序的前端/接口服务端（因此通常在 Internet 上公开），而不同的集（通常包含不同的硬件资源）处理计算或存储层的工作（因此通常不会在 Internet 上公开）。Service Fabric 甚至预期到，在微服务领域中有特定的工作负荷需要在特定硬件配置上运行，例如：

- 现有的 n 层应用程序已“提升并迁移”到 Service Fabric 环境
- 出于性能、缩放或安全性隔离原因，某个工作负荷需要在特定硬件上运行
-	出于策略或资源消耗原因，某个工作负荷需要与其他工作负荷隔离

为了支持这种配置，Service Fabric 提供所谓“放置约束”的第一类概念。放置约束可用于指示应在何处运行特定服务。约束集可由用户扩展，表示人员可以使用自定义属性来标记节点，然后针对这些节点做出选择。

![群集布局中的不同工作负荷][Image5]  


节点上的不同键/值标记称为节点放置*属性*（或简称为节点属性），而服务中的语句称为放置*约束*。节点属性中指定的值可以是字符串、布尔值或带符号的长型值。约束可以是针对群集中的不同节点属性运行的任何布尔值语句。这些布尔值语句（字符串）中的有效选择器为：

- 用于创建特定语句的条件检查
  - “等于”==
  - “大于”>
  - “小于”<
  - “不等于”!=
  - “大于等于”>=
  - “小于等于”<=
- 用于分组和求反的布尔值语句
  - “与”&&
  - “或”||
  - “非”!
- 用于分组操作的括号
  - ()

  下面是使用上述某些符号的基本约束语句示例。请注意，节点属性可以是字符串、布尔值或数字值。

  - "Foo >= 5"
  - "NodeColor != green"
  - "((OneProperty < 100) || ((AnotherProperty == false) && (OneProperty >= 100)))"


只有整个语句求值为“True”的节点才能放置服务。未定义属性的节点不匹配包含该属性的任何放置约束。

Service Fabric 还定义了一些默认属性，无需用户进行定义，系统即会自动使用这些属性。截至本文发布时，在每个节点上定义的默认属性是 NodeType 和 NodeName。因此举例而言，可以将放置约束编写为 "(NodeType == NodeType03)"。通常我们发现 NodeType 是最常用的属性之一，因为它通常与计算机的类型之间存在 1:1 的对应关系，而这又相当于与传统 n 层应用程序体系结构的工作负荷类型发生 1:1 的对应关系。

![放置约束和节点属性][Image6]  


假设为给定节点类型定义了以下节点属性：ClusterManifest.xml


    <NodeType Name="NodeType01">
      <PlacementProperties>
        <Property Name="HasSSD" Value="true"/>
        <Property Name="NodeColor" Value="green"/>
        <Property Name="SomeProperty" Value="5"/>
      </PlacementProperties>
    </NodeType>


你可以针对服务创建服务放置约束，如下所示：

C#


	FabricClient fabricClient = new FabricClient();
	StatefulServiceDescription serviceDescription = new StatefulServiceDescription();
	serviceDescription.PlacementConstraints = "(HasSSD == true && SomeProperty >= 4)";
	// add other required servicedescription fields
	//...
	await fabricClient.ServiceManager.CreateServiceAsync(serviceDescription);


Powershell：


	New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceType -Stateful -MinReplicaSetSize 2 -TargetReplicaSetSize 3 -PartitionSchemeSingleton -PlacementConstraint "HasSSD == true && SomeProperty >= 4"


如果你确定 NodeType01 的所有节点都有效，则也可以只选择该节点类型，使用如上图所示的放置约束。

服务放置约束的一个突出优点是它们可以在运行时动态更新。因此如果需要，你可以在群集中移动服务、添加和删除要求，等等。Service Fabric 负责确保服务保持运行且可用，即使此类更改持续进行。

C#：


	StatefulServiceUpdateDescription updateDescription = new StatefulServiceUpdateDescription();
	updateDescription.PlacementConstraints = "NodeType == NodeType01";
	await fabricClient.ServiceManager.UpdateServiceAsync(new Uri("fabric:/app/service"), updateDescription);


Powershell：


	Update-ServiceFabricService -Stateful -ServiceName $serviceName -PlacementConstraints "NodeType == NodeType01"


放置约束（以及即将讨论的许多其他协调器控制）是针对每个不同的命名服务实例指定的。更新始终会取代（覆盖）以前指定的值。

此外，值得注意的是，节点上的属性目前是通过群集定义来定义的，因此在不升级群集的情况下无法更新，需要先将每个节点关闭再启动才能刷新其属性。

## 容量
任何协调器的最重要作业之一是帮助管理群集中的资源消耗。如果你想要有效运行服务，最不想遇到的情况是许多节点是热的（导致资源争用和性能不佳），而其他节点是冷的（浪费资源）。但是，让我们试着想想比平衡（稍后将会讲解）还要简单的事情 – 只要确保节点不在第一时间用完资源，情况会怎样？

Service Fabric 使用“指标”表示资源。指标是你想要向 Service Fabric 描述的任何逻辑或物理资源。指标的示例是诸如“WorkQueueDepth”或“MemoryInMb”的参数。指标与放置约束和节点属性的不同之处在于，节点属性通常是节点本身的静态描述符，而指标与节点包含的资源，以及当服务在节点上运行时服务消耗的资源相关。因此，属性看起来像是 HasSSD，可设置为 true 或 false，但是该 SSD 上的可用空间量（和服务使用的空间量）是类似于“DriveSpaceInMb”的指标。节点上的容量将“DriveSpaceInMb”设置为驱动器上的非保留总空间量，服务将报告在运行时使用了多少指标。

如果关闭所有资源*均衡*，Service Fabric 群集资源管理器仍能确保最终没有节点超出其容量（除非群集整体上太满）。容量是群集资源管理器用来了解节点包含的资源量的另一个*约束*。服务级别的容量和消耗量均以指标来表示。因此举例而言，指标可能是“MemoryInMb”，给定的节点可能拥有 2048 个单位的 MemoryInMb 容量，而给定的服务可以说它目前正在消耗 64 个单位的 MemoryInMb。

在运行时，群集资源管理器将跟踪每个节点上的每个资源有多少（根据其容量定义），以及剩余多少（从每个服务中减去任何声明的使用量）。使用此信息，Service Fabric 资源管理器可找出要放置或移动副本的位置，使节点不会超过容量。

C#：


	StatefulServiceDescription serviceDescription = new StatefulServiceDescription();
	ServiceLoadMetricDescription metric = new ServiceLoadMetricDescription();
	metric.Name = "MemoryInMb";
	metric.PrimaryDefaultLoad = 64;
	metric.SecondaryDefaultLoad = 64;
	metric.Weight = ServiceLoadMetricWeight.High;
	serviceDescription.Metrics.Add(metric);
	await fabricClient.ServiceManager.CreateServiceAsync(serviceDescription);


Powershell：


	New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName –Stateful -MinReplicaSetSize 2 -TargetReplicaSetSize 3 -PartitionSchemeSingleton –Metric @("Memory,High,64,64”)


![群集节点和容量][Image7]  


你可以在群集清单中看到这些内容：

ClusterManifest.xml


    <NodeType Name="NodeType02">
      <Capacities>
        <Capacity Name="MemoryInMb" Value="2048"/>
        <Capacity Name="Disk" Value="10000"/>
      </Capacities>
    </NodeType>


此外，服务的负载也会动态变化。假设副本的负载从 64 更改为 1024，但是当时正在运行该副本的节点上只剩下 512（个单位的“MemoryInMb”度量）。因此，当前放置副本或实例的位置可能失效，因为该节点上所有副本和实例的合并使用量超出了该节点的容量。稍后将更详细讨论负载可以动态更改的方案，但是只要还有容量，就能够以相同方式处理 - 群集资源管理器将自动启动，并通过将低于容量的节点上的一个或多个副本或实例移到其他节点，来恢复该节点。执行此操作时，群集资源管理器会尝试将所有移动的成本降到最低（稍后再回头讨论“成本”的概念）。

## 群集容量
那么，我们要如何防止整体群集太满？ 使用动态负载，实际上我们并没有太多可以执行的操作（因为服务有自己的负载高峰，独立于群集资源管理器所执行的操作 – 今天群集具有许多空余空间，但是明天当用户成名之后空间就不足），但是有些控件可以防止基本问题。我们可做的第一件事是防止创建导致群集空间变满的新工作负荷。

假设你要创建一个简单的无状态服务，并且它具有某些关联的负载（比默认值还多且动态负载稍后才报告）。对于此服务，让我们假设它考虑某些资源（例如磁盘空间），默认情况下针对服务的每个实例使用 5 个单位的磁盘空间。你需要创建服务的 3 个实例。很好！ 这意味着我们需要群集中有 15 个单位的磁盘空间才能创建这些服务实例。Service Fabric 将持续计算整体容量和每个指标的消耗量，因此我们可以轻松做出决定并且在空间不足时拒绝创建服务调用。

请注意，由于要求仅是 15 个可用单位，此空间可以使用不同的方式分配；例如，可能是在 15 个不同节点上各一个剩余单位，或是在 5 个不同节点上各三个剩余单位，等等。如果三个不同节点上没有足够的容量，Service Fabric 将重新组织已在群集中的服务，以便在三个必要的节点上腾出空间。此类重新排列几乎始终可行，除非整个群集几乎已满。

##<a name="buffered-capacity"></a> 缓冲容量
为了帮助用户管理整体群集容量所做的另一件事是，针对每个节点的指定容量添加了一些保留缓冲区。此设置是可选的，但可以让用户保留整体节点容量的某些部分，让它只用于在升级和失败期间放置服务 - 应对群集容量降低的情况。目前缓冲区是通过 ClusterManifest 针对所有节点全局指定的。针对保留容量选择的值是服务更受约束的资源的函数，以及在群集中具有的容错和升级域的数目。通常较多的容错和升级域表示可以选择较少数目的缓冲处理容量，因为预期在升级和失败期间有较少量的群集无法使用。请注意，指定缓冲区百分比只有在同时指定了指标的节点容量时才有意义。

以下示例演示如何指定缓冲容量：

ClusterManifest.xml


        <Section Name="NodeBufferPercentage">
            <Parameter Name="DiskSpace" Value="0.10" />
            <Parameter Name="Memory" Value="0.15" />
            <Parameter Name="SomeOtherMetric" Value="0.20" />
        </Section>

创建新服务会在群集耗尽缓冲容量时失败，确保群集保留足够的备用额外负荷，使升级和失败不会造成节点实际超过容量。群集资源管理器通过 PowerShell 和查询 API 公开许多此类信息，以便查看缓冲容量设置、总容量及每个群集中使用的每个指标的当前消耗量。下面提供了该输出的示例：


	PS C:\Users\user> Get-ServiceFabricClusterLoadInformation
	LastBalancingStartTimeUtc : 9/1/2015 12:54:59 AM
	LastBalancingEndTimeUtc   : 9/1/2015 12:54:59 AM
	LoadMetricInformation     :
	                            LoadMetricName        : Metric1
	                            IsBalancedBefore      : False
	                            IsBalancedAfter       : False
	                            DeviationBefore       : 0.192450089729875
	                            DeviationAfter        : 0.192450089729875
	                            BalancingThreshold    : 1
	                            Action                : NoActionNeeded
	                            ActivityThreshold     : 0
	                            ClusterCapacity       : 189
	                            ClusterLoad           : 45
	                            ClusterRemainingCapacity : 144
	                            NodeBufferPercentage  : 10
	                            ClusterBufferedCapacity : 170
	                            ClusterRemainingBufferedCapacity : 125
	                            ClusterCapacityViolation : False
	                            MinNodeLoadValue      : 0
	                            MinNodeLoadNodeId     : 3ea71e8e01f4b0999b121abcbf27d74d
	                            MaxNodeLoadValue      : 15
	                            MaxNodeLoadNodeId     : 2cc648b6770be1bc9824fa995d5b68b1


## 后续步骤
- 有关群集资源管理器中的体系结构和信息流的信息，请查看[此文](/documentation/articles/service-fabric-cluster-resource-manager-architecture/)
- 定义重整指标是合并（而不是分散）节点上负载的一种方式。若要了解如何配置重整，请参阅[此文](/documentation/articles/service-fabric-cluster-resource-manager-defragmentation-metrics/)
- 参阅 [Service Fabric 群集资源管理器简介](/documentation/articles/service-fabric-cluster-resource-manager-introduction/)，帮助自己入门
- 若要了解群集资源管理器如何管理和均衡群集中的负载，请查看有关[均衡负载](/documentation/articles/service-fabric-cluster-resource-manager-balancing/)的文章

[Image1]: ./media/service-fabric-cluster-resource-manager-cluster-description/cluster-fault-domains.png
[Image2]: ./media/service-fabric-cluster-resource-manager-cluster-description/cluster-uneven-fault-domain-layout.png
[Image3]: ./media/service-fabric-cluster-resource-manager-cluster-description/cluster-fault-and-upgrade-domains-with-placement.png
[Image4]: ./media/service-fabric-cluster-resource-manager-cluster-description/cluster-fault-and-upgrade-domain-layout-strategies.png
[Image5]: ./media/service-fabric-cluster-resource-manager-cluster-description/cluster-layout-different-workloads.png
[Image6]: ./media/service-fabric-cluster-resource-manager-cluster-description/cluster-placement-constraints-node-properties.png
[Image7]: ./media/service-fabric-cluster-resource-manager-cluster-description/cluster-nodes-and-capacity.png

<!---HONumber=Mooncake_Quality_Review_0117_2017-->