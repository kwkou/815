若要修改网关 IP 地址，请使用 `New-AzureRmVirtualNetworkGatewayConnection` cmdlet。只要保持本地网络网关名称和现有名称完全相同，就会覆盖这些设置。此时，“Set”cmdlet 不支持修改网关 IP 地址。

### <a name="gwipnoconnection"></a>如何修改网关 IP 地址 - 无网关连接

若要更新尚没有连接的本地网络网关的网关 IP 地址，请使用以下示例。此外可同时更新地址前缀。所指定的设置将覆盖现有设置。请务必使用本地网络网关的现有名称。如果不这样做，将创建一个新的本地网络网关，而不是覆盖现有本地网络网关。

使用下面的示例，将值替换为自己的值。

	New-AzureRmLocalNetworkGateway -Name MyLocalNetworkGWName `
	-Location "China North" -AddressPrefix @('10.0.0.0/24','20.0.0.0/24','30.0.0.0/24') `
	-GatewayIpAddress "5.4.3.2" -ResourceGroupName MyRGName


### <a name="gwipwithconnection"></a>如何修改网关 IP 地址 - 现有网关连接

如果网关连接已存在，首先需要删除该连接。然后，可修改网关 IP 地址并重新创建一个新的连接。这将导致 VPN 连接中断一段时间。


>[AZURE.IMPORTANT] 不要删除 VPN 网关。如果将其删除，则必须返回执行相应步骤来重新创建它，并使用将分配到新创建的网关的 IP 地址来重新配置本地路由器。
 

1. 删除连接。可使用 `Get-AzureRmVirtualNetworkGatewayConnection` cmdlet 找到连接的名称。

		Remove-AzureRmVirtualNetworkGatewayConnection -Name MyGWConnectionName `
		-ResourceGroupName MyRGName

2. 修改 GatewayIpAddress 值。如有必要，此时也可修改地址前缀。请注意，这将覆盖现有的本地网络网关设置。修改时使用本地网络网关的现有名称，以覆盖这些设置。如果不这样做，将创建一个新的本地网络网关，而不是修改现有本地网络网关。

		New-AzureRmLocalNetworkGateway -Name MyLocalNetworkGWName `
		-Location "China North" -AddressPrefix @('10.0.0.0/24','20.0.0.0/24','30.0.0.0/24') `
		-GatewayIpAddress "104.40.81.124" -ResourceGroupName MyRGName

3. 创建连接。在此示例中，我们将配置 IPsec 连接类型。重新创建连接时，请使用针对配置指定的连接类型。有关其他连接类型，请参阅 [PowerShell cmdlet](https://msdn.microsoft.com/zh-cn/library/mt603611.aspx) 页面。若要获取 VirtualNetworkGateway 名称，可运行 `Get-AzureRmVirtualNetworkGateway` cmdlet。

	设置变量：

		$local = Get-AzureRMLocalNetworkGateway -Name MyLocalNetworkGWName -ResourceGroupName MyRGName `
		$vnetgw = Get-AzureRmVirtualNetworkGateway -Name RMGateway -ResourceGroupName MyRGName

	创建连接：
	
		New-AzureRmVirtualNetworkGatewayConnection -Name MyGWConnectionName -ResourceGroupName MyRGName `
		-Location "China North" `
		-VirtualNetworkGateway1 $vnetgw `
		-LocalNetworkGateway2 $local `
		-ConnectionType IPsec -RoutingWeight 10 -SharedKey 'abc123'

<!---HONumber=Mooncake_0822_2016-->