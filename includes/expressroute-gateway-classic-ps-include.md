你必须先创建 VNet 和网关子网，然后再处理以下任务。有关详细信息，请参阅 [Configure a Virtual Network using the classic portal](/documentation/articles/expressroute-howto-vnet-portal-classic/)（使用经典管理门户配置虚拟网络）一文。

## 添加网关

使用以下命令可创建网关。请务必将所有值替换为你自己的值。

	New-AzureVirtualNetworkGateway -VNetName "MyAzureVNET" -GatewayName "ERGateway" -GatewayType Dedicated -GatewaySKU  Standard

## 验证是否已创建网关

使用以下命令来验证是否已创建网关。此命令还将检索执行其他操作所需的网关 ID。

	Get-AzureVirtualNetworkGateway

## 重设网关大小

这里有许多[网关 SKU](/documentation/articles/expressroute-about-virtual-network-gateways/)。你可以使用以下命令随时更改网关 SKU。

>[AZURE.IMPORTANT] 此命令对 UltraPerformance 网关不起作用。若要将网关更改为 UltraPerformance 网关，首先要删除现有的 ExpressRoute 网关，然后创建新的 UltraPerformance 网关。若要将网关从 UltraPerformance 网关降级，首先要删除 UltraPerformance 网关，然后创建新网关。

	Resize-AzureVirtualNetworkGateway -GatewayId <Gateway ID> -GatewaySKU HighPerformance

## 删除网关

使用以下命令可删除网关

	Remove-AzureVirtualNetworkGateway -GatewayId <Gateway ID>
<!---HONumber=Mooncake_0509_2016-->