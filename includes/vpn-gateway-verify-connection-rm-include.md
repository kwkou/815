<!-- Ibiza portal: tested -->

### 使用 Azure 门户预览验证连接

可以在 Azure 门户预览中验证 VPN 连接，方法是导航到“虚拟网络网关”> 单击你的网关名称 >“设置”>“连接”。选择连接名称即可查看有关连接的详细信息。在以下示例中，未建立连接，因此没有数据流动。


![验证连接](./media/vpn-gateway-verify-connection-rm-include/connectionverify450.png)


### 使用 PowerShell 验证连接

也可能使用 `Get-AzureRmVirtualNetworkGatewayConnection –Debug` 来验证连接是否成功。你可以使用下面的 cmdlet 示例，配置符合自己需要的值。出现提示时，选择 *A* 以运行所有。

	Get-AzureRmVirtualNetworkGatewayConnection -Name localtovon -ResourceGroupName testrg -Debug

 cmdlet 运行完毕后，通过滚动查看该值。在下面的示例中，连接状态显示为“已连接”，且你可以看到入口和出口字节数。

	Body:
	{
	  "name": "localtovon",
	  "id":
	"/subscriptions/086cfaa0-0d1d-4b1c-9455-f8e3da2a0c7789/resourceGroups/testrg/providers/Microsoft.Network/connections/loca
	ltovon",
	  "properties": {
	    "provisioningState": "Succeeded",
	    "resourceGuid": "1c484f82-23ec-47e2-8cd8-231107450446b",
	    "virtualNetworkGateway1": {
	      "id":
	"/subscriptions/086cfaa0-0d1d-4b1c-9455-f8e3da2a0c7789/resourceGroups/testrg/providers/Microsoft.Network/virtualNetworkGa
	teways/vnetgw1"
	    },
	    "localNetworkGateway2": {
	      "id":
	"/subscriptions/086cfaa0-0d1d-4b1c-9455-f8e3da2a0c7789/resourceGroups/testrg/providers/Microsoft.Network/localNetworkGate
	ways/LocalSite"
	    },
	    "connectionType": "IPsec",
	    "routingWeight": 10,
	    "sharedKey": "abc123",
	    "connectionStatus": "Connected",
	    "ingressBytesTransferred": 33509044,
	    "egressBytesTransferred": 4142431
	  }

<!---HONumber=Mooncake_0425_2016-->
