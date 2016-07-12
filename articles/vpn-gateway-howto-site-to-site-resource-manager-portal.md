<!-- Ibiza Portal: tested -->

<properties
   pageTitle="使用 Azure Resource Manager 和 Azure 门户预览创建具有站点到站点 VPN 连接的虚拟网络 | Azure"
   description="本文将指导你完成使用资源管理器模型创建 VNet 并使用 S2S VPN 网关连接将其连接到你的本地网络。"
   services="vpn-gateway"
   documentationCenter="na"
   authors="cherylmc"
   manager="carmonm"
   editor=""
   tags="azure-resource-manager"/>

<tags
   ms.service="vpn-gateway"
   ms.date="05/13/2016"
   wacn.date="06/08/2016"/>

# 使用 Azure 门户预览和 Azure Resource Manager 创建具有站点到站点 VPN 连接的 VNet

> [AZURE.SELECTOR]
- [Azure 门户预览](/documentation/articles/vpn-gateway-howto-site-to-site-resource-manager-portal/)
- [Azure 经典管理门户](/documentation/articles/vpn-gateway-site-to-site-create/)
- [PowerShell - Resource Manager](/documentation/articles/vpn-gateway-create-site-to-site-rm-powershell/)


本文将指导你使用 Azure Resource Manager 部署模型和 Azure 门户预览创建一个虚拟网络和一个连接到本地网络的站点到站点 VPN 连接。在以下步骤中，你将创建 VNet、添加网关子网、网关、本地站点及连接。此外，还需要配置 VPN 设备。



**关于 Azure 部署模型**

[AZURE.INCLUDE [vpn-gateway-clasic-rm](../includes/vpn-gateway-classic-rm-include.md)]

## 连接关系图

![站点到站点](./media/vpn-gateway-howto-site-to-site-resource-manager-portal/site2site.png)

**用于站点到站点连接的部署模型和工具**

[AZURE.INCLUDE [vpn-gateway-table-site-to-site-table](../includes/vpn-gateway-table-site-to-site-include.md)]

如果你想要将多个 VNet 连接到一起，但又不想创建连接到本地位置的连接，则请参阅[配置 VNet 到 VNet 连接](/documentation/articles/vpn-gateway-vnet-vnet-rm-ps/)。如果你正在寻找不同类型的连接配置，请参阅 [VPN 网关连接拓扑](/documentation/articles/vpn-gateway-topology/)一文。

## 开始之前

在开始配置之前，请确认你具有以下各项。

- 一台兼容的 VPN 设备和能够对其进行配置的人员。请参阅[关于 VPN 设备](/documentation/articles/vpn-gateway-about-vpn-devices/)。如果你不熟悉 VPN 设备的配置，或者不熟悉本地网络配置中的 IP 地址范围，则需咨询能够为你提供此类详细信息的人员。

- 一个用于 VPN 设备的面向外部的公共 IP 地址。此 IP 地址不得位于 NAT 之后。
	
- Azure 订阅。如果你还没有 Azure 订阅，你可以注册一个[试用版](/pricing/1rmb-trial)。

### <a name="values"></a>此练习的示例配置值


练习这些步骤时，可以使用示例配置值。

- VNet 名称：TestVNet1
- 地址空间：10.11.0.0/16 和 10.12.0.0/16
- 子网： 
	- FrontEnd：10.11.0.0/24
	- BackEnd：10.12.0.0/24
	- GatewaySubnet：10.12.255.0/27
- 资源组：TestRG1
- 位置：中国东部
- DNS 服务器：8.8.8.8
- 网关名称：VNet1GW
- 公共 IP：VNet1GWIP
- VPN 类型：基于路由
- 连接类型：站点到站点 (IPsec)
- 网关类型：VPN
- 局域网网关名称：Site2
- 连接名称：VNet1toSite2



## 1\.创建虚拟网络 

如果已创建虚拟网络，请确认设置是否与 VPN 网关设计兼容，应特别注意任何可能与其他网络重叠的子网。如果你有重叠的子网，则连接将无法正常工作。如果你已确认为 VNet 配置了正确的设置，则可以开始执行[指定 DNS 服务器](#dns)部分中的步骤。

### 创建虚拟网络

[AZURE.INCLUDE [vpn-gateway-basic-vnet-rm-portal](../includes/vpn-gateway-basic-vnet-rm-portal-include.md)]

## 2\.添加其他地址空间和子网

可以将其他地址空间和子网添加到已创建的 VNet。

[AZURE.INCLUDE [vpn-gateway-additional-address-space](../includes/vpn-gateway-additional-address-space-include.md)]

## <a name="dns"></a>3.指定 DNS 服务器

如果你在练习创建此配置，请在指定 DNS 服务器时引用这些[值](#values)。

### 指定 DNS 服务器

[AZURE.INCLUDE [vpn-gateway-add-dns-rm-portal](../includes/vpn-gateway-add-dns-rm-portal-include.md)]

## 4\.创建网关子网

将虚拟网络连接到网关之前，必须先创建虚拟网络要连接的网关子网。创建的网关子网必须命名为 GatewaySubnet 才能正常工作。

某些配置的网关子网前缀需是 /28 或更大的子网才能容纳池中所需的 IP 地址数。这意味着网关子网前缀必须是 /28、/27、/26 等。可以在此处创建较大的子网，以适应将来可能的配置添加。

如果你在练习中创建此配置，请在创建网关子网时引用这些[值](#values)。

### 创建网关子网

[AZURE.INCLUDE [vpn-gateway-no-nsg](../includes/vpn-gateway-no-nsg-include.md)]

[AZURE.INCLUDE [vpn-gateway-add-gwsubnet-rm-portal](../includes/vpn-gateway-add-gwsubnet-rm-portal-include.md)]

## 5\.创建虚拟网络网关

如果你在练习创建此配置，请在创建网关时引用这些[值](#values)。

### 创建虚拟网络网关

[AZURE.INCLUDE [vpn-gateway-add-gw-rm-portal](../includes/vpn-gateway-add-gw-rm-portal-include.md)]

## 6\.创建局域网网关

局域网网关是指你的本地位置。你将为局域网网关命名，而 Azure 可通过该名称引用该站点。

如果你在练习创建此配置，请在添加本地站点时引用这些[值](#values)。

### 创建局域网网关

[AZURE.INCLUDE [vpn-gateway-add-lng-rm-portal](../includes/vpn-gateway-add-lng-rm-portal-include.md)]

## 7\.配置 VPN 设备

[AZURE.INCLUDE [vpn-gateway-configure-vpn-device-rm](../includes/vpn-gateway-configure-vpn-device-rm-include.md)]

## 8\.创建站点到站点 VPN 连接

接下来，你将在虚拟网络网关和 VPN 设备之间创建站点到站点 VPN 连接。请务必替换为你自己的值。共享密钥必须与你用于 VPN 设备配置的值匹配。

在开始本部分之前，请确认虚拟网络网关与局域网网关已完成创建。如果你在练习创建此配置，请在创建连接时引用这些[值](#values)。

### 创建 VPN 连接


[AZURE.INCLUDE [vpn-gateway-add-site-to-site-connection-rm-portal](../includes/vpn-gateway-add-site-to-site-connection-rm-portal-include.md)]

## 9\.验证 VPN 连接

你可以在门户预览中或使用 PowerShell 验证 VPN 连接。

[AZURE.INCLUDE [vpn-gateway-verify-connection-rm](../includes/vpn-gateway-verify-connection-rm-include.md)]

## 后续步骤

- 连接完成后，即可将虚拟机添加到虚拟网络。有关详细信息，请参阅虚拟机[学习路径](https://azure.microsoft.com/documentation/learning-paths/virtual-machines)。

- 有关 BGP 的信息，请参阅 [BGP 概述](/documentation/articles/vpn-gateway-bgp-overview/)和[如何配置 BGP](/documentation/articles/vpn-gateway-bgp-resource-manager-ps/)。

<!---HONumber=Mooncake_0613_2016-->