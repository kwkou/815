<properties 
	pageTitle="针对 Web 应用和移动应用的 Azure 应用服务 | Azure" 
	description="了解如何借助 Azure 应用服务开发、部署和管理 Web 应用和移动应用。" 
	keywords="应用服务, azure 应用服务, 应用服务成本, 缩放, 可缩放, 应用部署, azure 应用部署, paas, 平台即服务"
	services="app-service" 
	documentationCenter="" 
	authors="omarkmsft" 
	manager="dwrede" 
	editor="jimbe"/>

<tags
	ms.service="app-service"
	ms.date="05/25/2016"
	wacn.date="09/26/2016"/>

# 什么是 Azure 应用服务？

*应用服务* 是 Azure 的[平台即服务](https://zh.wikipedia.org/wiki/平台即服务) (PaaS) 产品。为任何平台或设备创建 Web 应用和移动应用。将应用与 SaaS 解决方案集成、与本地应用程序进行连接，以及实现业务流程的自动化。Azure 在完全托管的虚拟机 (VM) 上运行应用程序，由用户选择共享的 VM 资源或专用 VM。

应用服务所包括的 Web 功能和移动功能是以前作为 Azure 网站和 Azure 移动服务单独交付的。它还包括各种新功能，可以实现业务流程的自动化，并可托管云 API。应用服务是一项集成服务，允许用户将各种组件（网站、移动应用后端、RESTful API 和业务流程）组合到单个解决方案中。

## 为何使用应用服务？

下面是应用服务的某些主要特性和功能：

- **多种语言和框架** - 应用服务为 ASP.NET、Node.js、Java、PHP 和 Python 提供一流支持。也可以在应用服务 VM 上运行 [Windows PowerShell 和其他脚本或可执行文件](/documentation/articles/web-sites-create-web-jobs/)。

- **开发运营优化** - 使用 GitHub 设置[持续集成和部署](/documentation/articles/app-service-continuous-deployment/)。通过[测试和过渡环境](/documentation/articles/web-sites-staged-publishing/)提升更新。执行 [A/B 测试](/documentation/articles/app-service-web-test-in-production-get-start/)。使用 [Azure PowerShell](/documentation/articles/powershell-install-configure/) 或[跨平台命令行接口 (CLI)](/documentation/articles/xplat-cli-install/) 在应用服务中管理应用。
 
- **具有高可用性的全局缩放** - 以手动或自动方式[增加](/documentation/articles/web-sites-scale/)或[扩大](/documentation/articles/insights-how-to-scale/)。在 Azure 中国数据中心基础结构中的任意位置托管应用，并且应用服务 [SLA](/support/sla/app-service/) 承诺高可用性。

- **到 SaaS 平台和本地数据的连接** - 从适用于企业系统（例如 SAP、Siebel 和 Oracle）的 50 多个连接器、SaaS 服务（例如 Salesforce 和 Office 365）以及 Internet 服务中进行选择。使用[Azure 虚拟网络](/documentation/articles/app-service-vnet-integration-powershell/)访问本地数据。

- **安全性和合规性** - 应用服务符合 [ISO、SOC 和 PCI](https://www.trustcenter.cn/) 的要求。

- **Visual Studio 集成** - Visual Studio 中的专用工具可简化创建、部署和调试工作。

## 应用服务中的应用类型

应用服务提供多种 *应用类型* ，每种类型负责托管特定类型的工作负荷：

- [**Web 应用**](/documentation/articles/app-service-web-overview/) - 负责托管网站和 Web 应用程序。

- [**移动应用**](/documentation/articles/app-service-mobile-value-prop/) - 负责托管移动应用后端。
   
- [**API 应用**](/documentation/articles/app-service-api-apps-why-best-platform/) - 负责托管云 API。

此处的“应用”一词是指专用于运行工作负荷的托管资源。以“Web 应用”为例，用户可能习惯于将 Web 应用视为计算资源和应用程序代码，二者共同提供浏览器的功能。但在应用服务中， *Web 应用* 是 Azure 提供的用于托管应用程序代码的计算资源。如果应用程序由 Web 前端和 RESTful API 后端组成，则既可将二者都部署到 Web 应用，也可将前端代码部署到 Web 应用，将后端代码部署到 API 应用。应用程序可能由多个不同类型的应用服务应用组成。

## App Service 计划

[应用服务计划](/documentation/articles/azure-web-sites-web-hosting-plans-in-depth-overview/)指定运行应用的计算资源的类型。如果流量负载预计较轻，则可使用共享虚拟机 (VM)。负载较高时，可以从不同大小的专用 VM 中进行选择。多个应用服务应用可共享一个相同的计划，并可随计划一起缩放。

## 定价

有关应用服务成本的信息，请参阅[应用服务定价](/pricing/details/app-service/)。

## 应用服务入门

开具一个 [Azure 试用帐户](/pricing/1rmb-trial/)，试用下述某个入门教程：

* [教程：创建 Web 应用](/documentation/articles/app-service-web-get-started/)
* [教程：创建移动应用](/documentation/articles/app-service-mobile-android-get-started/)
* [教程：创建 API 应用](/documentation/articles/app-service-api-dotnet-get-started/)

<!---HONumber=Mooncake_0919_2016-->