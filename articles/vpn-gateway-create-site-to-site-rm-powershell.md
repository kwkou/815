<properties
   pageTitle="使用 Azure 资源管理器和 PowerShell 创建具有站点到站点 VPN 连接的虚拟网络 | Windows Azure"
   description="使用 Azure 资源管理器和 PowerShell 创建从虚拟网络到本地位置的站点到站点 VPN 连接"
   services="vpn-gateway"
   documentationCenter="na"
   authors="cherylmc"
   manager="carolz"
   editor=""
   tags="azure-resource-manager"/>

<tags
   ms.service="vpn-gateway"
   ms.date="07/28/2015"
   wacn.date="10/08/2015"/>

# 使用 Azure 资源管理器和 PowerShell 创建具有站点到站点 VPN 连接的虚拟网络

> [AZURE.SELECTOR]
- [Azure Portal](/documentation/articles/vpn-gateway-site-to-site-create)
- [PowerShell - Resource Manager](/documentation/articles/vpn-gateway-create-site-to-site-rm-powershell)


本主题将指导您创建一个 Azure 资源管理器虚拟网络和一个到本地网络的站点到站点 VPN 连接。

Azure 目前有两种部署模式：经典部署模式和 Azure 资源管理器部署模式。根据用于部署虚拟网络的模型，站点到站点设置会有所不同。以下说明适用于资源管理器。如果你想要使用经典部署模型创建站点到站点 VPN 网关连接，请参阅[在管理门户中创建站点到站点 VPN 连接](/documentation/articles/vpn-gateway-site-to-site-created)。


## 开始之前

在开始之前，请验证您具有以下各项：

- 一台兼容 VPN 设备（和能够对其进行配置的人员）。请参阅[关于 VPN 设备](/documentation/articles/vpn-gateway-about-vpn-devices)。
- 一个用于 VPN 设备的面向外部的公共 IP 地址。此 IP 地址不得位于 NAT 之后。
- 最新版本的 Azure PowerShell cmdlet。可以从[下载页面](http://azure.microsoft.com/downloads/)的 Windows PowerShell 部分下载并安装最新版本。 
- Azure 订阅。如果您还没有 Azure 订阅，可以注册一个 [Azure 试用版](http://www.windowsazure.cn/pricing/1rmb-trial/)。
	

## 连接到订阅 


打开 PowerShell 控制台并切换到 Azure 资源管理器模式。使用下面的示例来帮助您连接：

		Add-AzureAccount

使用 Select-AzureSubscription 连接到您要使用的订阅。

		Select-AzureSubscription "yoursubscription"

接下来，切换到 ARM 模式。这将切换模式以允许您使用 ARM cmdlet。

		Switch-AzureMode -Name AzureResourceManager


## 创建虚拟网络和网关子网

- 如果您已经拥有一个包含网关子网的虚拟网络，则可以跳转到[添加您的本地站点](#add-your-local-site)。 
- 如果您有一个虚拟网络并且希望向 VNet 添加一个网关子网，请参阅[向 VNet 添加网关子网](#gatewaysubnet)。

### 若要创建虚拟网络和网关子网

使用下面的示例创建一个虚拟网络和网关子网。替换为您自己的值。

首先，创建一个资源组：

	
		New-AzureResourceGroup -Name testrg -Location 'West US'

接下来，创建您的虚拟网络。下面的示例创建一个名为 *testvnet* 的虚拟网络和两个子网，其中一个名为 *GatewaySubnet*，另一个名为 *Subnet1*。特意创建一个名为 *GatewaySubnet* 的子网非常重要。如果您以其他名称为其命名，则您的连接配置将会失败。

		$subnet1 = New-AzureVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -AddressPrefix 10.0.0.0/28
		$subnet2 = New-AzureVirtualNetworkSubnetConfig -Name 'Subnet1' -AddressPrefix '10.0.1.0/28'
		New-AzurevirtualNetwork -Name testvnet -ResourceGroupName testrg -Location 'West US' -AddressPrefix 10.0.0.0/16 -Subnet $subnet1, $subnet2


### <a name="gatewaysubnet"></a>向 VNet 添加网关子网

如果您已经有一个现有的虚拟网络并且希望在其中添加一个网关子网，可以通过使用下面的示例来创建网关子网。请务必将网关子网命名为“GatewaySubnet”。如果以其他名称为其命名，则您的 VPN 配置将无法按预期工作。


	
		$vnet = Get-AzureVirtualNetwork -ResourceGroupName testrg -Name testvnet
		Add-AzureVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -AddressPrefix 10.0.3.0/28 -VirtualNetwork $vnet
		Set-AzureVirtualNetwork -VirtualNetwork $vnet

## 添加您的本地站点

在虚拟网络中，*本地站点*通常指您的本地位置。您将为该站点命名，而 Azure 可通过该名称引用该站点。

您还将指定本地站点的地址空间前缀。Azure 将使用您指定的 IP 地址前缀来识别要发送到本地站点的流量。这意味着您必须指定要与本地站点关联的各个地址前缀。如果本地网络出现变化，您可以轻松更新这些前缀。使用下面的 PowerShell 示例来指定您的本地站点。

	
- *GatewayIPAddress* 是本地 VPN 设备的 IP 地址。VPN 设备不能位于 NAT 之后。 
- *AddressPrefix* 是本地地址空间。

使用此示例添加您的本地站点：

		New-AzureLocalNetworkGateway -Name LocalSite -ResourceGroupName testrg -Location 'West US' -GatewayIpAddress '23.99.221.164' -AddressPrefix '10.5.51.0/24'

## 为 VNet 网关请求一个公共 IP 地址

接下来，您需要请求一个公共 IP 地址并将其分配到 Azure VNet VPN 网关。这个地址与分配到 VPN 设备的 IP 地址不同，而是分配到 Azure VPN 网关本身。您无法指定要使用的 IP 地址；它动态分配到您的网关。在配置本地 VPN 设备连接到网关时，您将使用这个 IP 地址。

使用下面的 PowerShell 示例。此地址的分配方法必须是“动态”。

		$gwpip= New-AzurePublicIpAddress -Name gwpip -ResourceGroupName testrg -Location 'West US' -AllocationMethod Dynamic

## 创建网关 IP 寻址配置

网关配置定义要使用的子网和公共 IP 地址。使用下面的示例创建您的网关配置。


		$vnet = Get-AzureVirtualNetwork -Name testvnet -ResourceGroupName testrg
		$subnet = Get-AzureVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -VirtualNetwork $vnet
		$gwipconfig = New-AzureVirtualNetworkGatewayIpConfig -Name gwipconfig1 -SubnetId $subnet.Id -PublicIpAddressId $gwpip.Id 


## 创建网关

在此步骤中，您将创建虚拟网络网关。使用以下值：

- 网关类型为 *Vpn*。
- VpnType 可以是 RouteBased*（在某些文档中称为动态网关）或 *Policy Based*（在某些文档中称为静态网关）。有关 VPN 网关类型的详细信息，请参阅[关于 VPN 网关](vpn-gateway-about-vpngateways.md)。New-AzureVirtualNetworkGateway -Name vnetgw1 -ResourceGroupName testrg -Location 'West US' -IpConfigurations $gwipconfig -GatewayType Vpn -VpnType RouteBased

## 配置 VPN 设备

此时，您需要使用虚拟网络网关的公共 IP 地址来配置本地 VPN 设备。请联系您的设备制造商以获得具体的配置信息。此外，请参阅 [VPN 设备](http://go.microsoft.com/fwlink/p/?linkid=615099)以获得更多信息。

若要查找虚拟网络网关的公共 IP 地址，请使用下面的示例：

	Get-AzurePublicIpAddress -Name gwpip -ResourceGroupName testrg

## 创建 VPN 连接

接下来，您将在虚拟网络网关和 VPN 设备之间创建站点到站点 VPN 连接。请务必替换为您自己的值。共享密钥必须与你用于 VPN 设备配置的值匹配。

		$gateway1 = Get-AzureVirtualNetworkGateway -Name vnetgw1 -ResourceGroupName testrg
		$local = Get-AzureLocalNetworkGateway -Name LocalSite -ResourceGroupName testrg

		New-AzureVirtualNetworkGatewayConnection -Name localtovon -ResourceGroupName testrg -Location 'West US' -VirtualNetworkGateway1 $gateway1 -LocalNetworkGateway2 $local -ConnectionType IPsec -RoutingWeight 10 -SharedKey 'abc123'

几分钟后，连接应建立完毕。此时，通过资源管理器创建的站点到站点 VPN 连接在门户中不可见。


## 后续步骤

将虚拟机添加到虚拟网络。[创建虚拟机](/documentation/articles/virtual-machines-windows-tutorial)。

<!---HONumber=71-->