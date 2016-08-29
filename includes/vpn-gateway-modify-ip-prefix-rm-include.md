### <a name="noconnection"></a>如何添加或删除前缀 - 无网关连接

- 若要将其他的地址前缀**添加**到已创建的但尚无网关连接的本地网关，请使用下面的示例。请确保将值更改为自己的值。

		$local = Get-AzureRmLocalNetworkGateway -Name MyLocalNetworkGWName -ResourceGroupName MyRGName `
		Set-AzureRmLocalNetworkGateway -LocalNetworkGateway $local `
		-AddressPrefix @('10.0.0.0/24','20.0.0.0/24','30.0.0.0/24')

- 若要从没有 VPN 连接的本地网关**删除**地址前缀，请使用下面的示例。省去你不再需要的前缀。在此示例中，我们不再需要前缀 20.0.0.0/24（来自前面的示例），因此我们将更新本地网关并排除该前缀。

		$local = Get-AzureRmLocalNetworkGateway -Name MyLocalNetworkGWName -ResourceGroupName MyRGName `
		Set-AzureRmLocalNetworkGateway -LocalNetworkGateway $local `
		-AddressPrefix @('10.0.0.0/24','30.0.0.0/24')

### <a name="withconnection"></a>如何添加或删除前缀 - 现有网关连接

如果已创建了网关连接并且想要添加或删除包含在您本地网关中的 IP 地址前缀，将需要按顺序执行以下步骤。这将导致 VPN 连接中断一段时间。更新前缀时，需要先删除该连接、修改前缀，然后创建一个新的连接。在以下示例中，请确保将值更改为自己的值。

>[AZURE.IMPORTANT] 不要删除 VPN 网关。如果将其删除，则必须返回执行相应的步骤以重新创建它，并使用新设置重新配置本地路由器。
 
1. 删除连接。

		Remove-AzureRmVirtualNetworkGatewayConnection -Name MyGWConnectionName -ResourceGroupName MyRGName

2. 修改本地网关的地址前缀。

	设置 LocalNetworkGateway 的变量。

		$local = Get-AzureRmLocalNetworkGateway -Name MyLocalNetworkGWName -ResourceGroupName MyRGName

	修改前缀。

		Set-AzureRmLocalNetworkGateway -LocalNetworkGateway $local `
		-AddressPrefix @('10.0.0.0/24','20.0.0.0/24','30.0.0.0/24')

4. 创建连接。在此示例中，我们将配置 IPsec 连接类型。重新创建连接时，请使用针对配置指定的连接类型。有关其他连接类型，请参阅 [PowerShell cmdlet](https://msdn.microsoft.com/zh-cn/library/mt603611.aspx) 页面。

 	设置 VirtualNetworkGateway 的变量。

		$gateway1 = Get-AzureRmVirtualNetworkGateway -Name RMGateway  -ResourceGroupName MyRGName

	创建连接。请注意，此示例使用先前步骤中设置的 $local 变量。


		New-AzureRmVirtualNetworkGatewayConnection -Name MyGWConnectionName `
		-ResourceGroupName MyRGName -Location 'China North' `
		-VirtualNetworkGateway1 $gateway1 -LocalNetworkGateway2 $local `
		-ConnectionType IPsec `
		-RoutingWeight 10 -SharedKey 'abc123'

<!---HONumber=Mooncake_0822_2016-->