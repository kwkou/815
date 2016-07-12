<properties 
   pageTitle="Service Fabric 概述 | Azure"
   description="Service Fabric 概览，其中应用程序由许多微服务组成，以提供扩展性和恢复能力。Service Fabric 是一个分布式系统平台，可用于构建面向云的可扩展、可靠且易管理的应用程序"
   services="service-fabric" 
   documentationCenter=".net" 
   authors="msfussell" 
   manager="timlt" 
   editor="masnider"/>

<tags
   ms.service="service-fabric"
   ms.date="03/24/2016"
   wacn.date="07/04/2016"/>

# Service Fabric 概述
Service Fabric 是一个分布式系统平台，可让你更加轻松地打包、部署和管理可扩展且可靠的微服务，并在开发和管理云应用程序的过程中处理重大难题。通过使用 Service Fabric，开发人员和管理员不仅可以避免解决复杂的基础结构问题，而且可以专注于实现可扩展、可靠且易于管理的所需的任务关键型工作负荷。Service Fabric 代表用于生成和管理这些企业级的一级云规模应用程序的下一代中间件平台。

## 由微服务组成的应用程序
利用 Service Fabric，你能够构建和管理可缩放且可靠的应用程序，该应用程序由在计算机的共享池（称为 Service Fabric 群集）上以非常高的密度运行的微服务组成。它可以提供复杂运行时，用于构建分布式、可扩展的无状态和有状态微服务。它还提供了全面的应用程序管理功能，用于设置、部署、监视、升级/修补和删除部署的应用程序。

为什么微服务方法很重要？ 有以下两个主要原因：

1. 它们让你能够根据应用程序的需求来扩展应用程序的不同部分

2. 开发团队能够更加灵活地推出更改，从而更快速和更频繁地向客户提供功能。

Service Fabric 为当今很多 Microsof 服务提供支持，包括 Azure SQL 数据库、Azure DocumentDB、Cortana、Power BI、Microsoft Intune、Azure 事件中心、Azure IoT、Skype for Business 和许多核心 Azure 服务，等等。

Service Fabric 适用于创建“基于云开发”的服务，该服务可以根据需要先为小型服务，然后成长为包含数百或数千台计算机的大规模服务。

当今的 Internet 规模服务是使用微服务构建而成的。微服务的例子包括协议网关、用户配置文件、购物车，清单处理、排队和缓存等。Service Fabric 是微服务平台，为每个微服务提供可以是无状态或有状态的唯一名称。

Service Fabric 为由这些微服务组成的应用程序提供全面的运行时和生命周期管理功能。它在 Service Fabric 群集间部署和激活的容器内部托管微服务。就像通过从 VM 移动到容器可能出现的密度的数量级增长一样，当从容器移动到微服务时，也可能出现类似的密度数量级增长。例如，在 Service Fabric 上构建的单个 Azure SQL 数据库群集包含数百台计算机，这些计算机运行数以万计的容器，这些容器总共托管数十万个数据库。（每个数据库都是一个 Service Fabric 有状态微服务。） 这同样适用于事件中心和上面提到的其他服务。这就是可使用“超大规模”一词来描述 Service Fabric 功能的原因。如果容器为你提供了高密度，那么微服务可为你提供超大规模。

有关微服务方法的详细信息，请阅读 [Why a microservices approach to building applications?（为什么通过微服务的方法生成应用程序？）](/documentation/articles/service-fabric-overview-microservices/)

## 随地创建 Service Fabric 群集
你可以在要部署应用程序的许多环境中创建 Service Fabric 群集。该环境可以是 Azure、本地、Windows Server 或 Linux。此外，SDK 中的开发环境与不包括模拟器的生产环境相同。换而言之，如果 SDK 在本地开发群集上运行，将部署到其他环境中的相同群集。

有关详细信息，请阅读 [Deploy anywhere on Windows Server or Linux with Service Fabric（使用 Service Fabric 在 Windows Server 和 Linux 上进行随地部署）](/documentation/articles/service-fabric-deploy-anywhere/)

![Service Fabric 平台][Image1]

## 无状态和有状态 Service Fabric 微服务

Service Fabric 允许你构建包含微服务的应用程序。无状态微服务（例如网关、Web 代理等）不维护除任何指定请求及其来自服务的响应之外任何可变状态。Azure 云服务辅助角色是无状态服务的一个示例。有状态微服务（用户帐户、数据库、设备、购物车、队列等）维护除请求及其响应之外的可变、授权状态。当今的 Internet 规模应用程序包含无状态和有状态微服务的组合。

为何要将有状态和无状态的微服务一同使用？ 有以下两个主要原因：

1. 通过在同一台计算机上维持代码和数据的封闭性来构建高吞吐量、低延迟、容错联机事务处理 (OLTP) 服务，例如交互式存储前端、搜索、物联网 (IoT) 系统、交易系统、信用卡处理和欺诈检测系统、个人记录管理等。

2. 简化应用程序设计，因为有状态微服务无需额外的队列和缓存。解决纯无状态应用程序的可用性和延迟需求时通常需要这些功能。由于有状态服务原本就具有高可用性和低延迟，这意味着在应用程序中要作为一个整体进行管理的移动部件更少。

有关使用 Service Fabric 的应用程序模式的详细信息，请阅读适用于你服务的 [Application scenarios（应用程序方案）](/documentation/articles/service-fabric-application-scenarios/) 和 [Choosing a programming model framework（选择编程模型框架）](/documentation/articles/service-fabric-choose-framework/)。

## 应用程序生命周期管理
Service Fabric 为云应用程序的整个应用程序生命周期管理 (ALM) 提供一流的支持：从开发到部署、到日常管理和维护，再到最终解除授权。

利用 Service Fabric ALM 功能，应用程序管理员/IT 操作人员能够使用低接触的简单工作流配置、部署、修补和监视应用程序。这些内置的工作流极大地减少了 IT 操作人员保持应用程序持续可用的负担。

大多数应用程序包含无状态和有状态微服务的组合，以及一起部署的其他可执行文件/运行时。通过在应用程序和已打包微服务上采用强类型，使用 Service Fabric 能够部署多个应用程序实例，并且每个都可以单独进行管理和升级。重要的是，Service Fabric 能够部署任何可执行文件或运行时并使其可靠。例如，它可以用于部署 ASP.NET Core 1、node.js、Java VM、脚本或组成你的应用程序的任何其他内容。

有关应用程序生命周期管理的详细信息，请阅读 [Application lifecycle（应用程序生命周期）](/documentation/articles/service-fabric-application-lifecycle/)；有关部署任何代码的详细信息，请参阅 [Deploy a guest executable（部署来宾可执行文件）](/documentation/articles/service-fabric-deploy-existing-app/)

## 关键功能
通过使用 Service Fabric，你可以：

- 开发可自我修复的高度可扩展应用程序。

- 使用 Service Fabric 编程模型开发由微服务组成的应用程序，或只是托管来宾可执行文件和所选的其他应用程序框架（例如 ASP.NET Core 1、node.js 等）。

- 开发无状态和有状态微服务并使这些服务高度可靠。

- 通过使用有状态微服务代替缓存和队列来简化你的应用程序的设计。

- 部署到 Azure 或部署到运行 Windows Server 或 Linux 的本地云，而无需改变任何代码。编写一次，然后随地部署到任何 Service Fabric 群集。

- 使用“计算机上的数据中心”方法进行开发。本地开发环境使用 Azure 数据中心运行的相同代码。

- 在数秒内部署应用程序。

- 以比虚拟机更高的密度部署应用程序，每台计算机部署成千上万的应用程序。

- 同时部署各种不同版本的相同应用程序，且每个版本都可以单独进行升级。

- 管理有状态应用程序的生命周期，且无需任何停机时间，包括中断升级和非中断升级。

- 使用 .NET API、Powershell 或 REST 接口管理应用程序。

- 在应用程序内独立地升级和修补微服务。

- 监视并诊断应用程序的运行状况，并设置策略以执行自动修复。

- 在根据可用的资源了解应用程序规模的情况下，轻松增加或减少 Service Fabric 群集。

- 监视自我修复的资源平衡器，在 Service Fabric 群集中安排应用程序的重新分布，以便从故障中恢复，并根据可用资源优化负载的分布。

<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->
## 后续步骤

* 更多相关信息：
	* [为什么通过微服务的方法构建应用程序？](/documentation/articles/service-fabric-overview-microservices/)
	* [术语概述](/documentation/articles/service-fabric-technical-overview/)
* 设置 Service Fabric [开发环境](/documentation/articles/service-fabric-get-started/)  
* 为服务[选择编程模型框架](/documentation/articles/service-fabric-choose-framework/)

[Image1]: ./media/service-fabric-overview/Service-Fabric-Overview.png

<!---HONumber=Mooncake_0425_2016-->