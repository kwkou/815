<properties
    pageTitle="发生影响 Azure 虚拟网络的 Azure 服务中断事件时该怎么办 | Azure"
    description="了解发生影响 Azure 虚拟网络的 Azure 服务中断事件时该怎么办。"
    services="virtual-network"
    documentationcenter=""
    author="NarayanAnnamalai"
    manager="jefco"
    editor="" />  

<tags
    ms.assetid="ad260ab9-d873-43b3-8896-f9a1db9858a5"
    ms.service="virtual-network"
    ms.workload="virtual-network"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="05/16/2016"
    wacn.date="12/26/2016"
    ms.author="narayan;aglick" />  


# 虚拟网络 - 业务连续性
## 概述
虚拟网络 (VNet) 是网络在云中的逻辑表示形式。它允许定义自己的专用 IP 地址空间以及将网络划分为子网。VNet 可用作信任边界来托管计算资源，例如 Azure 虚拟机和云服务（Web/辅助角色）。VNet 允许在其托管的资源之间直接进行专用 IP 通信。虚拟网络也可通过混合选项（例如 VPN 网关或 ExpressRoute）链接到本地网络。

VNet 在区域的范围内创建。可在两个不同的区域中创建具有相同地址空间的 VNet（即中国东部和美国西部，但无法让它们直接互相连接）。

## 业务连续性
可能有多种不同的情况会让应用程序中断。给定区域可能会因自然灾难而完全中断，或因多个设备/服务失败而部分中断。每种情况对 VNet 服务的影响各不相同。

**问：整个区域发生中断（即，某个区域因自然灾难而完全中断）怎么办？ 该区域中托管的虚拟网络会发生什么情况？**

答：在服务中断期间，受影响区域中的虚拟网络和资源始终保持无法访问的状态。

![虚拟网络简单关系图](./media/virtual-network-disaster-recovery-guidance/vnet.png)  


**问：若要在不同的区域中重新创建相同的虚拟网络，应该怎么做？**

答：虚拟网络 (VNet) 是非常轻量级的资源。可以调用 Azure API，在不同的区域中创建具有相同地址空间的 VNet。若要重新创建受影响区域具有的相同环境，需要进行 API 调用，重新部署云服务（Web/辅助角色）和拥有的虚拟机。如果具有本地连接（例如在混合部署中），还需要启动 VPN 网关并连接到本地网络。

[此处](/documentation/articles/virtual-networks-create-vnet-arm-pportal/)提供创建 VNet 的说明。

**问：是否可以提前在另一个区域中重新创建给定区域中的 VNet 的副本？**

答：可以，可提前在两个不同的区域中，使用相同的专用 IP 地址空间和资源来创建两个 VNet。如果客户在 VNet 中托管面向 Internet 的服务，他们可能已设置流量管理器，将流量异地路由到处于活跃状态的区域。但是，客户无法将具有相同地址空间的两个 VNet 连接到本地网络，因为这会导致路由问题。因发生灾难而失去一个区域中的 VNet 时，客户可将位于可用区域中、具有匹配的地址空间的另一个 VNet 连接到本地网络。

[此处](/documentation/articles/virtual-networks-create-vnet-arm-pportal/)提供创建 VNet 的说明。

<!---HONumber=Mooncake_1219_2016-->