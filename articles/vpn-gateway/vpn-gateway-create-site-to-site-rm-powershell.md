<properties
   pageTitle="使用 Azure Resource Manager 和 PowerShell 创建具有站点到站点 VPN 连接的虚拟网络 | Azure"
   description="本文逐步讲解如何使用 Resource Manager 部署模型创建 VNet，并使用 S2S VPN 网关连接将其连接到本地网络。"
   services="vpn-gateway"
   documentationCenter="na"
   authors="cherylmc"
   manager="carmonm"
   editor=""
   tags="azure-resource-manager"/>  


<tags
   ms.service="vpn-gateway"
   ms.devlang="na"
   ms.topic="hero-article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="10/14/2016"
   wacn.date=""
   ms.author="cherylmc"/>  


# 使用 PowerShell 创建具有站点到站点连接的 VNet

> [AZURE.SELECTOR]
- [Azure 门户预览](/documentation/articles/vpn-gateway-howto-site-to-site-resource-manager-portal/)
- [经典 - 经典管理门户](/documentation/articles/vpn-gateway-site-to-site-create/)

本文逐步讲解如何使用 Azure Resource Manager 部署模型创建一个虚拟网络和一个连接到本地网络的站点到站点 VPN 网关连接。站点到站点连接可以用于跨界和混合配置。

![站点到站点示意图](./media/vpn-gateway-create-site-to-site-rm-powershell/s2srmps.png "站点到站点")  



### 用于站点到站点连接的部署模型和方法

[AZURE.INCLUDE [部署模型](../../includes/vpn-gateway-deployment-models-include.md)]

下表显示了站点到站点配置当前可用的部署模型和方法。当有配置步骤相关的文章发布时，我们会直接从此表格链接到该文章。

[AZURE.INCLUDE [站点到站点连接表](../../includes/vpn-gateway-table-site-to-site-include.md)]

如果你想要将多个 VNet 连接到一起，但又不想创建连接到本地位置的连接，则请参阅[配置 VNet 到 VNet 连接](/documentation/articles/vpn-gateway-vnet-vnet-rm-ps/)。

若要将站点到站点连接添加到已有连接的 VNet，请参阅[此文](/documentation/articles/vpn-gateway-howto-multi-site-to-site-resource-manager-portal/)。

## 开始之前

在开始配置之前，请确认具有以下各项。

- 一台兼容的 VPN 设备和能够对其进行配置的人员。请参阅[关于 VPN 设备](/documentation/articles/vpn-gateway-about-vpn-devices/)。如果不熟悉 VPN 设备的配置，或者不熟悉本地网络配置中的 IP 地址范围，请咨询能够提供此类详细信息的人员。

- 一个用于 VPN 设备的面向外部的公共 IP 地址。此 IP 地址不得位于 NAT 之后。
	
- Azure 订阅。如果你还没有 Azure 订阅，你可以注册一个[试用版](/pricing/1rmb-trial)。
	
- 最新版本的 Azure Resource Manager PowerShell cmdlet。有关安装 PowerShell cmdlet 的详细信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。


## <a name="Login"></a>1.连接到订阅 

确保切换到 PowerShell 模式，以便使用资源管理器 cmdlet。有关详细信息，请参阅[将 Windows PowerShell 与资源管理器配合使用](/documentation/articles/powershell-azure-resource-manager/)。

打开 PowerShell 控制台并连接到你的帐户。使用下面的示例来帮助你连接：

	Login-AzureRmAccount -EnvironmentName AzureChinaCloud

检查该帐户的订阅。

	Get-AzureRmSubscription 

指定要使用的订阅。

	Select-AzureRmSubscription -SubscriptionName "Replace_with_your_subscription_name"

## <a name="VNet"></a>2.创建虚拟网络和网关子网

这些示例使用 /28 网关子网。尽管创建的网关子网最小可为 /29，但建议至少选择 /28 或 /27，创建包含更多地址的更大子网。这样便可以留出足够多的地址，满足将来可能需要使用的其他配置。

如果已拥有一个包含 /29 或更大网关子网的虚拟网络，则可以往前跳转到[添加本地网关](#localnet)。


[AZURE.INCLUDE [vpn-gateway-no-nsg](../../includes/vpn-gateway-no-nsg-include.md)]

### 若要创建虚拟网络和网关子网

使用以下示例创建一个虚拟网络和网关子网。替换为你自己的值。

首先，创建一个资源组：
	
	New-AzureRmResourceGroup -Name testrg -Location 'China North'

接下来，创建您的虚拟网络。请验证你指定的地址空间不与本地网络的任一个地址空间相重叠。

以下示例创建一个名为 *testvnet* 的虚拟网络和两个子网，其中一个名为 *GatewaySubnet* ，另一个名为 *Subnet1* 。特意创建一个名为 *GatewaySubnet* 的子网非常重要。如果您以其他名称为其命名，则您的连接配置将会失败。

设置变量。

	$subnet1 = New-AzureRmVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -AddressPrefix 10.0.0.0/28
	$subnet2 = New-AzureRmVirtualNetworkSubnetConfig -Name 'Subnet1' -AddressPrefix '10.0.1.0/28'

创建 VNet。

	New-AzureRmVirtualNetwork -Name testvnet -ResourceGroupName testrg `
	-Location 'China North' -AddressPrefix 10.0.0.0/16 -Subnet $subnet1, $subnet2

### <a name="gatewaysubnet"></a>将网关子网添加到已创建的虚拟网络

只有你需要向此前创建的 VNet 添加网关子网时，才需要使用此步骤。

可以使用以下示例创建网关子网。请务必将网关子网命名为“GatewaySubnet”。如果将其命名为其他名称，将创建子网，但 Azure 不将它视为网关子网。

设置变量。

	$vnet = Get-AzureRmVirtualNetwork -ResourceGroupName testrg -Name testvnet

创建网关子网。

	Add-AzureRmVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -AddressPrefix 10.0.3.0/28 -VirtualNetwork $vnet

设置配置。

	Set-AzureRmVirtualNetwork -VirtualNetwork $vnet

## 3\.<a name="localnet"></a>添加本地网关

在虚拟网络中，局域网网关通常指你的本地位置。指定该站点的名称以供 Azure 引用，同时指定局域网网关的地址空间前缀。

Azure 使用指定的 IP 地址前缀来识别要发送到本地位置的流量。这意味着需要指定要与局域网网关关联的各个地址前缀。如果本地网络出现变化，你可以轻松更新这些前缀。

在使用 PowerShell 示例时，请注意下列事项：
	
- *GatewayIPAddress* 是本地 VPN 设备的 IP 地址。VPN 设备不能位于 NAT 之后。
- *AddressPrefix* 是本地地址空间。

若要添加具有单个地址前缀的局域网网关：

	New-AzureRmLocalNetworkGateway -Name LocalSite -ResourceGroupName testrg `
	-Location 'China North' -GatewayIpAddress '23.99.221.164' -AddressPrefix '10.5.51.0/24'

若要添加具有多个地址前缀的局域网网关：

	New-AzureRmLocalNetworkGateway -Name LocalSite -ResourceGroupName testrg `
	-Location 'China North' -GatewayIpAddress '23.99.221.164' -AddressPrefix @('10.0.0.0/24','20.0.0.0/24')

### 若要为你的局域网网关修改 IP 地址前缀：

有时你的局域网网关前缀会有变化。修改 IP 地址前缀时采取的步骤取决于是否已创建 VPN 网关连接。请参阅本文的[修改本地网关的 IP 地址前缀](#modify)部分。


## <a name="PublicIP"></a>4.为 VPN 网关请求一个公共 IP 地址

接下来，需要请求一个公共 IP 地址并将其分配到 Azure VNet VPN 网关。这个地址与分配到 VPN 设备的 IP 地址不同，而是分配到 Azure VPN 网关本身。无法指定要使用的 IP 地址。它会动态分配到网关。在配置本地 VPN 设备连接到网关时，将使用这个 IP 地址。

Resource Manager 部署模型的 Azure VPN 网关目前使用动态分配方法，仅支持公共 IP 地址。但是，这并不意味着 IP 地址会更改。Azure VPN 网关 IP 地址只在删除或重新创建网关时更改。网关公共 public IP 地址不会因为重新调整大小、重置或其他 Azure VPN 网关内部维护/升级而更改。

使用以下 PowerShell 示例：

	$gwpip= New-AzureRmPublicIpAddress -Name gwpip -ResourceGroupName testrg -Location 'China North' -AllocationMethod Dynamic

## <a name="GatewayIPConfig"></a>5.创建网关 IP 寻址配置

网关配置定义要使用的子网和公共 IP 地址。使用以下示例创建网关配置。

	$vnet = Get-AzureRmVirtualNetwork -Name testvnet -ResourceGroupName testrg
	$subnet = Get-AzureRmVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -VirtualNetwork $vnet
	$gwipconfig = New-AzureRmVirtualNetworkGatewayIpConfig -Name gwipconfig1 -SubnetId $subnet.Id -PublicIpAddressId $gwpip.Id 

## <a name="CreateGateway"></a>6.创建虚拟网络网关

本步骤创建虚拟网络网关。创建网关可能需要很长时间才能完成。通常需要 45 分钟或更长的时间。

使用以下值：

- 站点到站点配置的 *-GatewayType* 为 *Vpn* 。网关类型永远是你要实现的配置的特定类型。例如，其他网关配置可能需要 -GatewayType ExpressRoute。

- *-VpnType* 可以是 *RouteBased* （在某些文档中称为动态网关）或 *PolicyBased* （在某些文档中称为静态网关）。有关 VPN 网关类型的详细信息，请参阅[关于 VPN 网关](/documentation/articles/vpn-gateway-about-vpngateways/#vpntype)。
- *-GatewaySku* 可以是 *Basic* 、 *Standard* 或 *HighPerformance* 。

		New-AzureRmVirtualNetworkGateway -Name vnetgw1 -ResourceGroupName testrg `
		-Location 'China North' -IpConfigurations $gwipconfig -GatewayType Vpn `
		-VpnType RouteBased -GatewaySku Standard

## <a name="ConfigureVPNDevice"></a>7.配置 VPN 设备

此时，需要使用虚拟网络网关的公共 IP 地址来配置本地 VPN 设备。请联系你的设备制造商以获得具体的配置信息。有关详细信息，请参阅 [VPN 设备](/documentation/articles/vpn-gateway-about-vpn-devices/)。

若要查找虚拟网络网关的公共 IP 地址，请使用下面的示例：

	Get-AzureRmPublicIpAddress -Name gwpip -ResourceGroupName testrg

## <a name="CreateConnection"></a>8.创建 VPN 连接

接下来，将在虚拟网络网关和 VPN 设备之间创建站点到站点 VPN 连接。请务必替换为你自己的值。共享密钥必须与你用于 VPN 设备配置的值匹配。请注意，站点到站点的 `-ConnectionType` 为 *IPsec* 。

设置变量。

	$gateway1 = Get-AzureRmVirtualNetworkGateway -Name vnetgw1 -ResourceGroupName testrg
	$local = Get-AzureRmLocalNetworkGateway -Name LocalSite -ResourceGroupName testrg

创建连接。

	New-AzureRmVirtualNetworkGatewayConnection -Name localtovon -ResourceGroupName testrg `
	-Location 'China North' -VirtualNetworkGateway1 $gateway1 -LocalNetworkGateway2 $local `
	-ConnectionType IPsec -RoutingWeight 10 -SharedKey 'abc123'

在一小段时间后，将建立该连接。

## <a name="toverify"></a>验证 VPN 连接

VPN 连接有几种不同的验证方式。

[AZURE.INCLUDE [vpn-gateway-verify-connection-rm](../../includes/vpn-gateway-verify-connection-rm-include.md)]

## <a name="modify"></a>修改本地网关的 IP 地址前缀

如果需要更改局域网网关的前缀，请使用下面的说明。提供了两套说明。要选择哪套说明取决于您是否已创建了网关连接。

[AZURE.INCLUDE [vpn-gateway-modify-ip-prefix-rm](../../includes/vpn-gateway-modify-ip-prefix-rm-include.md)]

## <a name="modifygwipaddress"></a>修改本地网关的 IP 地址

[AZURE.INCLUDE [vpn-gateway-modify-lng-gateway-ip-rm](../../includes/vpn-gateway-modify-lng-gateway-ip-rm-include.md)]

## 后续步骤

- 你可以将虚拟机添加到虚拟网络。请参阅[创建虚拟机](/documentation/articles/virtual-machines-windows-hero-tutorial/)以获取相关步骤。

- 有关 BGP 的信息，请参阅 [BGP 概述](/documentation/articles/vpn-gateway-bgp-overview/)和[如何配置 BGP](/documentation/articles/vpn-gateway-bgp-resource-manager-ps/)。

<!---HONumber=Mooncake_1031_2016-->