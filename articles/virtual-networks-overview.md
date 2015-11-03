<properties
   pageTitle="Azure 虚拟网络 (VNet) 概述"
   description="了解 Azure 中的虚拟网络 (VNet)"
   services="virtual-network"
   documentationCenter="na"
   authors="telmosampaio"
   manager="carolz"
   editor="tysonn" />
<tags
   ms.service="virtual-network"
   ms.date="09/14/2015"
   wacn.date="11/02/2015" />

# 虚拟网络概述

Azure 虚拟网络 (VNet) 是你自己的网络在云中的表示形式。你可以控制 Azure 网络设置并定义 DHCP 地址块、DNS 设置、安全策略和路由。还可以进一步将 VNet 细分为子网，并以将物理计算机和虚拟机部署到本地数据中心的相同方式部署 Azure IaaS 虚拟机 (VM) 和 PaaS 角色实例。从本质上讲，你可以将网络扩展到 Azure，自带 IP 地址块。

若要更好地了解 VNet，请看下图，其中显示了简化的本地网络。

![本地网络](./media/virtual-networks-overview/figure01.png)

上图显示了通过路由器连接到公共 Internet 的本地网络。你还可以看到路由器与托管 DNS 服务器和 Web 服务器场的外围网络之间的防火墙。Web 服务器场使用向 Internet 公开的硬件负载平衡器进行负载平衡，并使用内部子网中的资源。内部子网由另一个防火墙与外围网络隔开，并托管 Active Directory 域控制器服务器、数据库服务器和应用程序服务器。

可以在 Azure 中托管同一网络，如下图所示。

![Azure 虚拟网络](./media/virtual-networks-overview/figure02.png)

请注意 Azure 基础结构如何起着路由器作用，允许从 VNet 访问公共 Internet 而无需进行任何配置。防火墙可由应用于每个单独子网的网络安全组 (NSG) 替代。而物理负载平衡器可由 Azure 中面向 Internet 的负载平衡器和内部负载平衡器替代。

## 虚拟网络

VNet 为部署到它们的 IaaS VM 和 PaaS 角色的角色实例提供以下服务：

- **隔离**。VNet 彼此之间完全隔离。这使你可以为使用相同 CIDR 地址块的开发、测试和生产创建单独的 VNet。

- **包含**。VNet 不能跨多个 Azure 区域。

    >[AZURE.NOTE]在 Azure 中有两种部署模式：经典（也称为服务管理）和 Azure 资源管理器 (ARM)。不能将经典 VNet 添加到地缘组，或创建为区域 VNet。如果你在地缘组中有一个 VNet，建议你[将它迁移到区域 VNet](/documentation/articles/virtual-networks-migrate-to-regional-vnet)。

- **访问公共 Internet**。默认情况下，VNet 中的所有 IaaS VM 和 PaaS 角色实例都可以访问公共 Internet。可以通过使用网络安全组 (NSG) 来控制访问。

- **访问 VNet 中的 VM**。同一 VNet 中的 IaaS VM 和 PaaS 角色实例可以互相连接，即使它们位于不同子网，也无需配置网关或使用公共 IP 地址，从而将 PaaS 环境和 IaaS 环境连接在一起。

- **名称解析**。Azure 为部署在 VNet 中的 IaaS VM 和 PaaS 角色实例提供内部名称解析。你还可以部署自己的 DNS 服务器，并将 VNet 配置为使用这些服务器。

- **连接**。VNet 通过使用站点到站点 VPN 连接或 ExpressRoute 连接可以彼此连接，甚至可以连接到本地数据中心。若要了解有关 ExpressRoute 的详细信息，请访问 [ExpressRoute 技术概述](/documentation/articles/expressroute-introduction)。

    >[AZURE.NOTE]请确保在将任何 IaaS VM 或 PaaS 角色实例部署到 Azure 环境之前创建 VNet。基于 ARM 的 VM 需要 VNet，如果你未指定现有 VNet，Azure 将创建其 CIDR 地址块可能会与本地网络冲突的默认 VNet。使你无法将 VNet 连接到本地网络。

## 子网

可以为组织和安全性将 VNet 划分为多个子网。一个 VNet 中的子网可以相互通信，而无需任何额外的配置。此外，还可以更改子网级别的路由设置并将 NSG 应用于子网。

## IP 地址

有两种类型的 IP 地址分配给 Azure 中的组件：公共和专用。自动基于分配给子网的 CIDR 地址块为部署到 Azure 子网的 IaaS VM 和 PaaS 角色实例分配专用 IP 地址到其每个 NIC。你还可以为 IaaS VM 和 PaaS 角色实例分配公共 IP 地址。

这些 IP 地址是动态的，意味着它们可以随时更改。你可能希望确保某些服务的 IP 地址始终保持不变。为此，可以保留一个 IP 地址，将设为静态。

## Azure 负载平衡器

在 Azure 中可以使用两种类型的负载平衡器：

- **外部负载平衡器**。可以使用外部负载平衡器为从公共 Internet 访问的 IaaS VM 和 PaaS 角色实例提供高可用性。

- **内部负载平衡器**。可以使用内部负载平衡器为从 VNet 中的其他服务访问的 IaaS VM 和 PaaS 角色实例提供高可用性。

<!--若要了解有关 Azure 中的负载平衡的详细信息，请访问[负载平衡器概述](/documentation/articles/load-balancer-overview)。-->

## 网络安全组 (NSG)

可以创建 NSG 来控制对网络接口 (NIC)、VM 和子网的入站和出站访问。每个 NSG 包含一条或多条规则，这些规则指定基于源 IP 地址、源端口、目标 IP 地址和目标端口是批准还是拒绝流量。若要了解有关 NSG 的详细信息，请访问[什么是网络安全组](/documentation/articles/virtual-networks-nsg)。

## 虚拟设备

虚拟设备就是 VNet 中的另一种 VM，它运行基于软件的设备函数，如防火墙、WAN 优化或入侵检测。你可以在 Azure 中创建一个路由，以便通过虚拟设备路由 VNet 流量来使用其功能。

例如，NSG 可用于在 VNet 上提供安全性。但是，NSG 为传入和传出数据包提供第 4 层访问控制列表 (ACL)。如果你要使用第 7 层安全模型，则需要使用防火墙设备。

虚拟设备依赖于[用户定义的路由和 IP 转发](/documentation/articles/virtual-networks-udr-overview)。

## 后续步骤

- [创建 VNet](/documentation/articles/virtual-networks-create-a-vnet) 和子网。
- [在 VNet 中创建 VM](/documentation/articles/virtual-machines-windows-tutorial)。
- 了解 [NSG](/documentation/articles/virtual-networks-nsg)。
<!--- 了解[负载平衡器](/documentation/articles/load-balancer-overview)。-->
- [保留内部 IP 地址](/documentation/articles/virtual-networks-reserved-private-ip)
- [保留公共 IP 地址](/documentation/articles/virtual-networks-reserved-public-ip)。
- 了解[用户定义的路由和 IP 转发](/documentation/articles/virtual-networks-udr-overview)。

<!---HONumber=76-->