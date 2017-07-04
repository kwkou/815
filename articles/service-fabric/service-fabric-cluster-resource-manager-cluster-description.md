<properties
    pageTitle="资源平衡器群集描述 | Azure"
    description="通过在群集资源管理器中指定容错域、升级域、节点属性和节点容量描述 Service Fabric 群集。"
    services="service-fabric"
    documentationcenter=".net"
    author="masnider"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="55f8ab37-9399-4c9a-9e6c-d2d859de6766"
    ms.service="Service-Fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="01/05/2017"
    wacn.date="02/20/2017"
    ms.author="masnider" />  


# 描述 Service Fabric 群集
Service Fabric 群集资源管理器提供多种用于描述群集的的机制。在运行时，群集资源管理器使用此信息来确保群集中运行的服务的高可用性。同时强制实施这些重要规则，并尝试优化群集的资源消耗。

## 关键概念
群集资源管理器支持多种用于描述群集的功能：

* 容错域
* 升级域
* 节点属性
* 节点容量

## 容错域
容错域是协调故障的任何区域。一个计算机就是一个容错域（计算机本身可能出于各种原因而发生故障，包括电源故障、驱动器故障、NIC 固件错误等）。连接到同一以太网交换机的计算机位于同一个容错域中，连接到单一电源的计算机也是如此。硬件故障在性质上是重叠的，因此容错域原本就有层次性，在 Service Fabric 中以 URI 的形式表示。

设置自己的群集时，需要考虑这些不同范畴的故障。对容错域进行正确设置很重要，因为 Service Fabric 根据此信息来安全放置服务。Service Fabric 不希望服务的放置方式使得因容错域丢失（因某些组件故障导致）导致服务停止运行。在 Azure 环境中，Service Fabric 利用环境提供的容错域信息，代表用户正确配置群集中的节点。

下图中，我们将所有导致容错域的实体着色，并列出生成的所有不同容错域。此示例列出了数据中心 ("DC")、机架 ("R") 和刀片服务器 ("B")。可以想象，如果每个刀片服务器包含多个虚拟机，则容错域层次结构中可能有另一个层。

<center> 
![通过容错域组织的节点][Image1]
</center>

在运行时，Service Fabric 群集资源管理器会考虑群集中的容错域，并计划布局。它会尝试分散放置服务器的有状态副本或无状态实例，以便使它们位于不同的容错域中。在任何一个容错域（位于层次结构中的任何一个级别）发生故障时，此过程有助于确保该服务的可用性不遭到破坏。

Service Fabric 群集资源管理器不会考虑容错域层次结构中有多少层。但是它会确保丢失任何一部分层次结构都不会影响到在此层次结构以上运行的服务。因此容错域层次结构中的每一个深度级别上的节点数目最好相同。保持级别均衡可以防止一部分层次结构比其他层析结构包含更多的服务。否则将导致单个节点的负载不均衡，并使某些域的故障比其他故障更严重。

群集中容错域的“树”不平衡时，会增大群集资源管理器确定最佳服务分配的难度。容错域布局不均衡意味着丢失特定的域可能会对某些群集的可用性造成更严重的影响。因此群集资源管理器会在两个目标之间举棋不定：将服务放置在“繁忙”域中，以便使用该域中的计算机；还是以某种特定方式放置服务以使域的丢失不会造成问题。这呈现出来是什么样呢？

<center> 
![两种不同的群集布局][Image2]
</center>

上图显示了两个不同的示例群集布局。第一个示例中，节点均匀分散到各容错域中。而另一个示例中，其中一个容错域承载了多得多的节点。如果曾在本地或另一个环境中维护自己的群集，则需要注意这种情况。

在 Azure 中，系统会为用户选择哪个容错域包含节点。但是，根据设置的节点数目，仍然可能出现某容错域比其他容错域承载更多节点的情况。例如，假设有五个容错域，但是针对某个给定节点类型配置了七个节点。在这种情况下，前两个容错域将承载更多的节点。如果继续使用几个实例部署更多的节点类型，问题会变得更严重。

## 升级域
升级域是另一项功能，可帮助 Service Fabric 群集资源管理器了解群集的布局，使它可以针对故障事先做出计划。升级域定义同时升级的节点集。

升级域非常类似于容错域，但有几个重要的差异。首先，容错域由协调硬件故障区域严格定义；升级域由策略定义。使用升级域可自主决定所需数量，而不由环境规定。另一个差异在于（截至目前为止），升级域不是分层的 – 更像是简单的标记。

下图显示了跨三个容错域条带化的三个升级域。该图还显示了有状态服务的三个不同副本的一种可能的放置方式，即三个副本位于不同的容错域和升级域中。这种放置允许在服务升级到半途中丢失容错域，且仍拥有代码和数据的一个副本。

<center> 
![容错域和升级域放置][Image3]
</center>

使用大量升级域既有利也有弊。升级域越多，升级的每个步骤就更精细，影响到的节点或服务就越少。这样每次需要移动的服务数就越少，系统中的流动也越少。这样还可以提高可靠性（因为升级过程中出现的任何问题影响到的服务更少）。升级域越多，还意味着其他节点上需用于处理升级影响的可用开销越少。例如，如果有五个升级域，每个域中的节点处理大约 20% 的流量。如果升级时需要关闭该升级域，则这些负载需要转移到其他地方。升级域越多，意味着在群集中的其他节点上必需维持的开销越少。

拥有多个升级域的缺点是升级需要花费的时间更长。升级域完成后，Service Fabric 将等待一小段时间再继续处理。此延迟可以让升级导致的问题能够显现出来并被检测到。这种折衷是可接受的，因为可以防止错误的更改一次性影响过多的服务。

升级域太少会产生负面影响 – 当每个单独的升级域关闭并进行升级时，大部分的整体容量将不可用。例如，如果只有三个升级域，则一次只能采用整体服务或群集容量的 1/3。让如此多的服务同时停止并非理想的结果，因为必须在其余的群集中拥有足够的容量才能处理工作负荷。维护缓冲区意味着在正常情况下，这些节点处理的负载量降低，由此增加了服务运行的成本。

环境中容错域或升级域的总数没有实际限制，对于它们如何重叠也没有约束。常见的结构为：

* 容错域和升级域按 1:1 映射
* 每个节点一个升级域（物理或虚拟 OS 实例）
* “条带化”或“矩阵”模型，其中的容错域和升级域构成了通常沿对角线分布的计算机矩阵

<center> 
![容错域和升级域布局][Image4]
</center>

至于要选择哪种布局并没有最佳答案，每种做法各有利弊。例如，1FD:1UD 模型相当容易设置。而每个节点 1 个 UD 的模型是最类似于以前人们用于管理少量计算机的模型（其中每个计算机可独立关闭）。

最常见的模型（Azure 中使用的模型）是 FD/UD 矩阵，其中 FD 和 UD 构成一个表，节点沿着对角线开始放置。最后的结构是稀疏还是紧凑取决于相比于 FD 和 UD 数目的节点总数。换而言之，对于足够大的群集，几乎所有布局最终看起来都像是密集矩阵模式，如上图中右下选项所示。

## 容错域与升级域约束及最终行为
群集资源管理器将要在容错域与升级域之间保持服务的均衡视为约束。可以在[此文](/documentation/articles/service-fabric-cluster-resource-manager-management-integration/)中详细了解约束。容错域和升级域约束的定义如下：“针对给定服务分区，两个域之间的服务对象（无状态服务实例或有状态服务副本）的数目差异应该永远不*大于一*”。 这实际上意味着，对于给定的服务而言，特定的移动或排列方式可能无效，因为它们会违反容错域或升级域的约束。

让我们看一个示例。假设有一个 6 节点群集，其中配置了 5 个容错域和 5 个升级域。

| | FD0 | FD1 | FD2 | FD3 | FD4 |
| --- |:---:|:---:|:---:|:---:|:---:|
| **UD0** |N1 | | | | |
| **UD1** |N6 |N2 | | | |
| **UD2** | | |N3 | | |
| **UD3** | | | |N4 | |
| **UD4** | | | | |N5 |

假设要创建一个 TargetReplicaSetSize 为 5 的服务。副本驻留在 N1-N5 上。事实上，无论创建多少个服务，N6 永远不会被使用。为什么？ 看看当前布局和选择 N6 时所发生情况之间的差异。

下面是获得的布局，以及每个容错域和升级域的副本总数：

| | FD0 | FD1 | FD2 | FD3 | FD4 | UDTotal |
| --- |:---:|:---:|:---:|:---:|:---:|:---:|
| **UD0** |R1 | | | | |1 |
| **UD1** | |R2 | | | |1 |
| **UD2** | | |R3 | | |1 |
| **UD3** | | | |R4 | |1 |
| **UD4** | | | | |R5 |1 |
| **FDTotal** |1 |1 |1 |1 |1 |- |

此布局在每个容错域与升级域的节点数目上是均衡的。并且在每个容错域和升级域的副本数目上也是均衡的。每个域都拥有相同数量的节点，以及相同数量的副本。

现在让我们看看，如果不使用 N2，而是改用 N6 会发生什么情况。副本将如何分布？

| | FD0 | FD1 | FD2 | FD3 | FD4 | UDTotal |
| --- |:---:|:---:|:---:|:---:|:---:|:---:|
| **UD0** |R1 | | | | |1 |
| **UD1** |R5 | | | | |1 |
| **UD2** | | |R2 | | |1 |
| **UD3** | | | |R3 | |1 |
| **UD4** | | | | |R4 |1 |
| **FDTotal** |2 |0 |1 |1 |1 |- |

看出来了吗？ 此布局违反了容错域约束的定义。FD0 有 2 个副本，而 FD1 有 0 个副本，FD0 和 FD1 之间的总差异为 2。群集资源管理器不允许这种排列方式。同样，如果选择 N2 和 N6（不选择 N1 和 N2），则会得到：

| | FD0 | FD1 | FD2 | FD3 | FD4 | UDTotal |
| --- |:---:|:---:|:---:|:---:|:---:|:---:|
| **UD0** | | | | | |0 |
| **UD1** |R5 |R1 | | | |2 |
| **UD2** | | |R2 | | |1 |
| **UD3** | | | |R3 | |1 |
| **UD4** | | | | |R4 |1 |
| **FDTotal** |1 |1 |1 |1 |1 |- |

尽管此布局从容错域的角度看是均衡的，但它违反了升级域的约束（因为 UD0 有 0 个副本，而 UD1 有 2 个副本）。此布局也是无效的

## 配置容错域和升级域
对容错域和升级域的定义在 Azure 托管的 Service Fabric 部署中自动完成。Service Fabric 从 Azure 中选择并使用环境信息。

如果正在创建自己的群集（或想要在开发中运行特定的拓扑），则需要自行提供容错域和升级域信息。在此示例中，定义了一个 9 节点本地开发群集，该群集跨三个“数据中心”（每个中心设有三个机架）。该群集还有跨这三个数据中心条带化的三个升级域。在群集清单模板中，将如下所示：

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

通过 ClusterConfig.json 实现独立部署


	"nodes": [
	  {
	    "nodeName": "vm1",
	    "iPAddress": "localhost",
	    "nodeTypeRef": "NodeType0",
	    "faultDomain": "fd:/dc1/r0",
	    "upgradeDomain": "UD1"
	  },
	  {
	    "nodeName": "vm2",
	    "iPAddress": "localhost",
	    "nodeTypeRef": "NodeType0",
	    "faultDomain": "fd:/dc1/r0",
	    "upgradeDomain": "UD2"
	  },
	  {
	    "nodeName": "vm3",
	    "iPAddress": "localhost",
	    "nodeTypeRef": "NodeType0",
	    "faultDomain": "fd:/dc1/r0",
	    "upgradeDomain": "UD3"
	  },
	  {
	    "nodeName": "vm4",
	    "iPAddress": "localhost",
	    "nodeTypeRef": "NodeType0",
	    "faultDomain": "fd:/dc2/r0",
	    "upgradeDomain": "UD1"
	  },
	  {
	    "nodeName": "vm5",
	    "iPAddress": "localhost",
	    "nodeTypeRef": "NodeType0",
	    "faultDomain": "fd:/dc2/r0",
	    "upgradeDomain": "UD2"
	  },
	  {
	    "nodeName": "vm6",
	    "iPAddress": "localhost",
	    "nodeTypeRef": "NodeType0",
	    "faultDomain": "fd:/dc2/r0",
	    "upgradeDomain": "UD3"
	  },
	  {
	    "nodeName": "vm7",
	    "iPAddress": "localhost",
	    "nodeTypeRef": "NodeType0",
	    "faultDomain": "fd:/dc3/r0",
	    "upgradeDomain": "UD1"
	  },
	  {
	    "nodeName": "vm8",
	    "iPAddress": "localhost",
	    "nodeTypeRef": "NodeType0",
	    "faultDomain": "fd:/dc3/r0",
	    "upgradeDomain": "UD2"
	  },
	  {
	    "nodeName": "vm9",
	    "iPAddress": "localhost",
	    "nodeTypeRef": "NodeType0",
	    "faultDomain": "fd:/dc3/r0",
	    "upgradeDomain": "UD3"
	  }
	],


> [AZURE.NOTE]
在 Azure 部署中，由 Azure 分配容错域和升级域。因此，Azure 基础结构选项中节点和角色的定义不包含容错域或升级域信息。
>
>

##<a name="placement-constraints-and-node-properties"></a> 放置约束和节点属性
有时（事实上是大多数情况下），需要确保只在群集中的特定节点或一组特定节点上运行某些工作负荷。例如，某些工作负荷可能需要 GPU 或 SSD，而有些则不用。将特定硬件用于特定工作负载的一个很好的示例是 n 层体系结构（几乎每个这样的体系结构都可看作是一个好例子）。在这些体系结构中，特定计算机充当应用程序的前端/接口服务端（因此通常在 Internet 上公开）。计算层或存储层的工作由不同的计算机组（通常具有不同的硬件资源）负责处理（因此通常不会在 Internet 上公开）。Service Fabric 甚至预期到，在微服务领域中有特定的工作负荷需要在特定硬件配置上运行，例如：

* 现有的 n 层应用程序已“提升并迁移”到 Service Fabric 环境
* 出于性能、规模或安全性隔离原因，某个工作负荷需要在特定硬件上运行
* 出于策略或资源消耗原因，某个工作负荷应该与其他工作负荷隔离

为了支持这种配置，Service Fabric 提供可用于节点的一级标记概念。这些标记称为放置约束。放置约束可用于指示应在何处运行特定服务。约束集可扩展 - 任何键/值对都适用。

<center> 
![群集布局中的不同工作负荷][Image5]
</center>

节点上的不同键/值标记称为节点放置*属性*（或简称为节点属性）。节点属性中指定的值可以是字符串、布尔值或带符号的长型值。服务处的语句称为放置*约束*，它可以约束服务在群集中的运行位置。约束可以是针对群集中的不同节点属性运行的任何布尔值语句。这些布尔值语句中的有效选择器为：

1) 用于创建特定语句的条件检查

| 语句 | 语法 |
| --- |:---:|
| “等于” | "==" |
| “不等于” | "!=" |
| “大于” | ">" |
| “大于等于” | ">=" |
| “小于” | "<" |
| “小于等于” | "<=" |

2) 用于分组和逻辑操作的布尔值语句

| 语句 | 语法 |
| --- |:---:|
| “和” | "&&" |
| “或” | "\|\|" |
| “非” | "!" |
| “分组为单个语句” | "()" |

下面是基本约束语句的一些示例。

  * `"Value >= 5"`  

  * `"NodeColor != green"`
  * `"((OneProperty < 100) || ((AnotherProperty == false) && (OneProperty >= 100)))"`  


只有整个语句求值为“True”的节点才能放置服务。未定义属性的节点不匹配包含该属性的任何放置约束。

Service Fabric 定义了一些默认节点属性，无需用户进行定义，系统即会自动使用这些属性。截至本文发布时，在每个节点上定义的默认属性是 **NodeType **和 **NodeName**。因此举例而言，可以将放置约束编写为 `"(NodeType == NodeType03)"`。通常来说，我们发现 NodeType 是最常用的属性之一。它很有用，因为它与计算机的类型之间存在 1:1 的对应关系，而这又相当于与传统 n 层应用程序体系结构的工作负荷类型发生 1:1 的对应关系。

<center> 
![放置约束和节点属性][Image6]
</center>

假设为给定节点类型定义了以下节点属性：

ClusterManifest.xml

    <NodeType Name="NodeType01">
      <PlacementProperties>
        <Property Name="HasSSD" Value="true"/>
        <Property Name="NodeColor" Value="green"/>
        <Property Name="SomeProperty" Value="5"/>
      </PlacementProperties>
    </NodeType>

通过 ClusterConfig.json 进行独立部署或将 Template.json 用于 Azure 托管群集。在群集的 Azure 资源管理器模板中，节点类型名称等内容可能参数化，会类似于“[parameters('vmNodeType1Name')]”，而不会是“NodeType01”。


	"nodeTypes": [
	    {
	        "name": "NodeType01",
	        "placementProperties": {
	            "HasSSD": "true",
	            "NodeColor": "green",
	            "SomeProperty": "5"
	        },
	    }
	],


可以针对服务创建服务放置*约束*，如下所示：

C#


	FabricClient fabricClient = new FabricClient();
	StatefulServiceDescription serviceDescription = new StatefulServiceDescription();
	serviceDescription.PlacementConstraints = "(HasSSD == true && SomeProperty >= 4)";
	// add other required servicedescription fields
	//...
	await fabricClient.ServiceManager.CreateServiceAsync(serviceDescription);


Powershell：


	New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceType -Stateful -MinReplicaSetSize 2 -TargetReplicaSetSize 3 -PartitionSchemeSingleton -PlacementConstraint "HasSSD == true && SomeProperty >= 4"


如果确定 NodeType01 的所有节点都有效，也可以只选择该节点类型。

服务放置约束的一个突出优点是它们可以在运行时动态更新。因此如果需要，你可以在群集中移动服务、添加和删除要求等。Service Fabric 负责确保服务保持运行且可用，即使此类更改持续进行。

C#：


	StatefulServiceUpdateDescription updateDescription = new StatefulServiceUpdateDescription();
	updateDescription.PlacementConstraints = "NodeType == NodeType01";
	await fabricClient.ServiceManager.UpdateServiceAsync(new Uri("fabric:/app/service"), updateDescription);


Powershell：


	Update-ServiceFabricService -Stateful -ServiceName $serviceName -PlacementConstraints "NodeType == NodeType01"


放置约束（以及即将讨论的许多其他协调器控制）是针对每个不同的命名服务实例指定的。更新始终会取代（覆盖）以前指定的值。

节点上的属性目前是通过群集定义来定义的，因此在不升级群集的情况下无法更新。需要先将每个受影响的节点关闭，然后再重新启用，才能升级节点的属性。

## 容量
任何协调器的最重要作业之一是帮助管理群集中的资源消耗。如果想要有效运行服务，则最不想遇到的情况是一些节点是热的，而其他节点是冷的。热节点导致资源争用和性能不佳，而冷节点代表资源浪费和成本增加。考虑均衡之前，建议首先确保节点不会耗尽资源。

Service Fabric 使用 `Metrics` 表示资源。指标是你想要向 Service Fabric 描述的任何逻辑或物理资源。指标的示例是诸如“WorkQueueDepth”或“MemoryInMb”的参数。有关配置指标及其用法的信息，请参阅[此文](/documentation/articles/service-fabric-cluster-resource-manager-metrics/)

指标与放置约束和节点属性不同。节点属性是节点本身的静态描述符，而指标与节点包含的资源，以及当服务在节点上运行时服务消耗的资源相关。节点属性可能为“HasSSD”，可设置为 true 或 false。但是该 SSD 上的可用空间量（和服务使用的空间量）会是类似于“DriveSpaceInMb”的指标。节点会将其“DriveSpaceInMb”容量纳入驱动器上的非保留空间总量。服务将报告在运行时使用了多少指标。

请注意，与放置约束和节点属性相同，Service Fabric 群集资源管理器不理解指标名称的含义。指标名称只是字符串。建议不明确时，将单位声明为创建的指标名称的一部分。

如果关闭所有资源*均衡*，Service Fabric 群集资源管理器仍将尝试确保最终没有节点超出其容量。通常来说可以这样，除非整个群集过满。容量是群集资源管理器用来了解节点包含的资源量的另一个*约束*。还会针对整个群集来追踪剩余容量。服务级别的容量和消耗量均以指标来表示。例如，指标可能是“MemoryInMb”，给定的节点可能有 2048 个单位的 MemoryInMb 容量。可以这么描述：该节点上的某些服务目前正在消耗 64 个单位的 MemoryInMb。

在运行时，群集资源管理器将跟踪每个节点上的每个资源消耗了多少，以及剩余了多少。从节点容量中减去该节点上运行的每个服务的已声明用量即可得出。使用此信息，Service Fabric 群集资源管理器可找出要放置或移动副本的位置，使节点不会超过容量。

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

<center> 
![群集节点和容量][Image7]
</center>

可以在群集清单中看到定义的容量：

ClusterManifest.xml


    <NodeType Name="NodeType02">
      <Capacities>
        <Capacity Name="MemoryInMb" Value="2048"/>
        <Capacity Name="DiskInMb" Value="512000"/>
      </Capacities>
    </NodeType>

通过 ClusterConfig.json 进行独立部署或将 Template.json 用于 Azure 托管群集。在群集的 Azure 资源管理器模板中，节点类型名称等内容可能参数化，类似于“[parameters('vmNodeType2Name')]”，而不会是“NodeType02”。


	"nodeTypes": [
	    {
	        "name": "NodeType02",
	        "capacities": {
	            "MemoryInMb": "2048",
	            "DiskInMb": "512000"
	        }
	    }
	],

此外，服务的负载也会动态变化（这种情况很常见）。假设副本的负载从 64 更改为 1024，但是当时正在运行该副本的节点上只剩下 512 个单位（“MemoryInMb”度量值）。现在副本或实例的位置无效，因为该节点上没有足够的空间。如果该节点上副本和实例的总用量超出了节点容量，也可能发生这种情况。在这两种情况下，群集资源管理器都必须实施操作，将节点中的用量降低至容量下。将该节点上的一个或多个副本或实例移到其他节点即可。移动副本时，群集资源管理器会尝试将移动成本降至最低。[本文](/documentation/articles/service-fabric-cluster-resource-manager-movement-cost/)中对移动成本进行了讨论。

## 群集容量
那么，我们要如何防止整体群集太满？ 使用动态负载时，实际上群集资源管理器并没有太多可以执行的操作。服务可使自己的负载高峰独立于群集资源管理器所执行的操作。因此，群集当前或许拥有足够的容量，但将来需要扩大规模时，可能就不够用了。话虽如此，但是可以通过加入某些控件来防止基本问题。我们可做的第一件事是防止创建导致群集空间变满的新工作负荷。

假设要创建一个无状态服务，并且它具有某些关联的负载（比默认值还多且动态负载稍后才报告）。假设服务需考虑“DiskSpaceInMb”指标。并且假设服务的每个实例将使用 5 个单位的“DiskSpaceInMb”。需要创建服务的 3 个实例。很好！ 这意味着我们需要群集中有 15 个单位的“DiskSpaceInMb”才能创建这些服务实例。群集资源管理器将持续计算整体容量和每个指标的消耗量，因此可以轻松确定群集中是否有足够的空间。空间不足时，群集资源管理器将拒绝创建服务调用。

因为只要求有 15 个可用单位，所以可以使用不同的方式分配此空间。例如，可能是在 15 个不同节点上各有一个剩余单位的容量，或是在 5 个不同节点上各有三个剩余单位的容量。或是在 3 个节点上有五个单位的可用容量，只要群集资源管理器能够重新安排，就最终会放置服务。此类重新排列几乎始终可行，除非整个群集几乎已满，或者所有服务都非常“庞大”，或者同时出现这两种情况。

##<a name="buffered-capacity"></a> 缓冲容量
群集资源管理器具备的另一个可用于帮助用户管理整体群集容量的功能是针对每个节点的指定容量添加一些保留缓冲区。缓冲容量允许保留整体节点容量的一部分，以便只用于在升级和节点失败期间放置服务。目前缓冲区通过群集定义针对所有节点的每个指标进行全局指定。为保留容量选择的值是一个函数值，涉及群集中容错域和升级域的数量以及所需开销。较多的容错域和升级域意味着可以选择较少数值的缓冲处理容量。域越多，则升级和故障过程中无法使用的群集数量越少。指定缓冲区百分比只有在同时指定了指标的节点容量时才有意义。

以下示例演示如何指定缓冲容量：

ClusterManifest.xml


        <Section Name="NodeBufferPercentage">
            <Parameter Name="DiskSpace" Value="0.10" />
            <Parameter Name="Memory" Value="0.15" />
            <Parameter Name="SomeOtherMetric" Value="0.20" />
        </Section>

通过用于独立部署的 ClusterConfig.json 或用于 Azure 托管群集的 Template.json：


	"fabricSettings": [
	{
		"name": "NodeBufferPercentage",
		"parameters": [
		{
			"name": "DiskSpace",
			"value": "0.10"
		},
		{
			"name": "Memory",
			"value": "0.15"
		},
		{
			"name": "SomeOtherMetric",
			"value": "0.20"
		}
		]
	}
	]


群集用于某个指标的缓冲容量不足时，创建新服务将失败。这样可以确保群集留有足够的备用开销，使升级和故障不会造成节点容量不足。缓冲容量是可选项，但建议为定义了指标容量的所有群集启用。

群集资源管理器通过 PowerShell 和查询 API 公开此信息。由此可查看缓冲容量设置、总容量及群集中使用的每个指标的当前耗用量。下面提供了该输出的示例：

	PS C:\Users\user> Get-ServiceFabricClusterLoadInformation
	LastBalancingStartTimeUtc : 9/1/2016 12:54:59 AM
	LastBalancingEndTimeUtc   : 9/1/2016 12:54:59 AM
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
- 定义碎片整理指标是合并（而不是分散）节点上负载的一种方式。若要了解如何配置重整，请参阅[此文](/documentation/articles/service-fabric-cluster-resource-manager-defragmentation-metrics/)
- 参阅 [Service Fabric 群集资源管理器简介](/documentation/articles/service-fabric-cluster-resource-manager-introduction/)，帮助自己入门
- 若要了解群集资源管理器如何管理和均衡群集中的负载，请查看有关[均衡负载](/documentation/articles/service-fabric-cluster-resource-manager-balancing/)的文章

[Image1]: ./media/service-fabric-cluster-resource-manager-cluster-description/cluster-fault-domains.png
[Image2]: ./media/service-fabric-cluster-resource-manager-cluster-description/cluster-uneven-fault-domain-layout.png
[Image3]: ./media/service-fabric-cluster-resource-manager-cluster-description/cluster-fault-and-upgrade-domains-with-placement.png
[Image4]: ./media/service-fabric-cluster-resource-manager-cluster-description/cluster-fault-and-upgrade-domain-layout-strategies.png
[Image5]: ./media/service-fabric-cluster-resource-manager-cluster-description/cluster-layout-different-workloads.png
[Image6]: ./media/service-fabric-cluster-resource-manager-cluster-description/cluster-placement-constraints-node-properties.png
[Image7]: ./media/service-fabric-cluster-resource-manager-cluster-description/cluster-nodes-and-capacity.png

<!---HONumber=Mooncake_0213_2017-->
<!--Update_Description: add ClusterConfig.json script for "独立部署";add contraint clause introduction; add Template.json for nodetype configation-->