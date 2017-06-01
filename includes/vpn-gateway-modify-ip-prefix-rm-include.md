### <a name="noconnection"></a>修改本地网关 IP 地址前缀 - 无网关连接

- 添加其他地址前缀：

        $local = Get-AzureRmLocalNetworkGateway -Name MyLocalNetworkGWName -ResourceGroupName MyRGName `
        Set-AzureRmLocalNetworkGateway -LocalNetworkGateway $local `
        -AddressPrefix @('10.0.0.0/24','20.0.0.0/24','30.0.0.0/24')

- 删除地址前缀：<br>
  省去你不再需要的前缀。 在此示例中，我们不再需要前缀 20.0.0.0/24（来自前面的示例），因此我们将更新本地网关，排除该前缀。

        $local = Get-AzureRmLocalNetworkGateway -Name MyLocalNetworkGWName -ResourceGroupName MyRGName `
        Set-AzureRmLocalNetworkGateway -LocalNetworkGateway $local `
        -AddressPrefix @('10.0.0.0/24','30.0.0.0/24')

### <a name="withconnection"></a>修改本地网关 IP 地址前缀 - 现有网关连接

如果有一个网关连接并且想要添加或删除包含在本地网关中的 IP 地址前缀，将需要按顺序执行以下步骤。 这将导致 VPN 连接中断一段时间。 修改 IP 地址前缀时，不需删除 VPN 网关。 只需删除连接。

1. 删除连接。

        Remove-AzureRmVirtualNetworkGatewayConnection -Name MyGWConnectionName -ResourceGroupName MyRGName

2. 修改本地网关的地址前缀。

    设置 LocalNetworkGateway 的变量。

        $local = Get-AzureRmLocalNetworkGateway -Name MyLocalNetworkGWName -ResourceGroupName MyRGName

    修改前缀。

        Set-AzureRmLocalNetworkGateway -LocalNetworkGateway $local `
        -AddressPrefix @('10.0.0.0/24','20.0.0.0/24','30.0.0.0/24')

3. 创建连接。 在此示例中，我们配置 IPsec 连接类型。 重新创建连接时，请使用针对配置指定的连接类型。 有关其他连接类型，请参阅 [PowerShell cmdlet](https://msdn.microsoft.com/zh-cn/library/mt603611.aspx) 页面。

    设置 VirtualNetworkGateway 的变量。

        $gateway1 = Get-AzureRmVirtualNetworkGateway -Name RMGateway    -ResourceGroupName MyRGName

    创建连接。 此示例使用在步骤 2 中设置的变量 $local。

        New-AzureRmVirtualNetworkGatewayConnection -Name MyGWConnectionName `
        -ResourceGroupName MyRGName -Location 'China North' `
        -VirtualNetworkGateway1 $gateway1 -LocalNetworkGateway2 $local `
        -ConnectionType IPsec `
        -RoutingWeight 10 -SharedKey 'abc123'