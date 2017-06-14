### <a name="to-verify-your-connection-by-using-powershell"></a>使用 PowerShell 验证连接

可以验证连接是否成功，方法是使用“Get-AzureRmVirtualNetworkGatewayConnection”cmdlet，带或不带“-Debug”。 

1. 使用以下 cmdlet 示例，配置符合自己需要的值。 如果出现提示，请选择“A”运行“所有”。 在此示例中，“-Name”是指所创建的需要测试的连接的名称。

            Get-AzureRmVirtualNetworkGatewayConnection -Name MyGWConnection -ResourceGroupName MyRG

2. cmdlet 运行完毕后，查看该值。 在以下示例中，连接状态显示为“已连接”，且可以看到入口和出口字节数。

                "connectionType": "IPsec",
                "routingWeight": 10,
                "sharedKey": "abc123",
                "connectionStatus": "Connected",
                "ingressBytesTransferred": 33509044,
                "egressBytesTransferred": 4142431

### <a name="to-verify-your-connection-by-using-the-azure-portal-preview"></a>使用 Azure 门户验证连接

在 Azure 门户中，可通过导航到该连接来查看连接状态。 有多种方法可执行此操作。 以下步骤演示导航到连接并进行验证的一种方法。

1. 在 [Azure 门户](http://portal.azure.cn)中，单击“所有资源”，然后导航到虚拟网关。
2. 在“虚拟网络网关”边栏选项卡中，单击“连接” 。 可查看每个连接的状态。
3. 单击想要验证的连接的名称，打开“概要” 。 在“概要”中，可以查看有关连接的详细信息。 成功连接后，“状态”  为“已成功”和“已连接”。

    ![验证连接](./media/vpn-gateway-verify-connection-rm-include/connectionsucceeded.png)