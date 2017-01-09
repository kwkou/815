<properties
   pageTitle="为 ExpressRoute 配置虚拟网络和网关 | Microsoft Azure"
   description="本文指导你使用经典部署模型为 ExpressRoute 设置虚拟网络 (VNet)。"
   documentationCenter="na"
   services="expressroute"
   authors="cherylmc"
   manager="carmonm"
   editor=""
   tags="azure-service-management"/>

<tags 
   ms.service="expressroute"
   ms.devlang="na"
   ms.topic="article" 
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services" 
   ms.date="09/20/2016" 
   wacn.date="01/09/2017"
   ms.author="cherylmc"/>  


# 在经典管理门户中为 ExpressRoute 创建虚拟网络

本文中的步骤指导通过经典管理部署模型和经典门户配置用于 ExpressRoute 的虚拟网络和虚拟网络网关。

有关 Resource Manager 部署模型的说明，请参阅以下文章：[使用 PowerShell 创建虚拟网络](/documentation/articles/virtual-networks-create-vnet-arm-ps/)和[将 VPN 网关添加到 ExpressRoute 的 Resource Manager VNet](/documentation/articles/expressroute-howto-add-gateway-resource-manager/)。

**关于 Azure 部署模型**

[AZURE.INCLUDE [vpn-gateway-clasic-rm](../../includes/vpn-gateway-classic-rm-include.md)]

## 创建经典 VNet 和网关

以下步骤创建经典 VNet 和虚拟网络网关。如果你已经有了一个经典 VNet，请参阅本文的[配置现有的经典 VNet](#config) 部分。

1. 登录到 [Azure 管理门户](http://manage.windowsazure.cn)。

2. 在屏幕左下角，单击“新建”。在导航窗格中，单击“网络服务”，然后单击“虚拟网络”。单击“自定义创建”以启动配置向导。

3. 在“虚拟网络详细信息”页中输入以下信息：

	- **名称** - 为虚拟网络命名。在部署你的 VM 和 PaaS 实例时，将使用此虚拟网络名称，因此最好不要让此名称太复杂。
	- **位置** - 位置直接与你想让资源 (VM) 驻留在的物理位置（区域）有关。例如，如果你希望部署到此虚拟网络的 VM 的物理位置位于美国东部，请选择该位置。创建虚拟网络后，将无法更改与虚拟网络关联的区域。

4. 在“DNS 服务器和 VPN 连接”页上，输入以下信息，然后单击右下角的“下一步”箭头。

	- **DNS 服务器**：输入 DNS 服务器名称和 IP 地址，或从快捷菜单中选择一个以前注册的 DNS 服务器。此设置不创建 DNS 服务器。此设置允许指定要用于对此虚拟网络进行名称解析的 DNS 服务器。
	- **站点到站点连接**：选中“配置站点到站点 VPN”复选框。
	- **ExpressRoute**：选中“使用 ExpressRoute”复选框。仅在你选中了“配置站点到站点 VPN”的情况下，才会显示此选项。
	- **本地网络** - 你需要有一个针对 ExpressRoute 的本地网络站点。但是，在使用 ExpressRoute 连接时，将忽略为本地网络站点指定的地址前缀，而使用通过 ExpressRoute 线路播发到 Azure 的地址前缀进行路由。<BR>如果你已经针对 ExpressRoute 连接创建了本地网络，则可从下拉列表中进行选择。否则，请选择“指定新的本地网络”。

5. 如果在前面的步骤中选择指定新的本地网络，则会显示“站点到站点连接”页。若要配置本地网络，请输入以下信息，然后单击“下一步”箭头。

	- **名称** - 要称呼本地（内部）网络站点的名称。
	- **地址空间** - 包括“起始 IP”和“CIDR (地址计数)”。你可以指定任何地址范围，但前提是它不能与虚拟网络的地址范围重叠。通常情况下，这会为你的本地网络指定地址范围，但在使用 ExpressRoute 时，不会使用这些设置。不过，当你使用经典管理门户时，必须使用该设置才能创建本地网络。
	- **添加地址空间** - 此设置与 ExpressRoute 的设置不相关。


6. 在“虚拟网络地址空间”页上，输入以下信息，然后单击右下角的复选框以配置网络。

	- **地址空间** - 包括起始 IP 和地址计数。请验证你指定的地址空间不与本地网络的任一个地址空间相重叠。
	- **添加子网** - 包括起始 IP 和地址计数。不需其他子网。
	- 添加网关子网 - 单击此项可添加网关子网。网关子网仅用于虚拟网络网关，是此配置所必需的。<BR>ExpressRoute 的网关子网 CIDR（地址计数）必须是 /28 或更大（/27、/26 等）。这样一来，该子网中就会有足够的 IP 地址，配置就会起作用。在经典管理门户中，如果选择了“使用 ExpressRoute”复选框，门户会使用 /28 来指定网关子网。不能在经典管理门户中调整 CIDR 地址计数。网关子网在经典管理门户中将显示为“网关”，虽然已创建的网关子网的真实名称实际上是 **GatewaySubnet**。你可以通过 PowerShell 或 Azure 门户查看此名称。

7. 单击页面底部的复选标记，此时将开始创建虚拟网络。完成后，在经典管理门户的“网络”页上，可看到“状态”下面列出了“已创建”。

## <a name="gw"></a>创建网关

1. 在“网络”页上，单击刚创建的虚拟网络，然后单击页面顶部的“仪表板”。

2. 在“仪表板”页的底部，单击“创建网关”，然后选择“动态路由”。单击“是”确认你要创建网关。

3. 开始创建网关时，你将看到一条消息，告知网关已经开始创建。创建网关最多可能需要 45 分钟。

11. 将你的网络连接到线路。遵循[如何将 VNet 链接到 ExpressRoute 线路](/documentation/articles/expressroute-howto-linkvnet-classic/)一文中的说明。

## <a name="config"></a>为 ExpressRoute 配置现有的经典 VNet

如果你已经有一个经典的 VNet，则可将其配置为在经典管理门户中连接到 ExpressRoute。设置如以上部分所述，因此请通读这些部分以熟悉所需设置。如果想要创建 ExpressRoute/站点到站点共存连接，请参阅[此文](/documentation/articles/expressroute-howto-coexist-classic/)以了解相关步骤。这些步骤不同于本文中的步骤。
 
1. 需要先创建本地网络，然后才能更新 VNet 设置的其余部分。若要创建新的本地网络（这是通过经典管理门户配置 ExpressRoute 所必需的），请单击“新建”>“网络服务”>“虚拟网络”>“添加本地网络”。按照向导步骤创建本地网络。

2. 使用“配置”页更新 VNet 的设置的其余部分，并将 VNet 关联到本地网络。

3. 配置设置以后，请转到本文的[创建网关](#gw)部分，以便创建网关。


## 后续步骤

- 如果要将虚拟机添加到虚拟网络，请参阅[虚拟机学习路径](/documentation/services/virtual-machines/)。
- 如果要了解有关 ExpressRoute 的详细信息，请参阅 [ExpressRoute 概述](/documentation/articles/expressroute-introduction/)。


 

<!---HONumber=Mooncake_Quality_Review_0104_2017-->