<properties
	pageTitle="Azure App Service、虚拟机、Service Fabric 和云服务的比较 | Azure"
	description="了解托管 Web 应用程序时如何在 Azure App Service、虚拟机、Service Fabric 和云服务之间做出选择。"
	services="app-service\web, virtual-machines, cloud-services"
	documentationCenter=""
	authors="tdykstra"
	manager="wpickett"
	editor="jimbe"/>

<tags
	ms.service="app-service-web"
	ms.workload="web"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="07/07/2016"
	wacn.date="11/14/2016"
	ms.author="tdykstra"/>

# Azure App Service、虚拟机、Service Fabric 和云服务的比较

## 概述

Azure 提供几种托管网站的方式：[Azure App Service][]、[虚拟机][]、[Service Fabric][] 和[云服务][]。本文可帮助你了解这几种方式，并针对你的 Web 应用程序做出正确的选择。

Azure App Service 是大多数 Web 应用的最佳选择。部署和管理都已集成到平台，站点可以快速缩放以应对高流量负载，而内置的负载均衡和流量管理器可以实现高可用性。[WebJobs][] 功能可将后台作业处理轻松添加到应用服务 Web 应用。

如果你要创建新应用或重新编写现有应用以使用微服务体系结构，Service Fabric 是不错的选择。在共享计算机池上运行的应用可以从小规模开始，根据需要扩展为包含成百上千个计算机的大规模。有状态服务可轻松地以一致且可靠的方式存储应用状态，而 Service Fabric 会自动为你管理服务分区、缩放和可用性。Service Fabric 也支持具有 Open Web Interface for .NET (OWIN) 和 ASP.NET Core 的 WebAPI。与应用服务相比，Service Fabric 也能更好地控制或直接访问基础结构。你可以远程登录服务器，或配置服务器启动任务。云服务的控制和易用程度类似于 Service Fabric，但它现在是旧版服务，因此建议将 Service Fabric 用于新的开发。

如果现有应用程序需要大幅修改才能在应用服务或 Service Fabric 中运行，可选择虚拟机来简化到云的迁移。但相比于 Azure App Service 和 Service Fabric，正确配置、保护和维护 VM 需要更多的时间和 IT 专业知识。如果你考虑采用 Azure 虚拟机，请确保将修补、更新和管理 VM 环境所需的持续性维护工作纳入考虑。Azure 虚拟机是服务架构 (IaaS)，而应用服务和 Service Fabric 是平台即服务 (Paas)。

## <a name="features"></a>功能比较

下表比较了应用服务、云服务、虚拟机和 Service Fabric 的功能，帮助用户做出最佳选择。若要了解每个选项的 SLA 的当前信息，请参阅 [Azure 服务级别协议](/support/legal/sla/)。

功能|应用服务（Web 应用）|云服务（web 角色）|虚拟机|Service Fabric|说明
---|---|---|---|---|---
接近即时的部署|X|||X|将应用程序或应用程序更新部署到云服务（或者创建 VM）至少需要几分钟；将应用程序部署到 Web 应用只需数秒钟。
向上扩展到更大的计算机且无需重新部署|X|||X|
Web 服务器实例共享内容和配置，这意味着进行扩展时无需重新部署或重新配置。|X|||X|
多个部署环境（生产和过渡）|X|X||X|Service Fabric 允许你为应用创建多个环境，或者同时为应用部署不同的版本。
自动操作系统更新管理|X|X|||将来的 Service Fabric 发行版将推出自动 OS 更新。
无缝平台切换（轻松地在 32 位和 64 位之间转移）|X|X|||
使用 GIT、FTP 部署代码|X||X||
使用 Web 部署来部署代码|X||X||云服务支持使用 Web 部署将更新部署到单个角色实例。但是，不能将其用于初始部署角色，并且如果将 Web 部署用于更新，则必须单独部署到角色的每个实例。需要提供多个实例，才能针对生产环境获得云服务 SLA 资格。
WebMatrix 支持|X||X||
访问 Service Bus、存储空间、SQL 数据库之类的服务|X|X|X|X|
托管多层体系结构的 web 或 web 服务层|X|X|X|X|
托管多层体系结构的中间层|X|X|X|X|应用服务 Web 应用可轻松托管 REST API 中间层，而 [WebJobs](/documentation/articles/websites-webjobs-resources/) 功能可托管后台处理作业。可在专用网站中运行 WebJobs，实现层的独立可扩展性。预览版 [API 应用](/documentation/articles/app-service-api-apps-why-best-platform/)功能提供更多托管 REST 服务的功能。
支持 ASP.NET、经典 ASP、Node.js、PHP、Python|X|X|X|X|Service Fabric 支持使用 [ASP.NET 5](/documentation/articles/service-fabric-add-a-web-frontend/) 创建 Web 前端，或者也可按[来宾可执行文件](/documentation/articles/service-fabric-deploy-existing-app/)的形式部署任何类型的应用程序（Node.js、Java 等）。
向外扩展到多个实例且无需重新部署|X|X|X|X|虚拟机可以扩大到多个实例，但必须编写这些虚拟机上运行的服务，来处理向外扩展。你需要配置负载均衡器，跨计算机路由请求；还需要创建地缘组，防止因维护或硬件故障导致同时重新启动所有实例。
支持 SSL|X|X|X|X|对于应用服务 Web 应用，只有基本和标准模式支持自定义域名的 SSL。若要了解如何结合使用 SSL 和 Web 应用，请参阅[为 Azure 网站配置 SSL 证书](/documentation/articles/web-sites-configure-ssl-certificate/)。
Visual Studio 集成|X|X|X|X|
远程调试|X|X|X||
使用 [Azure 虚拟网络](/home/features/networking/)进行网络隔离|X|X|X|X|另请参阅 [Azure 网站的虚拟网络集成](/documentation/articles/app-service-vnet-integration-powershell/)
支持 [Azure 流量管理器](/home/features/traffic-manager/)|X|X|X|X|
集成的端点监视|X|X|X||
对服务器的远程桌面访问||X|X|X|
安装任何自定义 MSI||X|X|X|Service Fabric 可按[来宾可执行文件](/documentation/articles/service-fabric-deploy-existing-app/)的形式托管任何可执行文件，还可在 VM 上安装任意应用。
能够定义/执行启动任务||X|X|X|
可以侦听 ETW 事件||X|X|X|

## <a name="scenarios"></a>方案和建议

以下是一些常见的应用程序方案，其中每个方案都包含有关最适合的 Azure Web 托管选项的建议。

- [我需要具有后台处理的 Web 前端和数据库后端，运行与本地资产集成的业务应用程序。](#onprem)
- [我需要一种可靠的方式来托管公司网站，既可以进行良好地扩展也能实现全国覆盖。](#corp)
- [我具有在 Windows Server 2003 上运行的 IIS6 应用程序。](#iis6)
- [我是小型企业所有者，我需要使用一种成本较低的方式来托管站点，同时也要兼顾将来的业务增长。](#smallbusiness)
- [我是 Web 或图形设计师，我想为客户设计和构建网站。](#designer)
- [我要将带有 Web 前端的多层应用程序迁移到云中。](#multitier)
- [我的应用程序依赖高度自定义的 Windows 或 Linux 的环境，我想将其转移到云中。](#custom)
- [我的站点使用开放源代码软件，我希望在 Azure 中托管它。](#oss)
- [我有一个需要连接到公司网络的业务线应用程序。](#lob)
- [我想为移动客户端托管 REST API 或 Web 服务。](#mobile)


### <a id="onprem"></a> 我需要具有后台处理的 Web 前端和数据库后端，运行与本地资产集成的业务应用程序。

Azure App Service 是针对复杂业务应用程序的理想解决方案。你可以通过该网站开发应用，这些应用可以在负载均衡平台上自动缩放、使用 Active Directory 进行保护并连接到本地资源。使用该网站，可以通过世界级门户和 API 轻松地管理这些应用，并且还能通过应用洞察工具深入了解客户使用这些应用的情况。[Webjobs][] 功能可将后台进程和任务作为 Web 层的一部分运行，而 VNET 功能则可轻松地重新连接到本地资源。Azure App Service 针对 Web 应用提供三个 9 的 SLA，且能够：

* 在自愈性自动修补云平台上安全可靠地运行应用程序。
* 跨数据中心的全国网络进行自动缩放。
* 备份和还原，以进行灾难恢复。
* 遵守 ISO、SOC2 和 PCI 的要求。
* 与 Azure Active Directory 集成

### <a id="corp"></a> 我需要一种可靠的方式来托管公司网站，既可以进行良好地扩展也能实现全国覆盖。

Azure App Service 是托管公司网站的理想解决方案。通过 Web 应用，你可以轻松快速地缩放站点，满足整个数据中心全国网络的需求。它涵盖了本地范围，提供了容错和智能流量管理功能。所有内容均位于提供世界级管理工具的平台上，让你可以快速轻松地更深入了解站点运行状况和站点流量。Azure App Service 针对 Web 应用提供三个 9 的 SLA，且能够：

* 在自愈性自动修补云平台上安全可靠地运行网站。
* 跨数据中心的全国网络进行自动缩放。
* 备份和还原，以进行灾难恢复。
* 使用集成工具管理日志和流量。
* 遵守 ISO、SOC2 和 PCI 的要求。
* 与 Azure Active Directory 集成

### <a id="iis6"></a> 我具有在 Windows Server 2003 上运行的 IIS6 应用程序。

Azure App Service 可轻松避免在迁移较旧的 IIS6 应用程序时带来的基础结构成本。因为与 Visual Studio 和常见的 CMS 工具集成，所以能够更轻松地将 IIS6 应用程序直接部署到云中。部署后，Azure 门户可以提供强大的管理工具，帮助你通过缩减规模管理成本，并根据需要扩展规模来满足业务要求。使用迁移工具可以：

* 轻松快速地将旧版 Windows Server 2003 Web 应用程序迁移到云中。
* 选择在本地保留附加的 SQL 数据库，以创建混合应用程序。
* 自动转移 SQL 数据库与旧的应用程序。

### <a id="smallbusiness"></a>我是小型企业所有者，我需要使用一种成本较低的方式来托管站点，同时也要兼顾将来的业务增长。

Azure App Service 是适合该情况的强大解决方案，原因是其可免费使用且随后可按需添加更多功能。每个免费的 Web 应用都附带 Azure 提供的域 ( *your\_company* .chinacloudsites.cn)，并且平台中包含集成的部署和管理工具。还有许多其他服务和扩展选项，让站点可以随着用户需求的增加而发展。通过 Azure App Service，可以：

- 从免费层开始，然后根据需要向上扩展。
- 设置流行的 Web 应用程序，如 WordPress。
- 根据需要向你的应用程序添加其他 Azure 服务和功能。
- 使用 HTTPS 保护 Web 应用。

### <a id="designer"></a> 我是 Web 或图形设计师，我想为客户设计和构建网站

对 Web 开发人员和设计师而言，Azure App Service 可与各种框架和工具轻松集成（包括 Git 和 FTP 的部署支持），并可与 Visual Studio 和 SQL 数据库等工具和服务紧密集成。通过应用服务，可以：

- 将命令行工具用于自动化任务。
- 使用流行的语言，如 [.Net][dotnet]、[PHP][]、[Node.js][nodejs] 和 [Python][]。
- 选择三个不同的扩展级别，向上扩展到超高容量。
- 与其他 Azure 服务（例如 [SQL 数据库][sqldatabase]、[服务总线][servicebus]和[存储空间][]）集成。
- 与工具集成，例如 Visual Studio、Git、WebMatrix、WebDeploy 和 FTP。

### <a id="multitier"></a>我要将带有 Web 前端的多层应用程序迁移到云中

如果运行多层应用程序，如连接到数据库的 Web 服务器，Azure App Service 则是一个不错的选择，它可以与 Azure SQL 数据库紧密集成。你还可以使用 WebJobs 功能运行后端进程。

如果你需要加强对服务器环境的控制，例如想要远程登录服务器或配置服务器启动任务，可以为一个或多个层选择 Service Fabric。

如果你想要使用自己的计算机映像，或者想要运行不能在 Service Fabric 中配置的服务器软件或服务，可以为一个或多个层选择虚拟机。

### <a id="custom"></a>我的应用程序依赖高度自定义的 Windows 或 Linux 的环境，我想将其转移到云中。

如果你的应用程序需要对软件和操作系统进行复杂的安装或配置，虚拟机可能是最佳解决方案。通过虚拟机，你可以：

- 使用虚拟机库从某个操作系统（如 Windows 或 Linux）开始着手，然后针对你的应用程序要求对其进行定制。
- 创建并上传现有本地服务器的自定义映像，在 Azure 中的虚拟机上运行。

### <a id="oss"></a>我的站点使用开放源代码软件，我希望在 Azure 中托管它

如果应用服务支持开放源框架，则会自动配置应用程序所需的语言和框架。通过应用服务，可以：

- 使用多种流行的开放源代码语言，如 [.NET][dotnet]、[PHP][]、[Node.js][nodejs] 和 [Python][]。
- 安装 WordPress、Drupal、Umbraco、DNN 和许多其他第三方 Web 应用程序。
- 迁移现有应用程序。

如果应用服务不支持开源框架，则可在其他某个 Azure Web 托管选项上运行该框架。使用虚拟机，可以在计算机映像（基于 Windows 或 Linux）上安装和配置软件。

### <a id="lob"></a>我有一个需要连接到公司网络的业务线应用程序

如果你想要创建业务线应用程序，你的网站可能需要直接访问公司网络上的服务或数据。可在应用服务、Service Fabric 和虚拟机上使用 [Azure 虚拟网络服务](/home/features/networking/)来实现此目的。可在应用服务上使用 [VNET 集成功能](/documentation/articles/app-service-vnet-integration-powershell/)，该功能使 Azure 应用程序能够像在公司网络上一样运行。

### <a id="mobile"></a>我想为移动客户端托管 REST API 或 web 服务

利用基于 HTTP 的 Web 服务，你可以支持各种客户端，包括移动客户端。如 ASP.NET Web API 的框架与 Visual Studio 集成，能够更加轻松地创建和使用 REST 服务。这些服务来自 web 端点，因此可使用 Azure 上的任何 web 托管技巧支持此方案。但应用服务托管 REST API 的理想选择。通过应用服务，可以：

- 快速创建[移动应用](/documentation/articles/app-service-mobile-value-prop/)或 [API 应用](/documentation/articles/app-service-api-apps-why-best-platform/)，以便在 Azure 全国分布的某个数据中心托管 HTTP Web 服务。
- 迁移现有服务或创建新的服务。
- 实现 SLA 的单个实例可用性，或者将可用性扩展到多台专用计算机。
- 使用已发布的站点将 REST API 提供到任何 HTTP 客户端，包括移动客户端。

## <a id="nextsteps"></a>后续步骤

若要深入了解 3 个 Web 托管选项，请参阅 [Azure 简介](/documentation/articles/fundamentals-introduction-to-azure/)。

若要开始使用为应用程序选择的选项，请参阅以下资源：

* [Azure App Service](/documentation/services/app-service/)
* [Azure 云服务](/documentation/services/cloud-services/)
* [Azure 虚拟机](/documentation/services/virtual-machines/)
* [Service Fabric](/documentation/services/service-fabric/)

<!-- URL List -->

[Azure App Service]: /home/features/app-service/
[云服务]: /documentation/services/cloud-services/
[虚拟机]: /documentation/services/virtual-machines/
[Service Fabric]: /home/features/service-fabric
[ClearDB]: http://www.cleardb.com/
[WebJobs]: /documentation/articles/websites-webjobs-resources/
[Configuring an SSL certificate for an Azure Website]: /documentation/articles/web-sites-configure-ssl-certificate/
[dotnet]: /develop/net/
[nodejs]: /develop/nodejs/
[PHP]: /develop/php/
[Python]: /develop/python/
[servicebus]: /documentation/services/service-bus/
[sqldatabase]: /documentation/services/sql-databases/
[存储空间]: /documentation/services/storage/

<!-- IMG List -->

[ChoicesDiagram]: ./media/choose-web-site-cloud-service-vm/Websites_CloudServices_VMs_3.png

<!---HONumber=Mooncake_0919_2016-->