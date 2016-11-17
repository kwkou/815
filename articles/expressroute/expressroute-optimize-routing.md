<properties
   pageTitle="优化 ExpressRoute 路由 | Azure"
   description="本页详细介绍了在客户可以通过多个 ExpressRoute 线路在 Azure 与客户的公司网络之间进行连接时，如何优化路由。"
   documentationCenter="na"
   services="expressroute"
   authors="charwen"
   manager="carmonm"
   editor=""/>  

<tags
   ms.service="expressroute"
   ms.devlang="na"
   ms.topic="get-started-article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="10/10/2016"
   wacn.date="10/31/2016"
   ms.author="charwen"/>  


# 优化 ExpressRoute 路由
当你有多个 ExpressRoute 线路时，可以通过多个路径连接到 Azure。结果就是，你所采用的路由可能不是最理想的 - 也就是说，你的流量可能会经历较长的路径才能到达 Azure，而 Azure 的流量也可能会经历较长的路径才能到达你的网络。网络路径越长，延迟越严重。延迟对应用程序性能和用户体验有直接影响。本文将详述此问题，并说明如何使用标准路由技术来优化路由。

## 非最优路由情况 
让我们通过一个示例来详细了解路由问题。假设你在中国有两个办公室，一个在北京，一个在上海。你的办公室通过广域网 (WAN) 进行连接，该广域网可能是你自己的主干网络，也可能是你服务提供商的 IP VPN。你有两个 ExpressRoute 线路，一个在中国北部，一个在中国东部，也通过 WAN 连接。显然，你可以通过两个路径连接到 Azure 网络。现在，假设你在中国北部和中国东部都有 Azure 部署（例如 Azure App Service）。你的目的是将北京的用户连接到 Azure 中国北部，将上海的用户连接到 Azure 中国东部，因为你的服务管理员宣称每个办公室的用户都能够访问附近的 Azure 服务以获取最佳体验。遗憾的是，此方案适用于北部用户，但不适用于东部用户。以下是问题的原因。在每个 ExpressRoute 线路上，我们会向你播发 Azure 中国北部的前缀 (23.100.0.0/16) 和 Azure 中国东部的前缀 (13.100.0.0/16)。如果你不知道哪个前缀是来自哪个区域，则无法进行区别对待。你的 WAN 网络可能会认为这两种前缀更靠近中国北部而非中国东部，因此会将两个办公室的用户都路由到中国北部的 ExpressRoute 线路。结果就是，上海办公室的许多用户会很不满意。

![](./media/expressroute-optimize-routing/expressroute-case1-problem.png)

### 解决方案：使用 BGP 社区
若要优化两个办公室的用户的路由，你需要知道哪个前缀来自 Azure 中国北部，哪个前缀来自 Azure 中国东部。我们使用 [BGP 社区值](/documentation/articles/expressroute-routing)对此信息进行编码。我们向每个 Azure 区域分配了唯一的 BGP 社区值，例如：为中国北部分配“12076:51004”，为中国东部分配“12076:51006”。现在，你知道了哪个前缀来自哪个 Azure 区域，因此可以配置哪个 ExpressRoute 线路应该为首选线路。由于我们使用 BGP 来交换路由信息，因此你可以使用 BGP 的“本地首选项（local preference）”来影响路由。在我们的示例中，你可以在中国东部将比中国北部更高的本地首选项值分配给 13.100.0.0/16。类似地，你可以在中国北部将比中国东部更高的本地首选项值分配给 23.100.0.0/16。此配置将确保当通往 Azure 的两个路径都可用时，你在北京的用户将使用中国北部的 ExpressRoute 线路连接到 Azure 中国北部，而你在上海的用户将使用中国东部的 ExpressRoute 连接到 Azure 中国东部。两边的路由都获得了优化。

![](./media/expressroute-optimize-routing/expressroute-case1-solution.png)



<!---HONumber=Mooncake_1024_2016-->