<properties
    pageTitle="了解 Azure Service Fabric | Azure"
    description="Azure Service Fabric 的概述和入门指南。 了解 Service Fabric 并开始开发由微服务组成的可缩放、可靠且易于管理的应用程序。"
    services="service-fabric"
    documentationcenter=".net"
    author="rwike77"
    manager="timlt"
    editor="" />
<tags
    ms.assetid=""
    ms.service="service-fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="04/13/2017"
    wacn.date="05/15/2017"
    ms.author="ryanwi"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="0fbead014b90359c8c4489db95bd7a101e9b89f4"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="so-you-want-to-learn-about-service-fabric"></a>想要了解 Service Fabric 吗？
此入门提供了 Service Fabric 的简要概述、对核心概念和术语的简介、入门指南以及对 Service Fabric 各个部分的概述。  此入门不包含全面的内容列表，但提供 Service Fabric 每个部分的概述和入门文章的链接。 

## <a name="the-five-minute-overview"></a>五分钟概述
Azure Service Fabric 是一种分布式系统平台，适用于打包、部署和管理可缩放的可靠微服务。  Service Fabric 解决了开发和管理云应用程序中的重大难题。 通过使用 Service Fabric，开发人员和管理员可以避免解决复杂的基础结构问题。  你可以专注于实现任务关键型、要求苛刻的工作负荷，你知道它们可缩放、可靠且易于管理。 Service Fabric 代表用于生成和管理这些企业级的一级云规模应用程序的下一代中间件平台。  

第 9 频道中的这部简短视频介绍了 Service Fabric 和微服务：
<center><a target="_blank" href="https://aka.ms/servicefabricvideo">  
<img src="./media/service-fabric-content-roadmap/OverviewVid.png" WIDTH="360" HEIGHT="244">  
</a></center>

## <a name="the-detailed-overview"></a>详细概述
借助 Service Fabric，可以生成和管理由微服务构成的可缩放且可靠的应用程序。  这些微服务在计算机的共享池上以高密度运行，它们被称为群集。 它可以提供复杂运行时，用于构建分布式、可扩展的无状态和有状态微服务。 它还提供全面的应用程序管理功能，用于预配、部署、监视、升级/修补和删除部署的应用程序。  请阅读 [Service Fabric 概述](/documentation/articles/service-fabric-overview/)了解详细信息。

为何要使用微服务设计模式？ 所有应用程序会随着时间而发展。 成功的应用程序因为有实用性而发展。 现在对要求了解多少，未来又有何变化？ 在已知以后可以重新设计应用程序的情况下，有时向外寻求概念证明才是驱动因素。 另一方面，公司谈论构建云时都期望成长和使用量。 问题在于成长和规模不可预测。 在知道应用可缩放以对不可预测的成长和使用量做出反应的情况下，开发人员希望快速获得原型。  [什么是微服务？](/documentation/articles/service-fabric-overview-microservices/)说明了微服务设计方法如何应对这些挑战，以及如何创建可独立扩展或缩减、测试、部署和管理的微服务。

Service Fabric 提供了一个可靠而灵活的平台，使你能够编写和运行多种类型的业务应用程序与服务。  你还可以运行任何现有应用程序（以任何语言编写）。  这些应用程序和微服务可以无状态也可以有状态，它们在各虚拟机间资源平衡，可最大限度提高工作效率。 Service Fabric 的独特体系结构使你可以在应用程序中执行近实时数据分析、内存中计算、并行事务和事件处理。 你可以根据不断变化的资源要求轻松[向上或向下缩放应用程序](/documentation/articles/service-fabric-concepts-scalability/)（其实是扩展或缩减）。 阅读[应用程序方案](/documentation/articles/service-fabric-application-scenarios/)，了解可创建的应用程序和服务的类别以及客户案例研究。

这部较长的 Microsoft 虚拟大学视频介绍了 Service Fabric 的核心概念：
<center><a target="_blank" href="https://mva.microsoft.com/en-US/training-courses/building-microservices-applications-on-azure-service-fabric-16747?l=tbuZM46yC_5206218965">  
<img src="./media/service-fabric-content-roadmap/CoreConceptsVid.png" WIDTH="360" HEIGHT="244">  
</a></center>

## <a name="get-started-and-create-your-first-app"></a>入门并创建第一个应用 
使用 Service Fabric SDK 和工具可在 Windows、Linux 或 MacOS 环境中开发应用，并将这些应用程序部署到 Windows 或 Linux 上运行的群集。  以下指南可帮助你在短时间内部署应用。 

### <a name="on-windows"></a>在 Windows 上
Service Fabric SDK 包含一个用于 Visual Studio 的外接程序，它可提供用于创建、部署和调试 Service Fabric 应用程序的模板和工具。 这些主题会指导你完成在 Visual Studio 中创建你的第一个应用程序，并在开发计算机上运行该程序的过程。

[设置开发环境](/documentation/articles/service-fabric-get-started/)
[创建第一个应用 (C#)](/documentation/articles/service-fabric-create-your-first-application-in-visual-studio/)

#### <a name="practical-hands-on-labs"></a>实际动手实验
学习这篇内容丰富的 [动手实验第 1 部分](https://msdnshared.blob.core.windows.net/media/2016/07/SF-Lab-Part-I.docx) ，熟悉 Service Fabric 的端到端开发流。  了解如何创建一个无状态服务、配置监视和运行状况报告并执行应用程序升级。 阅读第 1 部分后，请参阅[动手实验第 2 部分](http://aka.ms/sflab2)，其将引导你创建有状态服务。

下面的 Channel9 视频会指导你完成在 Visual Studio 中创建 C# 应用的过程：  
<center><a target="_blank" href="https://channel9.msdn.com/Blogs/Azure/Creating-your-first-Service-Fabric-application-in-Visual-Studio">  
<img src="./media/service-fabric-content-roadmap/first-app-vid.png" WIDTH="360" HEIGHT="244">  
</a></center>

### <a name="on-linux"></a>在 Linux 上
Service Fabric 提供用于在 Linux 上构建服务的 .NET Core 和 Java SDK。 这些主题会指导你完成在 Linux 上创建你的第一个 Java 或 C# 应用程序，并在开发计算机上运行该程序的过程。
[设置开发环境](/documentation/articles/service-fabric-get-started-linux/)
[创建第一个应用 (Java)](/documentation/articles/service-fabric-create-your-first-linux-application-with-java/)
[创建第一个应用 (C#)](/documentation/articles/service-fabric-create-your-first-linux-application-with-csharp/)

以下 Microsoft 虚拟大学视频逐步讲解了在 Linux 上创建 Java 应用的过程：  
<center><a target="_blank" href="https://mva.microsoft.com/en-US/training-courses/building-microservices-applications-on-azure-service-fabric-16747?l=DOX8K86yC_206218965">  
<img src="./media/service-fabric-content-roadmap/LinuxVid.png" WIDTH="360" HEIGHT="244">  
</a></center>

### <a name="on-macos"></a>在 MacOS 上
可以在 MacOS X 中构建可在 Linux 群集上运行的 Service Fabric 应用程序。 这些文章介绍了如何在 Mac 中设置开发环境，并逐步讲解了如何在 MacOS 上创建 Java 应用程序，然后在 Ubuntu 虚拟机上运行它。
[设置开发环境](/documentation/articles/service-fabric-get-started-mac/)
[创建第一个应用 (Java)](/documentation/articles/service-fabric-create-your-first-linux-application-with-java/)

## <a name="core-concepts"></a>核心概念
此处介绍基础知识，有关详细概念和介绍，可参阅 [Service Fabric 术语](/documentation/articles/service-fabric-technical-overview/)、[应用程序模型](/documentation/articles/service-fabric-application-model/)和[支持的编程模型](/documentation/articles/service-fabric-choose-framework/)。

<table><tr><th>核心概念</th><th>设计时</th><th>运行时</th></tr>
<tr><td><a target="_blank" href="https://mva.microsoft.com/en-us/training-courses/building-microservices-applications-on-azure-service-fabric-16747?l=tbuZM46yC_5206218965"> <img src="./media/service-fabric-content-roadmap/CoreConceptsVid.png" WIDTH="240" HEIGHT="162"></a></td>
<td><a target="_blank" href="https://mva.microsoft.com/en-us/training-courses/building-microservices-applications-on-azure-service-fabric-16747?l=tlkI046yC_2906218965"><img src="./media/service-fabric-content-roadmap/RunTimeVid.png" WIDTH="240" HEIGHT="162"></a></td>
<td><a target="_blank" href="https://mva.microsoft.com/en-us/training-courses/building-microservices-applications-on-azure-service-fabric-16747?l=x7CVH56yC_1406218965"> <img src="./media/service-fabric-content-roadmap/RunTimeVid.png" WIDTH="240" HEIGHT="162"></a></td></tr>
</table>

### <a name="design-time-app-type-service-type-app-package-and-manifest-service-package-and-manifest"></a>设计时：应用类型、服务类型，应用包和清单、服务包和清单
应用程序类型是分配给服务类型集合的名称/版本。 在 ApplicationManifest.xml 文件中定义，在应用程序包目录中嵌入，并复制到 Service Fabric 群集的映像存储中。 然后，可基于此应用程序类型，创建在群集内运行的命名应用程序。 

服务类型是分配给服务的代码包、数据包、配置包的名称/版本。 服务类型在 ServiceManifest.xml 文件中定义，在服务包目录中嵌入，然后，应用程序包的 ApplicationManifest.xml 文件将引用该服务包目录。 在群集中创建命名应用程序后，你可以从应用程序类型的服务类型之一创建命名服务。 服务类型由其 ServiceManifest.xml 文件描述。  服务类型由在运行时加载的可执行代码服务配置设置和服务使用的静态数据组成。

![Service Fabric 应用程序类型和服务类型][cluster-imagestore-apptypes]  

应用程序包是一个磁盘目录，其中包含应用程序类型的 ApplicationManifest.xml 文件，该文件引用构成应用程序类型的每种服务类型的服务包。 例如，电子邮件应用程序类型的应用程序包可能包含对队列服务包、前端服务包和数据库服务包的引用。 应用程序包目录中的文件将复制到 Service Fabric 群集的映像存储中。 

服务包是包含服务类型的 ServiceManifest.xml 文件的磁盘目录，引用服务类型的代码、静态数据和配置包。 应用程序类型的 ApplicationManifest.xml 文件引用服务包目录中的文件。 例如，服务包可能引用构成数据库服务的代码、静态数据和配置包。

### <a name="run-time-clusters-and-nodes-named-apps-named-services-partitions-and-replicas"></a>运行时：群集和节点、命名应用、命名服务、分区和副本
[Service Fabric 群集](/documentation/articles/service-fabric-deploy-anywhere/)是一组通过网络连接在一起的虚拟机或物理计算机，你的微服务将在其中部署和管理。 群集可以扩展到成千上万台计算机。

构成群集的计算机或 VM 称为节点。 需为每个节点分配节点名称（字符串）。 节点具有各种特征，如放置属性。 每个计算机或 VM 都有一个自动启动 Windows 服务 FabricHost.exe，此服务在引导时开始运行，然后启动两个可执行文件：Fabric.exe 和 FabricGateway.exe。 这两个可执行文件构成了节点。 在开发或测试方案中，可以通过运行 Fabric.exe 和 FabricGateway.exe 的多个实例，在单台计算机或 VM 上托管多个节点。

命名应用程序是由执行一个或多个特定功能的命名服务组成的集合。 服务执行完整且独立的功能（它们可以独立于其他服务启动和运行），并由代码、配置和数据组成。 将应用程序包复制到映像存储后，可以通过指定应用程序包的应用程序类型（使用其名称/版本）在群集内创建应用程序的实例。 将为每个应用程序类型实例分配一个如下所示的 URI 名称：*fabric:/MyNamedApp*。 在群集中，你可以从单个应用程序类型创建多个命名应用程序。 也可以从不同的应用程序类型创建命名应用程序。 可单独管理每个命名应用程序并设置其版本。

创建命名应用程序后，可以通过指定服务类型（使用其名称/版本），在群集中创建应用程序服务类型之一的实例（命名服务）。 需为每个服务类型实例分配一个 URI 名称，该名称归并到实例的命名应用程序的 URI 之下。 例如，如果在命名应用程序“MyNamedApp”中创建命名服务“MyDatabase”，则 URI 将类似于： *fabric:/MyNamedApp/MyDatabase*。 在一个命名应用程序中可以创建一个或多个命名服务。 每个命名服务可以有自身的分区方案和实例/副本计数。 

有两种服务类型：无状态服务和有状态服务。  无状态服务将持久状态存储在 Azure 存储、Azure SQL 数据库、Azure DocumentDB 等外部存储服务中。 当服务根本没有永久性存储时，请使用无状态服务。 有状态服务使用 Service Fabric 通过其 Reliable Collections 或 Reliable Actors 编程模型管理服务的状态。  

创建命名服务时，需要指定一个分区方案。 包含大量状态的服务将跨分区拆分数据。  每个分区负责服务完整状态的一部分（分布在群集的节点上）。  在分区中，无状态命名服务具有实例，而有状态命名服务具有副本。 通常，无状态命名服务只有一个分区，因为它们没有内部状态。 有状态命名服务在副本中保持其状态，每个分区都有自己的副本集。  在一个副本（以下称为“主副本”）上执行读取和写入操作。 因写入操作发生的状态更改将复制到其他多个副本（以下称为活动辅助副本）。  

下图显示应用程序和服务实例、分区与副本之间的关系。

![服务中的分区和副本][cluster-application-instances]

### <a name="partitioning-scaling-and-availability"></a>分区、缩放和可用性
[分区](/documentation/articles/service-fabric-concepts-partitioning/)并不是 Service Fabric 所独有的。 一种众所周知的分区形式是数据分区，也称为分片。 包含大量状态的有状态服务将跨分区拆分数据。  每个分区负责服务完整状态的一部分。 

每个分区的副本分布在群集的节点上，以便命名服务状态进行[缩放](/documentation/articles/service-fabric-concepts-scalability/)。 随着数据需求的增长，分区也会增长，Service Fabric 会在节点间重新平衡分区，以高效利用硬件资源。 如果向群集添加新节点，Service Fabric 会在新增加的节点间重新平衡分区副本。  应用程序总体性能提高，访问内存的争用减少。  如果没有高效使用群集中的节点，可以减少群集中节点的数量。  Service Fabric 会再次在减少的节点间重新平衡分区副本以更加充分使用每个节点上的硬件。

在分区中，无状态命名服务具有实例，而有状态命名服务具有副本。 通常，无状态命名服务只有一个分区，因为它们没有内部状态。 分区实例提供[可用性](/documentation/articles/service-fabric-availability-services/)。  如果一个实例失败，其他实例可继续正常运行，然后 Service Fabric 将创建新的实例。 有状态命名服务在副本中保持其状态，每个分区都有自己的副本集。  在一个副本（以下称为“主副本”）上执行读取和写入操作。 因写入操作发生的状态更改将复制到其他多个副本（以下称为活动辅助副本）。  如果某个副本失败，Service Fabric 将从现有副本创建新副本。

## <a name="supported-programming-models"></a>支持的编程模型
Service Fabric 提供了多种方法来编写和管理服务。 服务可以选择使用 Service Fabric API 来充分利用平台的功能和应用程序框架。  或者，服务可以是采用任何语言编写的任意编译可执行程序，并在 Service Fabric 群集上托管。 有关详细信息，请参阅[支持的编程模型](/documentation/articles/service-fabric-choose-framework/)。

### <a name="guest-executables"></a>来宾可执行文件
[来宾可执行文件](/documentation/articles/service-fabric-deploy-existing-app/)是（采用任何语言编写的）任意现有可执行文件，并在 Service Fabric 群集及其他服务上托管。 但是，来宾可执行文件不直接与 Service Fabric API 集成。  来宾可执行文件不会从平台所提供的完整功能集中获益，例如自定义健康和负载报告、服务终结点注册和有状态计算。

### <a name="containers"></a>容器
默认情况下，Service Fabric 以进程形式部署和激活服务。 Service Fabric 还可以部署[容器映像](/documentation/articles/service-fabric-containers-overview/)中的服务。 重要的是，可以在同一应用程序中混合使用进程中的服务和容器中的服务。  目前，Service Fabric 支持在 Linux 上部署 Docker 容器。 在 Service Fabric 应用程序模型中，容器代表多个服务副本所在的应用程序主机。  可以使用 Service Fabric 在容器中部署现有应用程序、无状态服务或有状态服务。  

### <a name="reliable-services"></a>Reliable Services
[Reliable Services](/documentation/articles/service-fabric-reliable-services-introduction/) 是一个用于编写服务的轻型框架，这些服务与 Service Fabric 平台集成并且受益于完整的平台功能集。 Reliable Services 可以是无状态的（类似于大多数服务平台，例如 Azure 云服务中的 Web 服务器或辅助角色）。 状态将在外部解决方案（例如 Azure DB 或 Azure 表存储）中保留。 Reliable Services 也可以是有状态的，此时状态使用 Reliable Collections 直接保存在服务中。 通过复制使状态具有[高可用性](/documentation/articles/service-fabric-availability-services/)，以及通过[分区](/documentation/articles/service-fabric-concepts-partitioning/)来分布状态，所有状态由 Service Fabric 自动管理。

### <a name="reliable-actors"></a>Reliable Actors
[Reliable Actor](/documentation/articles/service-fabric-reliable-actors-introduction/) 框架在 Reliable Services 的基础上构建，是根据执行组件设计模式实现虚拟执行组件模式的应用程序框架。 Reliable Actor 框架使用称为执行组件的单线程执行的独立的计算单元和状态。 Reliable Actor 框架为执行组件提供内置通信，以及提供预设的状态暂留和扩展配置。

## <a name="clusters"></a>群集
[Service Fabric 群集](/documentation/articles/service-fabric-deploy-anywhere/)是一组通过网络连接在一起的虚拟机或物理计算机，你的微服务将在其中部署和管理。 群集可以扩展到成千上万台计算机。 群集中的计算机或 VM 称为群集节点。 需为每个节点分配节点名称（字符串）。 节点具有各种特征，如放置属性。 每个计算机或 VM 都有一个自动启动服务 FabricHost.exe，此服务在引导时开始运行，然后启动两个可执行文件：Fabric.exe 和 FabricGateway.exe。 这两个可执行文件构成了节点。 在测试方案中，可以通过运行 Fabric.exe 和 FabricGateway.exe 的多个实例，在单台计算机或 VM 上托管多个节点。

可在运行 Windows Server 或 Linux 的任何虚拟机或计算机上创建 Service Fabric 群集。 可在包含一组互连 Windows Server 或 Linux 计算机（本地计算机、Azure 计算机或任何云提供商的计算机）的任何环境中部署和运行 Service Fabric 应用程序。

### <a name="clusters-on-azure"></a>Azure 上的群集
在 Azure 上运行 Service Fabric 群集可提供与其他 Azure 功能和服务的集成，这样可使群集的操作和管理更容易且更可靠。  群集是 Azure Resource Manager 资源，因此可以像 Azure 中的其他资源一样对群集进行建模。 Resource Manager 还可以轻松管理群集作为单个单元使用的所有资源。  群集 nodetype 是[虚拟机规模集](/documentation/services/virtual-machine-scale-sets/)，因此自动缩放功能是内置的。

可以通过 [Azure 门户预览](/documentation/articles/service-fabric-cluster-creation-via-portal/)、[模板](/documentation/articles/service-fabric-cluster-creation-via-arm/)或 [Visual Studio](/documentation/articles/service-fabric-cluster-creation-via-visual-studio/) 在 Azure 上创建群集。

如同在 Windows 上一样，可以使用 Linux 上的 Service Fabric 预览版在 Linux 上构建、部署和管理高可用性、高度可缩放的应用程序。 Service Fabric 框架（Reliable Services 和 Reliable Actors）除了可在 C# (.NET Core) 中使用以外，也能在 Java on Linux 中使用。  还可以使用任何语言或框架来构建[来宾可执行的服务](/documentation/articles/service-fabric-deploy-existing-app/)。 此外，预览版还支持协调 Docker 容器。 Docker 容器可以运行使用 Service Fabric 框架的来宾可执行文件或本机 Service Fabric 服务。 有关详细信息，请参阅 [Linux 上的 Service Fabric](/documentation/articles/service-fabric-linux-overview/)。

由于 Linux 上的 Service Fabric 是预览版，因此某些在 Windows 上受支持的功能，在 Linux 上不受支持。 若要了解详细信息，请参阅 [Linux 上的 Service Fabric 与 Windows 上的 Service Fabric 之间的差异](/documentation/articles/service-fabric-linux-windows-differences/)。

### <a name="standalone-clusters"></a>独立群集
Service Fabric 提供一个安装包，用于在本地或者与任何云提供商合作创建独立的 Service Fabric 群集。  独立群集可让你随时随地托管群集。  如果你的数据受符合性或法规约束，或者需要将数据保留在本地，可以托管你自己的群集和应用。  Service Fabric 可在多个托管环境中运行，而不发生任何更改，因此你对生成应用的了解在各个托管环境中都适用。 

[创建第一个独立的 Service Fabric 群集](/documentation/articles/service-fabric-get-started-standalone-cluster/)

尚不支持 Linux 独立群集。

### <a name="cluster-security"></a>群集安全性
必须保护群集以防止未经授权的用户连接到群集，特别是群集上正在运行生产工作负荷时。 虽然可以创建不安全的群集，但这样做会导致在向公共 Internet 公开管理终结点时，匿名用户可与它建立连接。  无法以后在不安全的群集上启用安全性：群集安全性在群集创建时启用。

群集安全方案包括：
* 节点到节点安全性
* 客户端到节点安全性
* 基于角色的访问控制 (RBAC)

有关详细信息，请阅读[保护群集](/documentation/articles/service-fabric-cluster-security/)。

### <a name="scaling"></a>扩展
如果向群集添加新节点，Service Fabric 会在新增加的节点间重新平衡分区副本和实例。  应用程序总体性能提高，访问内存的争用减少。  如果没有高效使用群集中的节点，可以减少群集中节点的数量。  Service Fabric 会再次在减少的节点间重新平衡分区副本和实例以更加充分利用每个节点上的硬件。 可以[手动](/documentation/articles/service-fabric-cluster-windows-server-add-remove-nodes/)扩展独立群集。

### <a name="cluster-upgrades"></a>群集升级
我们会定期发布新版本的 Service Fabric 运行时。  在群集上执行运行时或结构升级，以便你始终运行[受支持的版本](/documentation/articles/service-fabric-support/)。  除了结构升级，还可以更新群集配置（例如证书或应用程序端口）。

Service Fabric 群集是你拥有的，但部分由 Azure 管理的资源。  我们负责修补基础 OS 并在群集上执行结构升级。  当我们发布新版本时，你可以将群集设置为接收自动结构升级，或选择所需的受支持结构版本。  可通过 Azure 门户预览或 Resource Manager 设置结构和配置升级。  有关详细信息，请参阅[升级 Service Fabric 群集](/documentation/articles/service-fabric-cluster-upgrade/)。  

独立群集是你完全拥有的资源。  你负责修补基础 OS 和启动结构升级。  如果群集可以连接到 [https://www.microsoft.com/download](https://www.microsoft.com/download)，可以将群集设置为自动下载并预配新的 Service Fabric 运行时包。  然后会启动升级。  如果群集无法访问 [https://www.microsoft.com/download](https://www.microsoft.com/download)，可以从连接 Internet 的计算机手动下载新的运行时包，然后启动升级。  有关详细信息，请参阅[升级独立 Service Fabric 群集](/documentation/articles/service-fabric-cluster-upgrade-windows-server/)。

## <a name="app-lifecycle"></a>应用生命周期
与其他平台一样，Service Fabric 上的应用程序通常将经历以下几个阶段：设计、开发、测试、部署、升级、维护和删除。 Service Fabric 为云应用程序的整个应用程序生命周期提供一流的支持：从开发到部署、到日常管理和维护，再到最终解除授权。 服务模型使多个不同角色可以独立参与到应用程序生命周期中。 [Service Fabric 应用程序生命周期](/documentation/articles/service-fabric-application-lifecycle/)一文提供了有关 API 的概述，以及在 Service Fabric 应用程序生命周期的各个阶段，它们是如何被不同角色所使用的。  

可以使用 [PowerShell cmdlet](https://docs.microsoft.com/zh-cn/powershell/module/ServiceFabric/)、[C# API](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.applicationmanagementclient)、[Java API](https://docs.microsoft.com/java/api/system.fabric._application_management_client) 和 [REST API](https://docs.microsoft.com/zh-cn/rest/api/servicefabric/) 管理整个应用生命周期。 还可以使用 [Visual Studio Team Services](/documentation/articles/service-fabric-set-up-continuous-integration/) 或 [Jenkins](/documentation/articles/service-fabric-cicd-your-linux-java-application-with-jenkins/) 等工具来设置连续集成/连续部署管道

以下 Microsoft Virtual Academy 视频介绍了如何管理应用程序生命周期： <center><a target="_blank" href="https://mva.microsoft.com/en-US/training-courses/building-microservices-applications-on-azure-service-fabric-16747?l=My3Ka56yC_6106218965">
<img src="./media/service-fabric-content-roadmap/AppLifecycleVid.png" WIDTH="360" HEIGHT="244">
</a></center>

## <a name="health-monitoring"></a>运行状况监视
Service Fabric 引入了[运行状况模型](/documentation/articles/service-fabric-health-introduction/)，用于在特定实体上标记不正常的群集和应用程序状态（例如群集节点和服务副本）。 运行状况模型使用运行状况报告器（系统组件和监视器）。 其目标是实现轻松快捷的诊断和修复。 服务写入程序需要预先考虑运行状况和如何[设计运行状况报告](/documentation/articles/service-fabric-report-health/#design-health-reporting)。 应报告任何可能会影响运行状况的条件，尤其是如果它有助于标记出接近根源的问题。 在生产中大规模启动并运行服务后，运行状况信息可以减少调试和调查工作所需的时间和精力。

Service Fabric 报告器可监视感兴趣的已标识条件。 它们会根据其本地视图报告这些条件。 [运行状况存储](/documentation/articles/service-fabric-health-introduction/#health-store)可汇总所有报告器发送的运行状况数据，从而确定实体的全局运行状况是否正常。 该模型应具有功能丰富、灵活且易于使用的特点。 运行状况报告的质量决定了群集运行状况视图的准确度。 错误显示不正常问题的误报会对升级或其他使用运行状况数据的服务产生负面影响。 修复服务和警报机制就是此类服务的例子。 因此，提供报表时需多加考量，才能让其以尽可能最佳的方式捕获感兴趣的条件。

可以基于以下内容完成报告：
* 受监视的 Service Fabric 服务副本或实例。
* 内部监视器部署为 Service Fabric 服务（例如，可监视状态和问题报告的 Service Fabric 无状态服务）。 可以在所有节点上部署监视器，也可以将监视器与受监视的服务关联。
* 在 Service Fabric 节点上运行，但未以 Service Fabric 服务形式实现的内部监视器。
* 从 Service Fabric 群集外探测资源的外部监视器（例如，诸如 Gomez 之类的监视服务）。

Service Fabric 组件报告包含群集中所有实体的运行状况。  [系统运行状况报告](/documentation/articles/service-fabric-understand-and-troubleshoot-with-system-health-reports/)提供有关群集和应用程序功能的可见性，并且通过运行状况标记问题。 对于应用程序和服务，系统运行状况报告从 Service Fabric 运行时的角度验证实体得到实现并且正常运行。 报告不对服务的业务逻辑进行任何运行状况监视，也不检测暂停的进程。 若要添加特定于服务逻辑的运行状况信息，请在服务中[执行自定义运行状况报告](/documentation/articles/service-fabric-report-health/)。

Service Fabric 提供了多种方式查看在运行状况存储中聚合的[运行状况报告](/documentation/articles/service-fabric-view-entities-aggregated-health/)：
* [Service Fabric Explorer](/documentation/articles/service-fabric-visualizing-your-cluster/) 或其他可视化工具/
* 运行状况查询（通过 [PowerShell](https://docs.microsoft.com/zh-cn/powershell/module/ServiceFabric/)、[C# FabricClient Api](https://docs.microsoft.com/api/system.fabric.fabricclient.healthclient) 和 [Java FabricClient API](https://docs.microsoft.com/java/api/system.fabric._health_client) 或 [REST API](https://docs.microsoft.com/zh-cn/rest/api/servicefabric)）。
* 常规查询，返回将运行状况作为属性之一的实体的列表（通过 PowerShell、API 或 REST）。

以下 Microsoft Virtual Academy 视频介绍 Service Fabric 运行状况模型及其用法：<center><a target="_blank" href="https://mva.microsoft.com/en-US/training-courses/building-microservices-applications-on-azure-service-fabric-16747?l=tevZw56yC_1906218965">
<img src="./media/service-fabric-content-roadmap/HealthIntroVid.png" WIDTH="360" HEIGHT="244">
</a></center>

## <a name="next-steps"></a>后续步骤
* 了解如何[在 Azure 中创建群集](/documentation/articles/service-fabric-cluster-creation-via-portal/)或[在 Windows 上创建独立群集](/documentation/articles/service-fabric-cluster-creation-for-windows-server/)。
* 尝试使用 [Reliable Services](/documentation/articles/service-fabric-reliable-services-quick-start/) 或 [Reliable Actors](/documentation/articles/service-fabric-reliable-actors-get-started/) 编程模型创建服务。
* 了解如何[从云服务迁移](/documentation/articles/service-fabric-cloud-services-migration-differences/)。
* 学习[监视和诊断服务](/documentation/articles/service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally/)。 
* 了解如何[测试应用和服务](/documentation/articles/service-fabric-testability-overview/)。
* 了解如何[管理和协调群集资源](/documentation/articles/service-fabric-cluster-resource-manager-introduction/)。
* 了解 [Service Fabric 支持选项](/documentation/articles/service-fabric-support/)。


[cluster-application-instances]: ./media/service-fabric-content-roadmap/cluster-application-instances.png
[cluster-imagestore-apptypes]: ./media/service-fabric-content-roadmap/cluster-imagestore-apptypes.png
<!--Update_Description:add introduction to "分区、缩放和可用性";update next steps links-->