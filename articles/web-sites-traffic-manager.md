<properties 
	pageTitle="使用 Azure 流量管理器控制 Azure WEB 应用流量" 
	description="因为 Azure 流量管理器与 Azure WEB 应用相关，本文提供了有关 Azure 流量管理器的摘要信息。" 
	services="app-service\web" 
	documentationCenter="" 
	authors="cephalin" 
	writer="cephalin" 
	manager="wpickett" 
	editor="mollybos"/>

<tags
	ms.service="web-sites"
	ms.date="09/29/2015"
	wacn.date="01/21/2016"/>

# 使用 Azure 流量管理器控制 Azure WEB 应用流量

> [AZURE.NOTE]因为 Windows Azure 流量管理器与 Azure WEB 应用相关，本文提供了有关 Windows Azure 流量管理器的摘要信息。有关 Azure Traffic Manager 本身的更多信息，请访问本文结尾处的链接。

## 介绍
你可以使用 Azure 流量管理器控制如何将来自 Web 客户端的请求分发到 Azure 中。将 WEB 应用终结点添加到 Azure 流量管理器配置文件后，Azure 流量管理器会跟踪 WEB 应用的状态（正在运行、已停止或已删除），这样它就能确定那些终结点中有哪些应该接收流量。

## 负载平衡方法
Azure 流量管理器使用三种不同的负载平衡方法。以下列表描述了与 Azure WEB 应用相关的这些方法。

* **故障转移**：如果你在不同区域有克隆的 WEB 应用，可使用此方法配置一个 WEB 应用来服务所有的 Web 客户端流量，并配置另一区域的另一 WEB 应用在前一个 WEB 应用不可用时服务该流量。 
	
* **轮循机制**：如果你在不同区域有克隆的 WEB 应用，可使用此方法在不同区域的 WEB 应用间均等地分布流量。
	
* **性能**：“性能”方法根据到客户端的最短往返行程时间来分布流量。“性能”方法可用于同一区域或不同区域中的 WEB 应用。

有关 Azure 流量管理器中负载平衡的详细信息，请参阅[关于流量管理器负载平衡方法](/documentation/articles/traffic-manager-load-balancing-methods)。

## WEB 应用和流量管理器配置文件 
若要通过配置来控制 WEB 应用流量，你需要在 Azure 流量管理器中创建一个使用前述三种负载平衡方法之一的配置文件，然后将要控制其流量的终结点（在此例中是 WEB 应用）添加到该配置文件。你的 WEB 应用状态（正在运行、已停止或已删除）会定期传送到该配置文件，这样，Azure 流量管理器就可以相应地确定流量指向。

在通过 Azure 使用 Azure Traffic Manager 时，请记住以下几点：

* 对于同一区域内的仅限 WEB 应用的部署， WEB 应用已经提供了故障转移和轮询机制功能（与 WEB 应用模式无关）

* 对于同一区域中连同另一 Azure 云服务一起使用 WEB 应用的部署，可将两种终结点类型合并以启用混合方案。

* 在一个配置文件中，你只能给每个区域指定一个 WEB 应用终结点。当你选择某个 WEB 应用作为一个区域的终结点时，该区域中的其余 WEB 应用对于该配置文件会变成不可被选择的状态。

* 你在 Azure 流量管理器配置文件中指定的 WEB 应用终结点将出现在配置文件中 WEB 应用“配置”页面的“域名”部分下，但不会在那里进行配置。

* 在将 WEB 应用添加到配置文件后，该 WEB 应用门户页面仪表板上的“ WEB 应用 URL”将显示该 WEB 应用的自定义域 URL（如果你已经设置好了一个）。否则，它将显示流量管理器配置文件 URL（例如，`contoso.trafficmgr.com`）。在 WEB 应用的“配置”页面的“域名”部分下将可以看到 WEB 应用的直接域名和流量管理器 URL。

* 你的自定义域名将正常工作，但除了将它们添加到你的 WEB 应用之外，你还必须配置 DNS 映射，使之指向流量管理器 URL。有关如何为 Azure WEB 应用设置自定义域的信息，请参阅[为 Azure WEB 应用配置自定义域名](/documentation/articles/web-sites-custom-domain-name)。

* 你只能将标准模式下的 WEB 应用添加到 Azure 流量管理器配置文件。

## 后续步骤

有关 Azure 流量管理器概念及技术方面的概述，请参阅[流量管理器概述](/documentation/articles/traffic-manager-overview)。

有关 Azure 流量管理器中负载平衡的详细信息，请参阅[关于流量管理器负载平衡方法](/documentation/articles/traffic-manager-load-balancing-methods)。

有关将流量管理器用于 WEB 应用的详细信息，请参阅博客文章[将 Azure 流量管理器用于 Azure WEB 应用](http://blogs.msdn.com/b/waws/archive/2014/03/18/using-windows-azure-traffic-manager-with-waws.aspx)和 [Azure 流量管理器现在可以与 Azure WEB 应用集成](http://azure.microsoft.com/blog/2014/03/27/azure-traffic-manager-can-now-integrate-with-azure-web-sites/)。
 

<!---HONumber=82-->