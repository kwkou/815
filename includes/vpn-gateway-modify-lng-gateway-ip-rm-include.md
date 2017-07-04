### <a name="gwipnoconnection"></a> 修改本地网关的“GatewayIpAddress”- 无网关连接

如果要连接的 VPN 设备已更改其公共 IP 地址，则需根据该更改修改本地网关。 请使用此示例修改没有网关连接的本地网关。

修改此值时，还可同时修改地址前缀。 请务必使用本地网关的现有名称来覆盖当前设置。 如果使用其他名称，请创建一个新的本地网关，而不是覆盖现有本地网关。

    New-AzureRmLocalNetworkGateway -Name MyLocalNetworkGWName `
    -Location "China North" -AddressPrefix @('10.0.0.0/24','20.0.0.0/24','30.0.0.0/24') `
    -GatewayIpAddress "5.4.3.2" -ResourceGroupName MyRGName

### <a name="gwipwithconnection"></a> 修改本地网关的“GatewayIpAddress”- 现有网关连接

如果要连接的 VPN 设备已更改其公共 IP 地址，则需根据该更改修改本地网关。 如果网关连接已存在，首先需要删除该连接。 删除连接后，可修改网关 IP 地址并重新创建一个新的连接。 此外可同时修改地址前缀。 这将导致 VPN 连接中断一段时间。 修改网关 IP 地址时，不需删除 VPN 网关。 只需删除连接。

1. 删除连接。 可以使用“Get-AzureRmVirtualNetworkGatewayConnection”cmdlet 查找连接的名称。

        Remove-AzureRmVirtualNetworkGatewayConnection -Name MyGWConnectionName `
        -ResourceGroupName MyRGName

2. 修改“GatewayIpAddress”值。 此外可同时修改地址前缀。 请务必使用本地网关的现有名称来覆盖当前设置。 如果不这样做，请创建一个新的本地网关，而不是覆盖现有本地网关。

        New-AzureRmLocalNetworkGateway -Name MyLocalNetworkGWName `
        -Location "China North" -AddressPrefix @('10.0.0.0/24','20.0.0.0/24','30.0.0.0/24') `
        -GatewayIpAddress "104.40.81.124" -ResourceGroupName MyRGName

3. 创建连接。 在此示例中，我们配置 IPsec 连接类型。 重新创建连接时，请使用针对配置指定的连接类型。 有关其他连接类型，请参阅 [PowerShell cmdlet](https://msdn.microsoft.com/zh-cn/library/mt603611.aspx) 页面。  若要获取 VirtualNetworkGateway 名称，可运行“Get-AzureRmVirtualNetworkGateway”cmdlet。

    设置变量。

        $local = Get-AzureRMLocalNetworkGateway -Name MyLocalNetworkGWName -ResourceGroupName MyRGName `
        $vnetgw = Get-AzureRmVirtualNetworkGateway -Name RMGateway -ResourceGroupName MyRGName

    创建连接。

        New-AzureRmVirtualNetworkGatewayConnection -Name MyGWConnectionName -ResourceGroupName MyRGName `
        -Location "China North" `
        -VirtualNetworkGateway1 $vnetgw `
        -LocalNetworkGateway2 $local `
        -ConnectionType IPsec -RoutingWeight 10 -SharedKey 'abc123'