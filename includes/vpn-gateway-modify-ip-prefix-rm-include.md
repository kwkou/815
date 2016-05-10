### 在未创建 VPN 网关连接时添加或删除前缀

- 若要将其他地址前缀**添加**到你已创建的但尚无 VPN 网关连接的本地网关，请使用下面的示例。

		$local = Get-AzureRmLocalNetworkGateway -Name LocalSite -ResourceGroupName testrg
		Set-AzureRmLocalNetworkGateway -LocalNetworkGateway $local -AddressPrefix @('10.0.0.0/24','20.0.0.0/24','30.0.0.0/24')


- 若要从没有 VPN 连接的本地网关**删除**地址前缀，请使用下面的示例。省去你不再需要的前缀。在此示例中，我们不再需要前缀 20.0.0.0/24（来自前面的示例），因此我们将更新本地网关并排除该前缀。

		$local = Get-AzureRmLocalNetworkGateway -Name LocalSite -ResourceGroupName testrg
		Set-AzureRmLocalNetworkGateway -LocalNetworkGateway $local -AddressPrefix @('10.0.0.0/24','30.0.0.0/24')

### 在已创建 VPN 网关连接时添加或删除前缀

如果你已创建 VPN 连接并且想要添加或删除包含在你的本地网关中的 IP 地址前缀，你将需要按顺序执行以下步骤。这将导致你的 VPN 连接中断一段时间，因为你需要删除并重建网关。但是，由于你已为该连接请求一个 IP 地址，因此不需要重新配置你的本地 VPN 路由器，除非你决定要更改你以前使用过的值。
 
1. 删除网关连接。 
2. 修改本地网关的前缀。 
3. 创建新的网关连接。 

你可以使用下面的示例作为指导原则。

	$gateway1 = Get-AzureRmVirtualNetworkGateway -Name vnetgw1 -ResourceGroupName testrg
	$local = Get-AzureRmLocalNetworkGateway -Name LocalSite -ResourceGroupName testrg

	Remove-AzureRmVirtualNetworkGatewayConnection -Name localtovon -ResourceGroupName testrg

	$local = Get-AzureRmLocalNetworkGateway -Name LocalSite -ResourceGroupName testrg
	Set-AzureRmLocalNetworkGateway -LocalNetworkGateway $local -AddressPrefix @('10.0.0.0/24','20.0.0.0/24','30.0.0.0/24')
	
	New-AzureRmVirtualNetworkGatewayConnection -Name localtovon -ResourceGroupName testrg -Location 'China North' -VirtualNetworkGateway1 $gateway1 -LocalNetworkGateway2 $local -ConnectionType IPsec -RoutingWeight 10 -SharedKey 'abc123'

<!---HONumber=Mooncake_0425_2016-->
