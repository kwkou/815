<properties 
	pageTitle="App Service 环境简介" 
	description="了解有关可提供安全、加入 VNet 的专用缩放单位用于运行所有应用的 App Service 环境功能。" 
	services="app-service\web" 
	documentationCenter="" 
	authors="ccompy" 
	manager="wpickett" 
	editor=""/>

<tags 
	ms.service="app-service-web" 
	ms.date="04/14/2015" 
	editor=""/>

# App Service 环境简介

## 概述 ##
App Service 环境是 Azure App Service 的 [高级] [PremiumTier] 服务计划选项，可提供完全隔离和专用的环境，以便安全地运行所有应用。这包括具有扩展缩放选项的 [Web Apps][WebApps]、[Mobile Apps][MobileApps]、[API Apps][APIApps] 和 [Logic Apps][LogicApps]。

App Service 环境的计算资源专门用来运行你的应用。App Service 环境始终创建于区域虚拟网络中，你的应用有网络隔离的新选项。此外，App Service 环境支持其他缩放选项，最多有 50 个计算资源可用于运行你的应用。在 App Service 环境之外，用于托管应用的计算资源限制为 20 个。

## 虚拟网络支持 ##
可以在预先存在的区域虚拟网络或新的区域虚拟网络中创建 App Service 环境（[有关虚拟网络的详细信息][MoreInfoOnVirtualNetworks]）。由于 App Service 环境始终位于区域虚拟网络中，更准确地说，位于区域虚拟网络的子网中，因此你可以利用虚拟网络的安全功能来控制入站和出站网络通信。

可以使用[网络安全组][NetworkSecurityGroups]将入站网络通信限制为 App Service 环境所在的子网。这样，你便可以在上游设备和服务（例如 Ｗeb 应用程序防火墙和网络 SaaS 提供者）后面运行应用。

应用还经常需要访问公司资源，例如内部数据库和 Web 服务。常见的做法是让这些终结点仅可用于在 Azure 虚拟网络中流动的内部网络流量。一旦 App Service 环境加入到与内部服务相同的虚拟网络，在此环境中运行的应用即可访问这些内部服务，包括可通过[站点到站点][SiteToSite]访问的终结点。

## 专用计算资源 ##
App Service 环境中的所有计算资源都专属于单个订阅。App Service 环境由单个前端计算资源池，以及一到三个工作线程计算资源池组成。

前端池包含负责处理 SSL 终止以及 App Service 环境中应用请求的自动负载平衡的计算资源。

每个工作线程池包含分配给 [App Service 计划][AppServicePlan]的计算资源，而这些资源又包含一个或多个 Azure App Service 应用。由于 App Service 环境中可能有多达三个不同的工作线程池，因此你可以灵活地为每个工作线程池选择不同的计算资源。

例如，你可以针对主要用于开发或测试应用的 App Service 方案创建一个计算资源较不强大的工作线程池。第二个（甚至第三个）工作线程池可以使用比较强大的计算资源，以供 App Service 计划运行生产应用。

可将 App Service 环境配置为在单个工作线程池中最多包含 50 个计算资源。有关前端和工作线程池可用计算资源数量的详细信息，请参阅[如何配置 App Service 环境][HowToConfigureanAppServiceEnvironment]。

有关 App Service 环境中支持的可用计算资源大小的详细信息，请参阅 [App Service 定价][AppServicePricing] 页，并查看高级定价层中 App Service 环境可用的选项。


## 入门

若要开始使用 App Service 环境，请参阅[如何创建 App Service 环境][HowToCreateAnAppServiceEnvironment]

有关 Azure App Service 平台的详细信息，请参阅 [Azure App Service][AzureAppService]。

有关 App Service 环境网络体系结构的概述，请参阅[网络体系结构概述][NetworkArchitectureOverview]一文。

有关在 App Service 环境中使用 ExpressRoute 的详细信息，请参阅以下文章：[Express Route 和 App Service 环境][NetworkConfigDetailsForExpressRoute]。

[AZURE.INCLUDE [app-service-web-whats-changed](../includes/app-service-web-whats-changed.md)]

[AZURE.INCLUDE [app-service-web-try-app-service](../includes/app-service-web-try-app-service.md)]

<!-- LINKS -->
<!--[PremiumTier]: http://azure.microsoft.com/pricing/details/app-service/-->
[MoreInfoOnVirtualNetworks]: https://msdn.microsoft.com/zh-cn/library/azure/dn133803.aspx
[AppServicePlan]: /documentation/articles/azure-web-sites-web-hosting-plans-in-depth-overview
[Azure portal]: https://manage.windowsazure.cn
[HowToCreateAnAppServiceEnvironment]: /documentation/articles/app-service-web-how-to-create-an-app-service-environment
[AzureAppService]: /documentation/articles/app-service-value-prop-what-is
[WebApps]: /documentation/articles/app-service-web-overview
[MobileApps]: /documentation/articles/app-service-mobile-value-prop-preview
[APIApps]: /documentation/articles/app-service-api-apps-why-best-platform
[LogicApps]: /documentation/articles/app-service-logic-what-are-logic-apps
[NetworkSecurityGroups]: /documentation/articles/virtual-networks-nsg
[SiteToSite]: /documentation/articles/vpn-gateway-site-to-site-create
[HowToConfigureanAppServiceEnvironment]: /documentation/articles/app-service-web-configure-an-app-service-environment
[NetworkArchitectureOverview]: /documentation/articles/app-service-app-service-environment-network-architecture-overview
[NetworkConfigDetailsForExpressRoute]: /documentation/articles/app-service-app-service-environment-network-configuration-expressroute
<!--[AppServicePricing]: http://azure.microsoft.com/pricing/details/app-service/ -->

<!-- IMAGES -->

<!---HONumber=66-->