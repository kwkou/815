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
   ms.date="08/03/2016"
   wacn.date="08/29/2016"/>

# Service Fabric 概述
Service Fabric 是一种分布式系统平台，可让你轻松打包、部署和管理可缩放、可靠的微服务。Service Fabric 还解决了开发和管理云应用程序中的重大难题。开发人员和管理员不仅可以避免解决复杂的基础结构问题，而且可以专注于实现可扩展、可靠且易于管理的所需的任务关键型工作负荷。Service Fabric 代表用于生成和管理这些企业级的一级云规模应用程序的下一代中间件平台。

## 由微服务组成的应用程序
利用 Service Fabric，您可以生成和管理可缩放且可靠的应用程序，该应用程序由在计算机的共享池（称为群集）上以非常高的密度运行的微服务组成。它可以提供复杂运行时，用于构建分布式、可扩展的无状态和有状态微服务。它还提供了全面的应用程序管理功能，用于设置、部署、监视、升级/修补和删除部署的应用程序。

为什么微服务方法很重要？ 有以下两个主要原因：

1. 它们让你能够根据应用程序的需求来扩展应用程序的不同部分。

2. 开发团队可以更加灵活地推出更改，从而更快速和更频繁地向客户提供功能。

Service Fabric 为当今很多 Microsof 服务提供支持，包括 Azure SQL 数据库、Azure DocumentDB、Cortana、Power BI、Microsoft Intune、Azure 事件中心、Azure IoT、Skype for Business 和许多核心 Azure 服务。

Service Fabric 适用于创建“基于云开发”的服务，该服务可以根据需要先为小型服务，然后成长为包含数百或数千台计算机的大规模服务。

当今的 Internet 规模的服务是使用微服务构建而成的。微服务的例子包括协议网关、用户配置文件、购物车，清单处理、排队和缓存等。Service Fabric 是微服务平台，为每个微服务提供可以是无状态或有状态的唯一名称。

Service Fabric 为由这些微服务组成的应用程序提供全面的运行时和生命周期管理功能。它在 Service Fabric 群集间部署和激活的容器内部托管微服务。从 VM 移动到容器可能使密度出现数量级增长。同样，从容器移动到微服务也可能出现另一个密度数量级。例如，单个 Azure SQL 数据库群集包含数百台计算机，这些计算机运行数以万计的容器，这些容器总共托管数十万个数据库。每个数据库都是一个 Service Fabric 有状态微服务。这同样适用于前文提及的其他服务，正因如此，术语“hyperscale”可用于描述 Service Fabric 功能。如果容器为你提供了高密度，那么微服务可为你提供超大规模。

有关微服务方法的详细信息，请阅读[微服务方法为什么可以生成应用程序？](/documentation/articles/service-fabric-overview-microservices/)

## 随地创建 Service Fabric 群集
您可以在许多环境中（例如在 Azure 或本地、Windows Server 或 Linux 中）创建 Service Fabric 群集。此外，SDK 中的开发环境与生产环境完全相同，都不涉及模拟器。换而言之，如果 SDK 在本地开发群集上运行，则会部署到其他环境中的相同群集。

有关在本地创建群集的详细信息，请阅读[在 Windows Server 或 Linux 上创建群集](/documentation/articles/service-fabric-deploy-anywhere/)。

![Service Fabric 平台][Image1]

## 无状态和有状态 Service Fabric 微服务

Service Fabric 允许你构建包含微服务的应用程序。无状态微服务（例如网关、Web 代理等）不维护除任何指定请求及其来自服务的响应之外任何可变状态。Azure 云服务辅助角色是无状态服务的一个示例。有状态微服务（用户帐户、数据库、设备、购物车、队列等）维护除请求及其响应之外的可变、授权状态。当今的 Internet 规模应用程序包含无状态和有状态微服务的组合。

为何要将有状态和无状态的微服务一同使用？ 有以下两个主要原因：

1. 在同一台计算机上禁用代码和数据可以生成高吞吐量、低延迟、容错联机事务处理 (OLTP) 服务。一些示例包括互动商店、搜索、物联网 (IoT) 系统、交易系统、信用卡处理和欺诈检测系统，以及个人档案管理。

2. 应用程序设计简化。有状态微服务删除了传统上需要用来处理纯无状态应用程序的可用性和延迟需求的附加队列和缓存。有状态服务原本就具有高可用性和低延迟，减少了在应用程序中要作为一个整体进行管理的移动部件的数量。

有关使用 Service Fabric 的应用程序模式的详细信息，请阅读适用于你服务的 [Application scenarios](/documentation/articles/service-fabric-application-scenarios/)（应用程序方案）和 [Choosing a programming model framework](/documentation/articles/service-fabric-choose-framework/)（选择编程模型框架）。

## 应用程序生命周期管理
Service Fabric 为云应用程序的整个应用程序生命周期管理 (ALM) 提供一流的支持：从开发到部署、到日常管理和维护，再到最终解除授权。

利用 Service Fabric ALM 功能，应用程序管理员/IT 操作人员能够使用低接触的简单工作流配置、部署、修补和监视应用程序。这些内置的工作流极大地减少了 IT 操作人员保持应用程序持续可用的负担。

大多数应用程序包含无状态和有状态微服务的组合，以及一起部署的其他可执行文件/运行时。通过在应用程序和已打包微服务上采用强类型，使用 Service Fabric 能够部署多个应用程序实例。每个实例将单独进行管理和升级。重要的是，Service Fabric 能够部署*任何*可执行文件或运行时并使其可靠。例如，Service Fabric 可以用于部署 ASP.NET Core 1、Node.js、Java VM、脚本或组成您的应用程序的任何其他内容。

有关应用程序生命周期管理的详细信息，请阅读 [Application lifecycle](/documentation/articles/service-fabric-application-lifecycle/)（应用程序生命周期）；有关部署任何代码的详细信息，请参阅 [Deploy a guest executable](/documentation/articles/service-fabric-deploy-existing-app/)（部署来宾可执行文件）

## 关键功能
通过使用 Service Fabric，你可以：

- 开发可自我修复的高度可扩展应用程序。

- 使用 Service Fabric 编程模型部署由微服务组成的应用程序。或者，只需托管选择的来宾可执行文件和其他应用程序框架，如 ASP.NET Core 1 或 Node.js。

- 开发高度可靠的无状态和有状态微服务。

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

- 观看可自我修复的资源平衡器如何在群集间协调重新分发应用程序。Service Fabric 可从故障中恢复，并基于可用资源优化负载分布。

<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->
## 后续步骤

* 更多相关信息：
	* [为什么通过微服务的方法构建应用程序？](/documentation/articles/service-fabric-overview-microservices/)
	* [术语概述](/documentation/articles/service-fabric-technical-overview/)
* 设置 Service Fabric [开发环境](/documentation/articles/service-fabric-get-started/)
* 为服务[选择编程模型框架](/documentation/articles/service-fabric-choose-framework/)

[Image1]: ./media/service-fabric-overview/Service-Fabric-Overview.png

<!---HONumber=Mooncake_0822_2016-->