### 在未创建 VPN 网关连接时添加或删除前缀

- 若要将其他地址前缀**添加**到你已创建的但尚无 VPN 网关连接的本地网关，请使用下面的示例。

		$local = Get-AzureRmLocalNetworkGateway -Name LocalSite -ResourceGroupName testrg
		Set-AzureRmLocalNetworkGateway -LocalNetworkGateway $local -AddressPrefix @('10.0.0.0/24','20.0.0.0/24','30.0.0.0/24')


- 若要从没有 VPN 连接的本地网关**删除**地址前缀，请使用下面的示例。省去你不再需要的前缀。在此示例中，我们不再需要前缀 20.0.0.0/24（来自前面的示例），因此我们将更新本地网关并排除该前缀。

		$local = Get-AzureRmLocalNetworkGateway -Name LocalSite -ResourceGroupName testrg
		Set-AzureRmLocalNetworkGateway -LocalNetworkGateway $local -AddressPrefix @('10.0.0.0/24','30.0.0.0/24')

### 在已创建 VPN 网关连接时添加或删除前缀

如果你已创建 VPN 连接并且想要添加或删除包含在你的本地网关中的 IP 地址前缀，你将需要按顺序执行以下步骤。这将导致 VPN 连接中断一段时间。

>[AZURE.IMPORTANT] 不要删除 VPN 网关。如果你这样做，则将不得不返回执行相应的步骤以重新创建它，并使用新设置重新配置本地路由器。
 
1. 删除 IPsec 连接。 
2. 修改本地网关的前缀。 
3. 创建新的 IPsec 连接。 

你可以使用下面的示例作为指导原则。

	$gateway1 = Get-AzureRmVirtualNetworkGateway -Name vnetgw1 -ResourceGroupName testrg
	$local = Get-AzureRmLocalNetworkGateway -Name LocalSite -ResourceGroupName testrg

	Remove-AzureRmVirtualNetworkGatewayConnection -Name localtovon -ResourceGroupName testrg

	$local = Get-AzureRmLocalNetworkGateway -Name LocalSite -ResourceGroupName testrg
	Set-AzureRmLocalNetworkGateway -LocalNetworkGateway $local -AddressPrefix @('10.0.0.0/24','20.0.0.0/24','30.0.0.0/24')
	
	New-AzureRmVirtualNetworkGatewayConnection -Name localtovon -ResourceGroupName testrg -Location 'China North' -VirtualNetworkGateway1 $gateway1 -LocalNetworkGateway2 $local -ConnectionType IPsec -RoutingWeight 10 -SharedKey 'abc123'

<!---HONumber=Mooncake_0613_2016-->
