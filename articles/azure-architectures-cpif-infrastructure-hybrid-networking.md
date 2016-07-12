<properties 
   pageTitle="混合网络（Azure 体系结构模式）" 
   description="混合网络模式是基础结构区域的一部分，在 CPIF 体系结构文档中对其进行了广泛的介绍。" 
   services="" 
   documentationCenter="" 
   authors="arynes" 
   manager="fredhar" 
   editor=""/>

<tags
   ms.service="cloud-services"
   ms.date="03/25/2015"
   wacn.date="10/03/2015"/>

# 混合网络（Azure 体系结构模式）

[云平台集成框架 (CPIF)](/documentation/articles/azure-architectures-cpif-overview/) 提供工作负荷整合指南，将应用程序载入到 Microsoft 云解决方案中。

CPIF 介绍了组织、客户和合作伙伴应该如何设计和部署使用混合云平台及 Azure、System Center 和 Windows Server 管理功能的面向云的工作负荷。

**混合网络**模式是**基础结构**区域的一部分，在 CPIF 体系结构文档中对其进行了广泛的介绍。

##  混合网络

混合网络设计模式详细说明了 Azure 功能，以及能跨地理边界提供可预测性能和高可用性的网络功能所需的服务。Azure 文档站点内提供了 Azure 区域的完整列表和每个区域可用的服务。本文档概述了在混合环境下的 Azure 网络功能。Azure 虚拟网络可让您在 Azure 中创建逻辑隔离的网络，并通过 Internet 或使用专用网络连接将其安全地连接到您的本地数据中心。此外，单个客户端机器可以使用 IPsec VPN 连接，连接到隔离的 Azure 网络。

混合网络体系结构模式包括以下重点领域：

- 连接本地网络到 Azure 
- 跨区域扩展 Azure 虚拟网络 
- 扩展订阅之间的 Azure 虚拟网络 
- 提供开发人员的远程网络访问 

## 体系结构模式概述 

受可创建场景的可能数量影响，混合网络体系结构模式比较复杂。该体系结构模式将主要集中在以下四种场景：

- 站点到站点混合网络，以及多跃点虚拟网络中单个订阅和区域内路由 
- 站点到站点混合网络，以及多跃点虚拟网络中跨订阅和区域路由 
- ExpressRoute 混合网络使用 MPLS 连接 
- ExpressRoute 混合网络使用 IXP 连接 

##  其他资源
[混合网络 (pdf)](https://gallery.technet.microsoft.com/Cloud-Platform-Integration-5e401f38)

## 另请参阅
[CPIF 体系结构](https://gallery.technet.microsoft.com/Cloud-Platform-Integration-bd1e434a)

[全局负载平衡的 Web 层](https://gallery.technet.microsoft.com/Cloud-Platform-Integration-2c3c663a)

[负载平衡的数据层](https://gallery.technet.microsoft.com/Cloud-Platform-Integration-dfb09e41)

[Azure 搜索层](https://gallery.technet.microsoft.com/Cloud-Platform-Integration-e581d65d)

[批处理层](https://gallery.technet.microsoft.com/Cloud-Platform-Integration-0bc3f8b1)

<!---HONumber=71-->