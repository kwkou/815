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
   ms.date="03/08/2016"
   wacn.date="04/05/2016"/>

# 在经典门户中为 ExpressRoute 配置虚拟网络

本文中的步骤将指导你通过经典部署模型和管理门户配置用于 ExpressRoute 的虚拟网络和网关。

有关 Resource Manager 部署模型的说明，请参阅以下文章，其中将会指导你使用 PowerShell 创建虚拟网络，以及[将 VPN 网关添加到 ExpressRoute 的 Resource Manager VNet](/documentation/articles/expressroute-howto-add-gateway-resource-manager/)。

**关于 Azure 部署模型**

[AZURE.INCLUDE [vpn-gateway-clasic-rm](../includes/vpn-gateway-classic-rm-include.md)]

## 配置 VNet 和网关

1. 登录到 [Azure 管理门户](http://manage.windowsazure.cn)。

2. 在屏幕左下角，单击“新建”。在导航窗格中，单击“网络服务”，然后单击“虚拟网络”。单击“自定义创建”以启动配置向导。

3. 在“虚拟网络详细信息”页上，输入以下信息。

	- **名称** - 为虚拟网络命名。在部署你的 VM 和 PaaS 实例时，将使用此虚拟网络名称，因此最好不要让此名称太复杂。
	- **位置** - 位置直接与你想让资源 (VM) 驻留在的物理位置（区域）有关。例如，如果你希望部署到此虚拟网络的 VM 的物理位置位于美国东部，请选择该位置。创建虚拟网络后，将无法更改与虚拟网络关联的区域。

4. 在“DNS 服务器和 VPN 连接”页上，输入以下信息，然后单击右下角的“下一步”箭头。

	- **DNS 服务器** - 输入 DNS 服务器名称和 IP 地址，或从下拉列表中选择一个以前注册的 DNS 服务器。此设置不创建 DNS 服务器，但可以指定要用于对此虚拟网络进行名称解析的 DNS 服务器。
	- **配置站点到站点 VPN** - 选中“配置站点到站点 VPN”复选框。
	- **选择 ExpressRoute** - 选中“使用 ExpressRoute”复选框。仅当你在上一步中选中了***“配置站点到站点 VPN”***，才会显示此选项。
	- **本地网络** - 本地网络表示你的物理本地位置。你可以选择以前创建的本地网络，也可以创建一个新的本地网络。

	如果选择现有的本地网络，请跳过步骤 5。

5. 如果你要创建新的本地网络，则会看到“站点到站点连接”页。如果你选择了以前创建的本地网络，则不会在向导中显示此页，你可以转到下一部分。若要配置本地网络，请输入以下信息，然后单击“下一步”箭头。

	- **名称** - 要称呼本地（内部）网络站点的名称。
	- **地址空间** - 包括“起始 IP”和“CIDR (地址计数)”。你可以指定任何地址范围，但前提是它不能与虚拟网络的地址范围重叠。
	- **添加地址空间** - 此设置与 ExpressRoute 的设置不相关。**注意：**你需要为 ExpressRoute 创建本地网络站点。将忽略为本地网络站点指定的地址前缀。通过 ExpressRoute 线路公布给 Microsoft 的地址前缀将用于路由目的。

6. 在“虚拟网络地址空间”页上，输入以下信息，然后单击右下角的复选框以配置网络。

	- **地址空间** - 包括起始 IP 和地址计数。请验证你指定的地址空间不与本地网络的任一个地址空间相重叠。
	- **添加子网** - 包括起始 IP 和地址计数。附加的子网不是必需的，但你可能需要为具有动态 IP 地址 (DIP) 的 VM 创建一个单独的子网。或者你可能需要在子网中拥有与 PaaS 实例分开的 VM。
	- **添加网关子网** - 单击此项可添加网关子网。网关子网仅用于此虚拟网络网关，并且是此配置必需的。***重要说明：***ExpressRoute 的网关子网前缀必须是 /28 或更小值（/27、/26 等）。

7. 单击页面底部的复选标记，此时将开始创建虚拟网络。创建完成后，在 Azure 经典门户的“网络”页上，你将看到“状态”下面列出了“已创建”。

8. 在“网络”页上，单击刚创建的虚拟网络，然后单击“仪表板”。
9. 在“仪表板”页的底部，单击“创建网关”，然后单击“是”。

10. 开始创建网关时，你将看到一条消息，告知网关已经开始创建。创建网关最多可能需要 15 分钟。

11. 将你的网络连接到线路。遵循[如何将 VNet 链接到 ExpressRoute 线路](/documentation/articles/expressroute-howto-linkvnet-classic/)一文中的说明。

## 后续步骤

- 如果要将虚拟机添加到虚拟网络，请参阅[虚拟机学习路径](/documentation/services/virtual-machines/)。
- 如果要了解有关 ExpressRoute 的详细信息，请参阅 [ExpressRoute 技术概述](/documentation/articles/expressroute-introduction/)。


 

<!---HONumber=Mooncake_0104_2016-->