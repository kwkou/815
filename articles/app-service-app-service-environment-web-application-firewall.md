<properties 
	pageTitle="为 App Service 环境配置 Web 应用程序防火墙 (WAF)" 
	description="了解如何在 App Service 环境的前面配置 Web 应用程序防火墙。" 
	services="app-service\web" 
	documentationCenter="" 
	authors="naziml" 
	manager="wpickett" 
	editor=""/>

<tags 
	ms.service="app-service-web"  
	ms.date="07/02/2015" 
	wacn.date=""/>	

# 为 App Service 环境配置 Web 应用程序防火墙 (WAF)

## 概述 ##
[Azure 应用商店](http://azure.microsoft.com/marketplace/partners/barracudanetworks/waf-byol/)中的 Web 应用程序防火墙（如 [Barracuda WAF for Azure](https://www.barracuda.com/programs/azure)）可以通过检查入站 Web 流量来阻止 SQL 注入、跨站脚本、恶意上载和应用程序 DDoS 及其他攻击，从而帮助保护 Web 应用程序的安全。它还会检查后端 Web 服务器的响应，实现针对数据丢失预防 (DLP)。与隔离功能以及 App Service 环境提供的附加缩放相结合，它可以提供一个理想的环境，用于托管需要承受恶意请求和大量流量的业务关键型 Web 应用程序。

## 设置 ##
在本文中，我们将配置受多个 Barracuda WAF 负载平衡实例保护的 App Service 环境，只让来自 WAF 的流量到达该 App Service 环境，而且无法从外围网络访问该环境。在 Barracuda WAF 实例的前面，我们还部署了 Azure 流量管理器，用于在 Azure 数据中心和区域进行负载平衡。高级设置示意图如下所示。

![体系结构][Architecture]

## 配置 App Service 环境 ##
若要配置 App Service 环境，请参阅有关该主题的[文档](/documentation/articles/app-service-web-how-to-create-an-app-service-environment)。创建 App Service 环境后，可在此环境中创建 [Web Apps](/documentation/articles/app-service-web-overview)、[API Apps](/documentation/articles/app-service-api-apps-why-best-platform) 和 [Mobile Apps](/documentation/articles/app-service-mobile-value-prop-preview)，这些应用将受下一部分中配置的 WAF 保护。

## 配置 Barracuda WAF 云服务 ##
Barracuda 提供了有关在 Azure 中的虚拟机上部署其 WAF 的[详细文章](https://techlib.barracuda.com/WAF/AzureDeploy)。但是，由于我们想要冗余，但不想要造成单一故障点，因此可以在遵循这些说明时，将至少两个 WAF 实例 VM 部署到相同的云服务。

### 将终结点添加云服务 ###
云服务中有两个以上的 WAF VM 实例之后，即可使用 [Azure 门户](https://manage.windowsazure.cn)添加应用程序使用的 HTTP 和 HTTPS 终结点，如下图所示。

![配置终结点][ConfigureEndpoint]

如果你的应用程序使用其他终结点，则还要确保将这些终结点添加到此列表中。

### 通过管理门户配置 Barracuda WAF ###
Barracuda WAF 使用 TCP 端口 8000 通过其管理门户进行配置。由于我们有多个 WAF VM 实例，因此需要针对每个 VM 实例重复这些步骤。


> 注意：完成 WAF 配置后，从所有 WAF VM 中删除 TCP/8000 终结点，以保护 WAF 的安全。

若要配置 Barracuda WAF，请按下图所示添加管理终结点。

![添加管理终结点][AddManagementEndpoint]
 
使用浏览器浏览到云服务上的管理终结点。如果你的云服务名为 test.cloudapp.net，则浏览到 http://test.cloudapp.net:8000 即可访问此终结点。你应会看到与下面类似的登录页，在此页上，可以使用你在 WAF VM 设置阶段指定的凭据登录。

![管理登录页][ManagementLoginPage]

登录之后，应会出现下图中所示的仪表板，其中显示了有关 WAF 保护的基本统计信息。

![管理仪表板][ManagementDashboard]

单击“服务”选项卡可以根据 WAF 保护的服务配置 WAF。有关配置 Barracuda WAF 的详细信息，请查阅[相关文档](https://techlib.barracuda.com/waf/getstarted1)。在以下示例中，已配置处理 HTTP 和 HTTPS 流量的 Azure Web 应用。

![管理添加服务][ManagementAddServices]

> 注意：根据应用程序的配置方式与 App Service 环境中正在使用的功能，你需要转发非 80 和 443 TCP 端口的流量（例如，如果你为 Web 应用设置了 IP SSL）。有关 App Service 环境中使用的网络端口的列表，请参阅[控制入站流量文档](/documentation/articles/app-service-app-service-environment-control-inbound-traffic)中的“网络端口”部分。

## 配置 Microsoft Azure 流量管理器（可选） ##
如果多个区域中都有你的应用程序，则你可以使用 [Azure 流量管理器](/documentation/articles/traffic-manager)对这些区域进行负载平衡。为此，你可以使用流量管理器配置文件中 WAF 的云服务名称在 [Azure 门户](https://manage.windowsazure.cn)中添加终结点，如下图所示。

![流量管理器终结点][TrafficManagerEndpoint]

如果你的应用程序需要身份验证，请确保有某个资源不需要任何身份验证，使流量管理器能够 ping 出应用程序的可用性。可以在 [Azure 门户](https://manage.windowsazure.cn)的“配置”部分下配置 URL，如下所示。

![配置流量管理器][ConfigureTrafficManager]

若要将流量管理器 ping 从 WAF 转发给应用程序，需要在 Barracuda WAF 上设置网站转换，以将流量转发给应用程序，如以下示例中所示。

![网站转换][WebsiteTranslations]

## 使用网络资源组保护发往 App Service 环境的流量##
有关使用云服务的 VIP 地址只限制从 WAF 流入 App Service 环境的流量的详细信息，请遵循[控制入站流量文档](/documentation/articles/app-service-app-service-environment-control-inbound-traffic)。以下是针对 TCP 端口 80 运行此任务的示例 Powershell 命令。


    Get-AzureNetworkSecurityGroup -Name "RestrictWestUSAppAccess" | Set-AzureNetworkSecurityRule -Name "ALLOW HTTP Barracuda" -Type Inbound -Priority 201 -Action Allow -SourceAddressPrefix '191.0.0.1'  -SourcePortRange '*' -DestinationAddressPrefix '*' -DestinationPortRange '80' -Protocol TCP

将 SourceAddressPrefix 替换为 WAF 云服务的虚拟 IP 地址 (VIP)。

> 注意：删除并重新创建云服务之后，云服务的 VIP 会发生变化。在这样做之后，请务必更新网络资源组中的 IP 地址。
 
<!-- IMAGES -->
[Architecture]: ./media/app-service-app-service-environment-web-application-firewall/Architecture.png
[ConfigureEndpoint]: ./media/app-service-app-service-environment-web-application-firewall/ConfigureEndpoint.png
[AddManagementEndpoint]: ./media/app-service-app-service-environment-web-application-firewall/AddManagementEndpoint.png
[ManagementAddServices]: ./media/app-service-app-service-environment-web-application-firewall/ManagementAddServices.png
[ManagementDashboard]: ./media/app-service-app-service-environment-web-application-firewall/ManagementDashboard.png
[ManagementLoginPage]: ./media/app-service-app-service-environment-web-application-firewall/ManagementLoginPage.png
[TrafficManagerEndpoint]: ./media/app-service-app-service-environment-web-application-firewall/TrafficManagerEndpoint.png
[ConfigureTrafficManager]: ./media/app-service-app-service-environment-web-application-firewall/ConfigureTrafficManager.png
[WebsiteTranslations]: ./media/app-service-app-service-environment-web-application-firewall/WebsiteTranslations.png

<!---HONumber=66-->