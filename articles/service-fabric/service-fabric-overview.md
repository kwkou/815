<properties
    pageTitle="Service Fabric 概述 | Azure"
    description="Service Fabric 概述，其中应用程序由许多微服务组成，以提供扩展性和恢复能力。Service Fabric 是一个分布式系统平台，用于生成面向云的可缩放、可靠且易于管理的应用程序。"
    services="service-fabric"
    documentationcenter=".net"
    author="msfussell"
    manager="timlt"
    editor="masnider" />
<tags
    ms.assetid="bbcc652a-a790-4bc4-926b-e8cd966587c0"
    ms.service="service-fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="01/05/2017"
    wacn.date="02/20/2017"
    ms.author="mfussell" />  


# Azure Service Fabric 概述
Service Fabric 是分布式系统平台，可借助它轻松打包、部署和管理可缩放且可靠的微服务。Service Fabric 还解决了开发和管理云应用程序中的重大难题。开发人员和管理员不仅可以避免复杂的基础结构问题，而且可以专注于实现可缩放、可靠且可管理的要求苛刻的任务关键型工作负荷。Service Fabric 代表用于生成和管理这些企业级第 1 层云规模应用程序的下一代中间件平台。

第 9 频道中的这部简短视频介绍了 Service Fabric 和微服务：
<center><a target="_blank" href="https://aka.ms/servicefabricvideo"> 
<img src="./media/service-fabric-overview/OverviewVid.png" WIDTH="360" HEIGHT="244"> 
</a></center>

这部较长的 Microsoft 虚拟大学视频介绍了 Service Fabric 的核心概念：
<center><a target="_blank" href="https://mva.microsoft.com/training-courses/building-microservices-applications-on-azure-service-fabric-16747?l=tbuZM46yC_5206218965"> 
<img src="./media/service-fabric-overview/CoreConceptsVid.png" WIDTH="360" HEIGHT="244"> 
</a></center>


## 由微服务组成的应用程序
利用 Service Fabric，可以生成并管理由微服务组成的应用程序，这些应用程序可缩放且可靠，以非常高的密度在计算机共享池（称为群集）上运行。它提供复杂的运行时，可生成分布式、可缩放的无状态和有状态微服务。它还提供全面的应用程序管理功能，用于预配、部署、监视、升级/修补和删除部署的应用程序。

为什么微服务方法很重要？ 有以下两个主要原因：

* 能够根据应用程序的需求缩放应用程序的不同部分。
* 开发团队可以更加灵活地推出更改，从而更快速和更频繁地向客户提供功能。

Service Fabric 为当今很多 Microsof 服务提供支持，包括 Azure SQL 数据库、Azure DocumentDB、Cortana、Power BI、Microsoft Intune、Azure 事件中心、Azure IoT、Skype for Business 和许多核心 Azure 服务。

Service Fabric 专为创建云端本机服务而定制，这些服务可以根据需要从小规模开始，然后成长为包含数百或数千台计算机的大规模服务。

当今的 Internet 规模的服务是使用微服务构建而成的。微服务的例子包括协议网关、用户配置文件、购物车，清单处理、排队和缓存等。Service Fabric 是微服务平台，为每个微服务提供可以是无状态或有状态的唯一名称。

Service Fabric 为由这些微服务组成的应用程序提供全面的运行时和生命周期管理功能。它在 Service Fabric 群集间部署和激活的容器内部托管微服务。从虚拟机改为使用容器可能使密度出现数量级增长。同样，从容器改为使用微服务也可能使密度再次出现数量级增长。例如，单个 Azure SQL 数据库群集包含数百台计算机，这些计算机运行数以万计的容器，而这些容器总共托管数十万个数据库。每个数据库都是一个 Service Fabric 有状态微服务。这同样适用于前文提及的其他服务，正因如此，我们使用*“超大规模”*一词来描述 Service Fabric 功能。如果容器为你提供了高密度，那么微服务可为你提供超大规模。

有关微服务方法的详细信息，请阅读[微服务方法为什么可以生成应用程序？](/documentation/articles/service-fabric-overview-microservices/)

## 容器部署和业务流程
Service Fabric 是跨计算机群集的微服务[协调器](/documentation/articles/service-fabric-cluster-resource-manager-introduction/)。开发微服务的方式多种多样：使用 [Service Fabric 编程模型](/documentation/articles/service-fabric-choose-framework/)，或者部署[来宾可执行文件](/documentation/articles/service-fabric-deploy-existing-app/)，不一而足。Service Fabric 可在容器映像中部署服务。重要的是，可以在同一应用程序中混合使用进程中的服务和容器中的服务。
## 随处创建 Service Fabric 群集
可在许多环境（包括在 Azure 中或本地、在 Windows Server 或 Linux 上）中创建 Service Fabric 群集。此外，SDK 中的开发环境与生产环境完全相同，都不涉及模拟器。换而言之，在本地开发群集上运行的对象会部署到其他环境中的同一群集。

有关在本地创建群集的详细信息，请阅读[在 Windows Server](/documentation/articles/service-fabric-deploy-anywhere/)，或[通过 Azure 门户](/documentation/articles/service-fabric-cluster-creation-via-portal/)针对 Azure 创建群集。

![Service Fabric 平台][Image1]  


## 无状态和有状态 Service Fabric 微服务
借助 Service Fabric，可生成由微服务组成的应用程序。无状态微服务（例如协议网关和 Web 代理）不会维护除请求及其服务响应之外的可变状态。Azure 云服务辅助角色是无状态服务的一个示例。有状态微服务（例如用户帐户、数据库、设备、购物车和队列）会维护请求及其响应之外的可变、权威状态。当今的 Internet 规模应用程序包含无状态和有状态微服务的组合。

为何要将有状态和无状态的微服务一同使用？ 有以下两个主要原因：

* 在同一台计算机上禁用代码和数据可以生成高吞吐量、低延迟的容错联机事务处理 (OLTP) 服务。一些示例包括互动商店、搜索、物联网 (IoT) 系统、交易系统、信用卡处理和欺诈检测系统，以及个人档案管理。
* 可以简化应用程序设计。使用有状态微服务时，无需其他队列和缓存，而这在传统上是处理纯无状态应用程序的可用性和延迟要求所必需的。有状态服务原本就具有高可用性和低延迟，减少了在应用程序中要作为一个整体进行管理的移动部件的数量。

有关使用 Service Fabric 的应用程序模式的详细信息，请阅读适用于服务的[应用程序方案](/documentation/articles/service-fabric-application-scenarios/)并[选择编程模型框架](/documentation/articles/service-fabric-choose-framework/)。

也可观看下面的 Microsoft 虚拟大学视频，大致了解无状态服务和有状态服务：
<center><a target="_blank" href="https://mva.microsoft.com/training-courses/building-microservices-applications-on-azure-service-fabric-16747?l=HhD9566yC_4106218965"> 
<img src="./media/service-fabric-overview/ReliableServicesVid.png" WIDTH="360" HEIGHT="244"> 
</a></center>

## 应用程序生命周期管理
Service Fabric 为云应用程序的完整应用程序生命周期管理提供支持。此生命周期包括开发到部署、日常管理，以及维护到最终解除授权。

借助 Service Fabric 应用程序生命周期管理功能，应用程序管理员和 IT 操作人员能够使用低接触的简单工作流预配、部署、修补和监视应用程序。这些内置的工作流极大地减少了 IT 操作人员保持应用程序持续可用的负担。

大多数应用程序包含无状态和有状态微服务的组合，以及一起部署的其他可执行文件/运行时。通过在应用程序和已打包微服务上采用强类型，使用 Service Fabric 能够部署多个应用程序实例。每个实例将单独进行管理和升级。重要的是，Service Fabric 能够部署*任何*可执行文件或运行时并使其可靠。例如，Service Fabric 可部署 .NET、ASP.NET Core、Node.js、Java 虚拟机、脚本、Angular 或组成应用程序的任何其他对象。

有关应用程序生命周期管理的详细信息，请阅读[应用程序生命周期](/documentation/articles/service-fabric-application-lifecycle/)。有关如何部署任意代码的详细信息，请参阅[部署来宾可执行文件](/documentation/articles/service-fabric-deploy-existing-app/)

也可观看下面的 Microsoft 虚拟大学视频，大致了解应用生命周期管理：
<center><a target="_blank" href="https://mva.microsoft.com/training-courses/building-microservices-applications-on-azure-service-fabric-16747?l=My3Ka56yC_6106218965"> 
<img src="./media/service-fabric-overview/AppLifecycleVid.png" WIDTH="360" HEIGHT="244"> 
</a></center>

## 关键功能
通过使用 Service Fabric，你可以：

* 开发可自我修复的高度可扩展应用程序。
* 使用 Service Fabric 编程模型开发由微服务组成的应用程序。或者，可以直接托管选择的来宾可执行文件和其他应用程序框架，例如 ASP.NET Core 或 Node.js。
* 开发高度可靠的无状态和有状态微服务。
* 部署并协调包括整个群集中的 Windows 容器。这些容器可以包含来宾可执行文件，或可靠的无状态和有状态微服务。在任一情况下，都可以获得容器端口到主机端口的映射、容器可发现性和自动故障转移。
* 通过使用有状态微服务代替缓存和队列来简化你的应用程序的设计。
* 无需更改代码，即可部署到 Azure，或部署到运行 Windows 或 Linux 的本地数据中心。编写一次，即可随处部署到任何 Service Fabric 群集。
* 使用“计算机上的数据中心”方法进行开发。本地开发环境使用 Azure 数据中心运行的相同代码。
* 在数秒内部署应用程序。
* 以比虚拟机更高的密度部署应用程序，每台计算机部署成千上万的应用程序。
* 同时部署同一应用程序的不同版本，并单独升级每个应用程序。
* 管理有状态应用程序的生命周期，且无需任何停机时间，包括中断升级和非中断升级。
* 使用 .NET API、Java (Linux)、PowerShell、Azure 命令行接口 (Linux) 或 REST 接口来管理应用程序。
* 在应用程序内独立地升级和修补微服务。
* 监视并诊断应用程序的运行状况，并设置策略以执行自动修复。
* 增加或减少群集中的节点数量，扩大或缩小每个节点的大小。缩放节点时，应用程序会自动缩放，并根据可用资源进行分布。
* 观看可自我修复的资源平衡器如何在群集间协调重新分发应用程序。Service Fabric 可从故障中恢复，并基于可用资源优化负载分布。
* 使用错误分析服务在服务上进行混沌测试，找出问题和故障，然后再在生产环境中运行。

<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->

## 后续步骤
* 更多相关信息：
	* [为什么通过微服务的方法构建应用程序？](/documentation/articles/service-fabric-overview-microservices/)
	* [术语概述](/documentation/articles/service-fabric-technical-overview/)
* 设置 Service Fabric [开发环境](/documentation/articles/service-fabric-get-started/)
* 为服务[选择编程模型框架](/documentation/articles/service-fabric-choose-framework/)
* 了解 [Service Fabric 支持选项](/documentation/articles/service-fabric-support/)

[Image1]: ./media/service-fabric-overview/Service-Fabric-Overview.png

<!---HONumber=Mooncake_0213_2017-->
<!--Update_Description: add three MVA videos-->