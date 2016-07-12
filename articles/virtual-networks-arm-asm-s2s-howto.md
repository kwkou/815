<!-- ARM: tested -->

<properties 
   pageTitle="如何在 Azure 中将经典 VNet 连接到 ARM VNet"
   description="了解如何在经典 VNet 和新 VNet 之间创建 VPN 连接"
   services="virtual-network"
   documentationCenter="na"
   authors="telmosampaio"
   manager="carmonm"
   editor="tysonn" />
<tags
	ms.service="virtual-network"
	ms.date="03/22/2016"
	wacn.date="06/06/2016"/>

# 将经典 VNet 连接到新 VNet

[AZURE.INCLUDE [arm-api-version-powershell](../includes/arm-api-version-powershell.md)]

Azure 当前有两种管理模式：Azure 服务管理（称之为经典）和 Azure 资源管理器 (ARM)。如果 Azure 已经使用了一段时间，则你的 Azure VM 和实例角色可能是在经典 VNet 上运行。而较新的 VM 和角色实例可能是在 ARM 中创建的 VNet 上运行。

在此类情况下，你需确保新的基础结构能够与经典资源进行通信。为此，可以在两个 VNet 之间创建一个 VPN 连接。

在本文中，你将了解如何在经典 VNet 和 ARM VNet 之间创建站点到站点 (S2S) 的 VPN 连接。

>[AZURE.NOTE] 本文假设你已拥有经典 VNet 和 ARM VNet，并且已熟悉了解如何为经典 VNet 配置 S2S VPN 连接。有关经典 VNet 与 ARM VNet 之间 S2S VPN 连接的详细端到端解决方案，请访问[解决方案指南 - 使用 S2S VPN 将经典 VNet 连接到 ARM VNet](/documentation/articles/virtual-networks-arm-asm-s2s/)。

你会看到使用以下 Azure 网关在经典 VNet 与 ARM VNet 之间创建 S2S VPN 连接所需完成的任务概述。

1 - [为经典 VNet 创建 VPN 网关](#Step-1:-Create-a-VPN-gateway-for-the-classic-VNet)

2 - [为 ARM VNet 创建 VPN 网关](#Step-2:-Create-a-VPN-gateway-for-the-ARM-VNet)

3 - [在网关之间创建连接](#Step-3:-Create-a-connection-between-the-gateways)

##<a name="Step-1:-Create-a-VPN-gateway-for-the-classic-VNet"></a> 步骤 1：为经典 VNet 创建 VPN 网关

若要为经典 VNet 创建 VPN 网关，请遵循以下说明。

1. 从 https://manage.windowsazure.cn 打开经典管理门户，必要时输入你的凭据。
2. 在屏幕的左下角，单击“新建”按钮并单击“网络服务”，然后单击“虚拟网络”，再单击“添加本地网络”。
3. 在“指定你的本地网络详细信息”窗口中，键入要连接到的 ARM VNet 的名称，然后单击窗口右下角的箭头按钮。
3. 在地址空间“起始 IP”文本框中，键入要连接到的 ARM VNet 的网络前缀。 
4. 在“CIDR (地址计数器)”下拉列表中，选择要连接到的 ARM VNet 所使用的 CIDR 块的网络部分中使用的位数。
5. 在“VPN 设备 IP 地址(可选)”中，键入任何有效的公有 IP 地址。我们稍后将更改此 IP 地址。然后，单击屏幕右下角的复选标记按钮。下图显示了此页面的示例设置。

	![本地网络设置](./media/virtual-networks-arm-asm-s2s-howto/figurex1.png)

5. 在“网络”页面上，单击“虚拟网络”并单击你的经典 VNet，然后单击“配置”。
6. 在“站点到站点连接”下，选中“连接到本地网络”复选框。
7. 从“本地网络”下拉菜单中的可用网络列表中选择你在步骤 4 中创建的本地网络，然后单击“保存”。
8. 设置保存后，单击“仪表板”，然后在页面底部单击“创建网关”，再单击“动态路由”，最后单击“是”。
9. 等待网关创建完成后，复制其公有 IP 地址。在 ARM VNet 中设置网关时，将需要该地址。

##<a name="Step-2:-Create-a-VPN-gateway-for-the-ARM-VNet"></a> 步骤 2：为 ARM VNet 创建 VPN 网关

若要为 ARM VNet 创建 VPN 网关，请遵循以下说明。

1. 从 PowerShell 控制台中，运行以下命令以创建本地网络。本地网络必须使用你要连接到的经典 VNet 的 CIDR 块，以及在上述步骤 1 中创建的网关的公有 IP 地址。

		New-AzureRmLocalNetworkGateway -Name VNetClassicNetwork `
			-Location "China East" -AddressPrefix "10.0.0.0/20" `
			-GatewayIpAddress "168.62.190.190" -ResourceGroupName RG1

3. 通过运行以下命令，创建网关的公有 IP 地址。

		$ipaddress = New-AzureRmPublicIpAddress -Name gatewaypubIP`
			-ResourceGroupName RG1 -Location "China East" `
			-AllocationMethod Dynamic

4. 通过运行以下命令，检索网关的子网。

		$subnet = Get-AzureRmVirtualNetworkSubnetConfig -Name GatewaySubnet `
			-VirtualNetwork (Get-AzureVirtualNetwork -Name VNetARM -ResourceGroupName RG1) 

	>[AZURE.IMPORTANT] 网关子网必须已经存在，其必须命名为 GatewaySubnet。

5. 通过运行以下命令，为网关创建 IP 配置对象。注意网关子网的 ID。该子网必须存在于 VNet 中。

		$ipconfig = New-AzureRmVirtualNetworkGatewayIpConfig `
			-Name ipconfig -PrivateIpAddress 10.1.2.4 `
			-SubnetId $subnet.id -PublicIpAddressId $ipaddress.id

	>[AZURE.IMPORTANT] “SubnetId” 和 “PublicIpAddressId” 参数必须分别从子网和 IP 地址对象传递 ID 属性。不能使用简单字符串。
	
5. 通过运行以下命令，创建 ARM VNet 网关。

		New-AzureRmVirtualNetworkGateway -Name v1v2Gateway -ResourceGroupName RG1 `
			-Location "China East" -GatewayType Vpn -IpConfigurations $ipconfig `
			-EnableBgp $false -VpnType RouteBased

6. VPN 网关创建完成后，运行以下命令以检索其公有 IP 地址。复制 IP 地址，为经典 VNet 配置本地网络时需要该地址。

		Get-AzureRmPublicIpAddress -Name gatewaypubIP -ResourceGroupName RG1

##<a name="Step-3:-Create-a-connection-between-the-gateways"></a> 步骤 3：在网关之间创建连接

1. 从 https://manage.windowsazure.cn 打开经典管理门户，必要时输入你的凭据。
2. 在经典管理门户中，向下滚动并单击“网络”，然后单击“本地网络”，单击要连接到的 ARM VNet，然后单击“编辑”按钮。
3. 在“VPN 设备 IP 地址(可选)”中，键入上述步骤 2 中的 ARM VNet 网关检索的 IP 地址，然后单击右上角的向右箭头，再单击复选标记按钮。
4. 在 PowerShell 控制台中，运行以下命令以设置共享密钥。确保将 VNet 的名称更改为你自己的 VNet 名称。

		Set-AzureVNetGatewayKey -VNetName VNetClassic `
			-LocalNetworkSiteName VNetARM -SharedKey abc123

7. 通过运行以下命令，创建 VPN 连接。

		$vnet01gateway = Get-AzureRmLocalNetworkGateway -Name VNetClassic -ResourceGroupName RG1
		$vnet02gateway = Get-AzureRmVirtualNetworkGateway -Name v1v2Gateway -ResourceGroupName RG1
		
		New-AzureRmVirtualNetworkGatewayConnection -Name arm-asm-s2s-connection `
			-ResourceGroupName RG1 -Location "China East" -VirtualNetworkGateway1 $vnet02gateway `
			-LocalNetworkGateway2 $vnet01gateway -ConnectionType IPsec `
			-RoutingWeight 10 -SharedKey 'abc123'

## 后续步骤

- 创建[使用 S2S VPN 以将经典 VNet 连接到 ARM VNet 的端到端解决方案](/documentation/articles/virtual-networks-arm-asm-s2s/)。
<!---HONumber=Mooncake_0516_2016-->