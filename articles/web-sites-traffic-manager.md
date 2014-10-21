<properties linkid="web-sites-traffic-manager" urlDisplayName="Controlling Windows Azure Web Sites Traffic with Azure Traffic Manager" pageTitle="Controlling Azure Web Sites Traffic with Azure Traffic Manager" metaKeywords="Azure Web Sites, Traffic Manager, request routing, round robin, failover, performance" description="This article provides summary information for  Azure Traffic Manager as it relates to Azure Web Sites." metaCanonical="" services="web-sites" documentationCenter="" title="Controlling Azure Web Sites Traffic with Azure Traffic Manager" authors="timamm"  solutions="" writer="timamm" manager="paulettm" editor="mollybos"  />

# 使用 Azure 流量管理器控制 Azure 网站流量

> [WACOM.NOTE] 本文对 Microsoft Azure 流量管理器作简要介绍，因为它与 Azure 网站有关。有关 Azure 流量管理器本身的详细信息，请访问本文结尾处的链接。

## 介绍

你可以使用 Azure 流量管理器来控制如何将来自 Web 客户端的请求分发到 Azure 网站。将 Azure 网站终结点添加到 Azure 流量管理器配置文件时，Azure 流量管理器会跟踪你网站的状态（正在运行、已停止或已删除），这样它就能确定那些终结点中有哪些应该接收流量。

## 负载平衡方法

Azure 流量管理器使用三种不同的负载平衡方法。下面的列表中说明了这些方法，它们涉及 Azure 网站。

-   **故障转移**：如果你在不同区域有克隆的网站，可使用此方法配置一个网站来服务所有的 Web 客户端流量，并配置另一区域的另一网站在前一个网站不可用时服务该流量。

-   **循环**：如果你在不同区域有克隆的网站，可使用此方法在不同区域的网站间均等地分布流量。

-   **性能**：“性能”方法根据到客户端的最短往返行程时间来分布流量。“性能”方法可用于同一区域或不同区域的网站。

有关 Azure 流量管理器中负载平衡的详细信息，请参阅[关于流量管理器负载平衡方法][关于流量管理器负载平衡方法]。

## Azure 网站和流量管理器配置文件

若要通过配置来控制网站流量，你需要在 Azure 流量管理器中创建一个使用前述三种负载平衡方法之一的配置文件，然后将要控制其流量的终结点（在此例中是网站）添加到该配置文件。你的网站状态（正在运行、已停止或已删除）会定期传送到该配置文件，这样，Azure 流量管理器就可以相应地确定流量指向。

在通过 Azure 使用 Azure 流量管理器时，请记住以下几点：

-   对于同一区域内仅限网站的部署，Azure 网站已经提供了故障转移和循环功能，与网站模式无关。

-   对于同一区域中连同另一 Azure 云服务一起使用 Azure 网站的部署，可将两种终结点类型合并以启用混合方案。

-   在一个配置文件中，你只能给每个区域指定一个网站终结点。当你选择某个网站作为一个区域的终结点时，该区域中的其余网站对于该配置文件会变成不可被选择的状态。

-   你在 Azure 流量管理器配置文件中指定的网站终结点将出现在配置文件中网站“配置”页面的“域名”部分下，但不会在那里进行配置。

-   在将网站添加到配置文件后，该网站门户页面仪表板上的“网站 URL”将显示该网站的自定义域 URL（如果你已经设置好了一个）。否则，它将显示流量管理器配置文件 URL（例如，`contoso.trafficmgr.com`). 在网站的“配置”页面的“域名”部分下将可以看到网站的直接域名和流量管理器 URL。

-   你的自定义域名将正常工作，但除了将它们添加到你的网站之外，你还必须配置 DNS 映射，使之指向流量管理器 URL。有关如何为 Azure 网站设置自定义域的信息，请参阅[为 Azure 网站配置自定义域名][为 Azure 网站配置自定义域名]。

-   你只能将“标准”模式下的网站添加到 Azure 流量管理器配置文件。

## 后续步骤

有关 Azure 流量管理器概念及技术方面的概述，请参阅[流量管理器概述][流量管理器概述]。

有关如何配置 Azure 流量管理器的信息，包括如何配置才能供 Azure 网站使用，请参阅[流量管理器配置任务][流量管理器配置任务]。

有关 Azure 流量管理器中负载平衡的详细信息，请参阅[关于流量管理器负载平衡方法][关于流量管理器负载平衡方法]。

  [关于流量管理器负载平衡方法]: http://msdn.microsoft.com/zh-cn/library/azure/dn339010.aspx
  [为 Azure 网站配置自定义域名]: http://azure.microsoft.com/zh-cn/documentation/articles/web-sites-custom-domain-name/
  [流量管理器概述]: http://msdn.microsoft.com/zh-cn/library/azure/hh744833.aspx
  [流量管理器配置任务]: http://msdn.microsoft.com/zh-cn/library/azure/hh744830.aspx
