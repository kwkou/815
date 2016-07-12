## 配置概述

此任务的步骤使用基于以下值的 VNet。此列表中也概述了其他设置和名称。尽管我们确实基于此列表中的值添加变量，但是我们在任何步骤中不会直接使用此列表。你可以复制列表作为参考，并将列表中的值替换为自己的值。

配置参考列表：
	
- 虚拟网络名称（VNetName） = “TestVNet”
- 虚拟网络地址空间 = 192.168.0.0/16
- 资源组（RG） = “TestRG”
- Subnet1 名称 = “FrontEnd” 
- Subnet1 地址空间 = “192.168.0.0/16”
- 网关子网名称（GatewaySubnet）：“GatewaySubnet” 必须始终将网关子网命名为 *GatewaySubnet*。
- 网关子网地址空间 = “192.168.200.0/26”
- 区域（Location） =“中国东部”
- 网关名称（GWName） = “GW”
- 网关 IP 名称（GWIPName） = “GWIP”
- 网关 IP 配置名称（GWIPconfName） = “gwipconf”
- VPN 类型（VPNType） = “ExpressRoute” ExpressRoute 配置需要此 VPN 类型。
- 网关公共 IP 名称（GWPIP） = “gwpip”


## 添加网关

1. 连接到 Azure 订阅。 

		Login-AzureRmAccount -Environment $(Get-AzureRmEnvironment -Name AzureChinaCloud)
		Get-AzureRmSubscription 
		Select-AzureRmSubscription -SubscriptionName "Name of subscription"

2. 声明此示例的变量。本示例将在以下例子中使用这些变量。请务必编辑此例子，以反映想要使用的设置。
		
		$RG = "TestRG"
		$Location = "China East"
		$GWName = "GW"
		$GWIPName = "GWIP"
		$GWIPconfName = "gwipconf"
		$VNetName = "TestVNet"

3. 将虚拟网络对象赋值。

		$vnet = Get-AzureRmVirtualNetwork -Name $VNetName -ResourceGroupName $RG

4. 将网关子网添加到虚拟网络中。网关子网必须命名为“GatewaySubnet”。想要创建 /27 或更大（/26、/25 等）的网关。
			
		Add-AzureRmVirtualNetworkSubnetConfig -Name GatewaySubnet -VirtualNetwork $vnet -AddressPrefix 192.168.200.0/26

5. 设置配置。

			Set-AzureRmVirtualNetwork -VirtualNetwork $vnet

6. 将网关子网赋值。

		$subnet = Get-AzureRmVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -VirtualNetwork $vnet

7. 请求公共 IP 地址。创建网关之前请求 IP 地址。不支持静态指定IP， AllocationMethod 必须是动态的。

		$pip = New-AzureRmPublicIpAddress -Name gwpip -ResourceGroupName $RG -Location $Location -AllocationMethod Dynamic
		$ipconf = New-AzureRmVirtualNetworkGatewayIpConfig -Name $GWIPconfName -Subnet $subnet -PublicIpAddress $pip

8. 创建网关配置（gwipconfig）。网关配置定义要使用的子网和公共 IP 地址。在此步骤中，你将指定创建网关时使用的配置。此步骤不会实际创建网关对象。使用下面的示例创建你的网关配置。

		$gwipconfig = New-AzureRmVirtualNetworkGatewayIpConfig -Name $GWIPconfName -SubnetId $subnet.Id -PublicIpAddressId $pip.Id 

9. 创建网关。在此步骤中，**-GatewayType** 尤其重要。必须使用值 **ExpressRoute**。请注意，运行这些 cmdlet 后，可能需要 20 分钟或更多时间来创建网关。

		New-AzureRmVirtualNetworkGateway -Name $GWName -ResourceGroupName $RG -Location $Location -IpConfigurations $ipconf -GatewayType Expressroute -GatewaySku Standard

## 验证是否已创建网关

使用以下命令来验证是否已创建网关。

	Get-AzureRmVirtualNetworkGateway -ResourceGroupName $RG

## 重设网关大小

有三个[网关 SKU](/documentation/articles/vpn-gateway-about-vpngateways/)。你可以使用以下命令随时更改网关 SKU。

	$gw = Get-AzureRmVirtualNetworkGateway -Name $GWName -ResourceGroupName $RG
	Resize-AzureRmVirtualNetworkGateway -VirtualNetworkGateway $gw -GatewaySku HighPerformance

## 删除网关

使用以下命令可删除网关

	Remove-AzureRmVirtualNetworkGateway -Name $GWName -ResourceGroupName $RG  
<!---HONumber=Mooncake_0509_2016-->