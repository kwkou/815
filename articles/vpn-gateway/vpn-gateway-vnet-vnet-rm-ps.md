<properties
   pageTitle="使用 VPN 网关和 PowerShell 连接 Azure VNet | Azure"
   description="本文指导你使用 Azure 资源管理器和 PowerShell 将虚拟网络连接在一起。"
   services="vpn-gateway"
   documentationCenter="na"
   authors="cherylmc"
   manager="carmonm"
   editor=""
   tags="azure-resource-manager"/>  


<tags
   ms.service="vpn-gateway"
   ms.devlang="na"
   ms.topic="get-started-article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="08/31/2016"
   wacn.date="11/07/2016"
   ms.author="cherylmc"/>  


# 使用 PowerShell 为 Resource Manager 配置 VNet 到 VNet 连接

> [AZURE.SELECTOR]
- [Resource Manager - PowerShell](/documentation/articles/vpn-gateway-vnet-vnet-rm-ps/)
- [经典 - 经典管理门户](/documentation/articles/virtual-networks-configure-vnet-to-vnet-connection/)

本文逐步讲解如何使用 VPN 网关在 Resource Manager 部署模型中创建 VNet 之间的连接。虚拟网络可位于相同或不同的区域，来自相同或不同的订阅。


![v2v 示意图](./media/vpn-gateway-vnet-vnet-rm-ps/v2vrmps.png)  



### VNet 到 VNet 的部署模型和方法


[AZURE.INCLUDE [vpn-gateway-clasic-rm](../../includes/vpn-gateway-classic-rm-include.md)] 

可以通过使用多种不同工具在这两种部署模型中配置 VNet 到 VNet 的连接。当我们发布有关此配置的新文章和其他可用工具时，将会更新以下表格。有相关的文章发布时，我们会直接从该表格链接到该文章。<br><br>

[AZURE.INCLUDE [vpn-gateway-table-vnet-vnet](../../includes/vpn-gateway-table-vnet-to-vnet-include.md)] 

## 关于 VNet 到 VNet 的连接

将虚拟网络连接到虚拟网络（VNet 到 VNet）类似于将 VNet 连接到本地站点位置。这两种连接类型都使用 Azure VPN 网关来提供使用 IPsec/IKE 的安全隧道。你连接的 VNet 可位于不同的区域中。或者位于不同的订阅中。你甚至可以将 VNet 到 VNet 通信与多站点配置组合使用。这样，便可以建立将跨界连接与虚拟网络间连接相结合的网络拓扑，如下图所示：


![关于连接](./media/vpn-gateway-vnet-vnet-rm-ps/aboutconnections.png)  

 
### 为什么要连接虚拟网络？

你可能会出于以下原因而连接虚拟网络：

- **跨区域地域冗余和地域存在**
	- 你可以使用安全连接设置自己的异地复制或同步，而无需借助于面向 Internet 的终结点。
	- 使用 Azure 流量管理器和负载均衡器，可以设置支持跨多个 Azure 区域实现异地冗余的高可用性工作负荷。一个重要的示例就是对分布在多个 Azure 区域中的可用性组设置 SQL Always On。

- **具有隔离或管理边界的区域多层应用程序**
	- 在同一区域中，由于存在隔离或管理要求，可以设置具有多个虚拟网络的多层应用程序，这些虚拟网络相互连接在一起。


### VNet 到 VNet 常见问题

[AZURE.INCLUDE [vpn-gateway-vnet-vnet-faq](../../includes/vpn-gateway-vnet-vnet-faq-include.md)] 


## 我应使用哪个步骤集？

在本文中，可以看到两组不同的步骤。一组步骤适用于[相同订阅中的 VNet](#samesub)，另一组步骤适用于[不同订阅中的 VNet](#difsub)。两组步骤之间的主要差别在于是否可以在相同的 PowerShell 会话中创建和配置所有虚拟网络和网关资源。

本文中的步骤使用在每个部分开头声明的变量。如果使用现有 VNet，请修改这些变量，反映自身环境中的设置。

![两个连接](./media/vpn-gateway-vnet-vnet-rm-ps/differentsubscription.png)  



## <a name="samesub"></a>如何连接相同订阅中的 VNet

![v2v 示意图](./media/vpn-gateway-vnet-vnet-rm-ps/v2vrmps.png)  


### 开始之前
	
在开始之前，需要安装 Azure Resource Manager PowerShell cmdlet。有关安装 PowerShell cmdlet 的详细信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。

### <a name="Step1"></a>步骤 1 - 规划 IP 地址范围


以下步骤将创建两个虚拟网络，以及它们各自的网关子网和配置。然后，在两个 VNet 之间配置 VPN 连接。必须规划好网络配置的 IP 地址范围。请记住，必须确保没有任何 VNet 范围或本地网络范围存在任何形式的重叠。

示例中使用了以下值：

**TestVNet1 的值：**

- VNet 名称：TestVNet1
- 资源组：TestRG1
- 位置：中国东部
- TestVNet1：10.11.0.0/16 和 10.12.0.0/16
- FrontEnd：10.11.0.0/24
- BackEnd：10.12.0.0/24
- GatewaySubnet：10.12.255.0/27
- DNS 服务器：8.8.8.8
- GatewayName：VNet1GW
- 公共 IP：VNet1GWIP
- VPNType：RouteBased
- Connection(1to4)：VNet1toVNet4
- Connection(1to5)：VNet1toVNet5
- ConnectionType：VNet2VNet


**TestVNet4 的值：**

- VNet 名称：TestVNet4
- TestVNet2：10.41.0.0/16 和 10.42.0.0/16
- FrontEnd：10.41.0.0/24
- BackEnd：10.42.0.0/24
- GatewaySubnet：10.42.255.0/27
- 资源组：TestRG4
- 位置：中国北部
- DNS 服务器：8.8.8.8
- GatewayName：VNet4GW
- 公共 IP：VNet4GWIP
- VPNType：RouteBased
- 连接：VNet4toVNet1
- ConnectionType：VNet2VNet



### <a name="Step2"></a>步骤 2 - 创建并配置 TestVNet1

1. 声明变量

	首先声明变量。此示例使用本练习中的值来声明变量。在大多数情况下，应该将这些值替换为自己的值。但是，如果执行这些步骤是为了熟悉此类型的配置，则可以使用这些变量。根据需要修改变量，然后将其复制并粘贴到 PowerShell 控制台中。

		$Sub1 = "Replace_With_Your_Subcription_Name"
		$RG1 = "TestRG1"
		$Location1 = "China East"
		$VNetName1 = "TestVNet1"
		$FESubName1 = "FrontEnd"
		$BESubName1 = "Backend"
		$GWSubName1 = "GatewaySubnet"
		$VNetPrefix11 = "10.11.0.0/16"
		$VNetPrefix12 = "10.12.0.0/16"
		$FESubPrefix1 = "10.11.0.0/24"
		$BESubPrefix1 = "10.12.0.0/24"
		$GWSubPrefix1 = "10.12.255.0/27"
		$DNS1 = "8.8.8.8"
		$GWName1 = "VNet1GW"
		$GWIPName1 = "VNet1GWIP"
		$GWIPconfName1 = "gwipconf1"
		$Connection14 = "VNet1toVNet4"
		$Connection15 = "VNet1toVNet5"

2. 连接到订阅

	切换到 PowerShell 模式，使用 Resource Manager cmdlet。打开 PowerShell 控制台并连接到你的帐户。使用以下示例帮助建立连接：

		Login-AzureRmAccount -EnvironmentName AzureChinaCloud

	检查该帐户的订阅。

		Get-AzureRmSubscription 

	指定要使用的订阅。

		Select-AzureRmSubscription -SubscriptionName $Sub1

3. 创建新的资源组

		New-AzureRmResourceGroup -Name $RG1 -Location $Location1

4. 创建 TestVNet1 的子网配置

	本示例创建一个名为 TestVNet1 的虚拟网络和三个子网：一个名为 GatewaySubnet、一个名为 FrontEnd，还有一个名为 Backend。替换值时，请务必始终将网关子网特意命名为 GatewaySubnet。如果命名为其他名称，网关创建将会失败。

	以下示例使用前面设置的变量。在本示例中，网关子网使用 /27。尽管创建的网关子网最小可为 /29，但建议至少选择 /28 或 /27，创建包含更多地址的更大子网。这样便可以留出足够多的地址，满足将来可能需要使用的其他配置。

		$fesub1 = New-AzureRmVirtualNetworkSubnetConfig -Name $FESubName1 -AddressPrefix $FESubPrefix1
		$besub1 = New-AzureRmVirtualNetworkSubnetConfig -Name $BESubName1 -AddressPrefix $BESubPrefix1
		$gwsub1 = New-AzureRmVirtualNetworkSubnetConfig -Name $GWSubName1 -AddressPrefix $GWSubPrefix1


5. 创建 TestVNet1

		New-AzureRmVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1 `
		-Location $Location1 -AddressPrefix $VNetPrefix11,$VNetPrefix12 -Subnet $fesub1,$besub1,$gwsub1

6. 请求公共 IP 地址

	请求一个公共 IP 地址，以分配给要为 VNet 创建的网关。请注意，AllocationMethod 为 Dynamic。您无法指定要使用的 IP 地址。它会动态分配到网关。

		$gwpip1 = New-AzureRmPublicIpAddress -Name $GWIPName1 -ResourceGroupName $RG1 `
		-Location $Location1 -AllocationMethod Dynamic

7. 创建网关配置

	网关配置定义要使用的子网和公共 IP 地址。使用该示例创建网关配置。

		$vnet1 = Get-AzureRmVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1
		$subnet1 = Get-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet1
		$gwipconf1 = New-AzureRmVirtualNetworkGatewayIpConfig -Name $GWIPconfName1 `
		-Subnet $subnet1 -PublicIpAddress $gwpip1

8. 为 TestVNet1 创建网关

	本步骤为 TestVNet1 创建虚拟网络网关。VNet 到 VNet 配置需要基于路由的 VPN 类型。创建网关可能需要花费一段时间（45 分钟或更久）。

		New-AzureRmVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1 `
		-Location $Location1 -IpConfigurations $gwipconf1 -GatewayType Vpn `
		-VpnType RouteBased -GatewaySku Standard

### 步骤 3 - 创建并配置 TestVNet4

配置 TestVNet1 后，请创建 TestVNet4。遵循以下步骤，并根据需要替换为自己的值。此步骤可在相同的 PowerShell 会话中完成，因为其位于相同的订阅中。

1. 声明变量

	请务必将值替换为要用于配置的值。

		$RG4 = "TestRG4"
		$Location4 = "China North"
		$VnetName4 = "TestVNet4"
		$FESubName4 = "FrontEnd"
		$BESubName4 = "Backend"
		$GWSubName4 = "GatewaySubnet"
		$VnetPrefix41 = "10.41.0.0/16"
		$VnetPrefix42 = "10.42.0.0/16"
		$FESubPrefix4 = "10.41.0.0/24"
		$BESubPrefix4 = "10.42.0.0/24"
		$GWSubPrefix4 = "10.42.255.0/27"
		$DNS4 = "8.8.8.8"
		$GWName4 = "VNet4GW"
		$GWIPName4 = "VNet4GWIP"
		$GWIPconfName4 = "gwipconf4"
		$Connection41 = "VNet4toVNet1"

	继续操作之前，请确保仍与订阅 1 保持连接。

2. 创建新的资源组

		New-AzureRmResourceGroup -Name $RG4 -Location $Location4

3. 创建 TestVNet4 的子网配置

		$fesub4 = New-AzureRmVirtualNetworkSubnetConfig -Name $FESubName4 -AddressPrefix $FESubPrefix4
		$besub4 = New-AzureRmVirtualNetworkSubnetConfig -Name $BESubName4 -AddressPrefix $BESubPrefix4
		$gwsub4 = New-AzureRmVirtualNetworkSubnetConfig -Name $GWSubName4 -AddressPrefix $GWSubPrefix4

4. 创建 TestVNet4

		New-AzureRmVirtualNetwork -Name $VnetName4 -ResourceGroupName $RG4 `
		-Location $Location4 -AddressPrefix $VnetPrefix41,$VnetPrefix42 -Subnet $fesub4,$besub4,$gwsub4

5. 请求公共 IP 地址

		$gwpip4 = New-AzureRmPublicIpAddress -Name $GWIPName4 -ResourceGroupName $RG4 `
		-Location $Location4 -AllocationMethod Dynamic

6. 创建网关配置

		$vnet4 = Get-AzureRmVirtualNetwork -Name $VnetName4 -ResourceGroupName $RG4
		$subnet4 = Get-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet4
		$gwipconf4 = New-AzureRmVirtualNetworkGatewayIpConfig -Name $GWIPconfName4 -Subnet $subnet4 -PublicIpAddress $gwpip4

7. 创建 TestVNet4 网关

		New-AzureRmVirtualNetworkGateway -Name $GWName4 -ResourceGroupName $RG4 `
		-Location $Location4 -IpConfigurations $gwipconf4 -GatewayType Vpn `
		-VpnType RouteBased -GatewaySku Standard

### 步骤 4 - 连接网关

1. 获取两个虚拟网络网关

	在本示例中，由于这两个网关位于相同的订阅中，此步骤可在相同的 PowerShell 会话中完成。

		$vnet1gw = Get-AzureRmVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1
		$vnet4gw = Get-AzureRmVirtualNetworkGateway -Name $GWName4 -ResourceGroupName $RG4


2. 创建 TestVNet1 到 TestVNet4 的连接

	本步骤创建从 TestVNet1 到 TestVNet4 的连接。示例中引用了共享密钥。你可以对共享密钥使用自己的值。共享密钥必须与两个连接匹配，这一点非常重要。创建连接可能需要简短的一段时间才能完成。

		New-AzureRmVirtualNetworkGatewayConnection -Name $Connection14 -ResourceGroupName $RG1 `
		-VirtualNetworkGateway1 $vnet1gw -VirtualNetworkGateway2 $vnet4gw -Location $Location1 `
		-ConnectionType Vnet2Vnet -SharedKey 'AzureA1b2C3'

3. 创建 TestVNet4 到 TestVNet1 的连接

	此步骤类似上面的步骤，只不过是创建 TestVNet4 到 TestVNet1 的连接。确保共享密钥匹配。

		New-AzureRmVirtualNetworkGatewayConnection -Name $Connection41 -ResourceGroupName $RG4 `
		-VirtualNetworkGateway1 $vnet4gw -VirtualNetworkGateway2 $vnet1gw -Location $Location4 `
		-ConnectionType Vnet2Vnet -SharedKey 'AzureA1b2C3'

	几分钟后，应会建立连接。

4. 验证连接。请参阅[如何验证连接](#verify)部分。


## <a name="difsub"></a>如何连接不同订阅中的 VNet


![v2v 示意图](./media/vpn-gateway-vnet-vnet-rm-ps/v2vdiffsub.png)  


在此方案中，我们将连接 TestVNet1 和 TestVNet5。TestVNet1 和 TestVNet5 位于不同的订阅中。此配置的步骤将添加一个额外的 VNet 到 VNet 连接，以便将 TestVNet1 连接到 TestVNet5。

此处的差别是，在第二个订阅的上下文中，某些设置步骤需要在不同的 PowerShell 会话中执行，尤其是当两个订阅属于不同的组织时。

以下说明延续上面所列的前述步骤。必须完成[步骤 1](#Step1) 和[步骤 2](#Step2)，创建并配置 TestVNet1 和 TestVNet1 的 VPN 网关。完成步骤 1 和步骤 2 之后，继续执行步骤 5 创建 TestVNet5。

### 步骤 5 - 验证其他 IP 地址范围

必须确保新虚拟网络的 IP 地址空间 TestVNet5 不与任何 VNet 范围或局域网网关范围重叠。

在本示例中，虚拟网络可能属于不同的组织。对于本练习，你可以对 TestVNet5 使用以下值：

**TestVNet5 的值：**

- VNet 名称：TestVNet5
- 资源组：TestRG5
- 位置：中国东部
- TestVNet5：10.51.0.0/16 和 10.52.0.0/16
- FrontEnd：10.51.0.0/24
- BackEnd：10.52.0.0/24
- GatewaySubnet：10.52.255.0.0/27
- DNS 服务器：8.8.8.8
- GatewayName：VNet5GW
- 公共 IP：VNet5GWIP
- VPNType：RouteBased
- 连接：VNet5toVNet1
- ConnectionType：VNet2VNet

**TestVNet1 的其他值：**

- 连接：VNet1toVNet5


### 步骤 6 - 创建并配置 TestVNet5

必须在新订阅的上下文中完成此步骤。此部分可由不同的组织中拥有订阅的管理员执行。

1. 声明变量

	请务必将值替换为要用于配置的值。

		$Sub5 = "Replace_With_the_New_Subcription_Name"
		$RG5 = "TestRG5"
		$Location5 = "China East"
		$VnetName5 = "TestVNet5"
		$FESubName5 = "FrontEnd"
		$BESubName5 = "Backend"
		$GWSubName5 = "GatewaySubnet"
		$VnetPrefix51 = "10.51.0.0/16"
		$VnetPrefix52 = "10.52.0.0/16"
		$FESubPrefix5 = "10.51.0.0/24"
		$BESubPrefix5 = "10.52.0.0/24"
		$GWSubPrefix5 = "10.52.255.0/27"
		$DNS5 = "8.8.8.8"
		$GWName5 = "VNet5GW"
		$GWIPName5 = "VNet5GWIP"
		$GWIPconfName5 = "gwipconf5"
		$Connection51 = "VNet5toVNet1"

2. 连接到订阅 5

	打开 PowerShell 控制台并连接到你的帐户。使用下面的示例来帮助你连接：

		Login-AzureRmAccount -EnvironmentName AzureChinaCloud

	检查该帐户的订阅。

		Get-AzureRmSubscription 

	指定要使用的订阅。

		Select-AzureRmSubscription -SubscriptionName $Sub5

3. 创建新的资源组

		New-AzureRmResourceGroup -Name $RG5 -Location $Location5

4. 创建 TestVNet4 的子网配置
	
		$fesub5 = New-AzureRmVirtualNetworkSubnetConfig -Name $FESubName5 -AddressPrefix $FESubPrefix5
		$besub5 = New-AzureRmVirtualNetworkSubnetConfig -Name $BESubName5 -AddressPrefix $BESubPrefix5
		$gwsub5 = New-AzureRmVirtualNetworkSubnetConfig -Name $GWSubName5 -AddressPrefix $GWSubPrefix5

5. 创建 TestVNet5

		New-AzureRmVirtualNetwork -Name $VnetName5 -ResourceGroupName $RG5 -Location $Location5 `
		-AddressPrefix $VnetPrefix51,$VnetPrefix52 -Subnet $fesub5,$besub5,$gwsub5

6. 请求公共 IP 地址

		$gwpip5 = New-AzureRmPublicIpAddress -Name $GWIPName5 -ResourceGroupName $RG5 `
		-Location $Location5 -AllocationMethod Dynamic


7. 创建网关配置

		$vnet5 = Get-AzureRmVirtualNetwork -Name $VnetName5 -ResourceGroupName $RG5
		$subnet5  = Get-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet5
		$gwipconf5 = New-AzureRmVirtualNetworkGatewayIpConfig -Name $GWIPconfName5 -Subnet $subnet5 -PublicIpAddress $gwpip5

8. 创建 TestVNet5 网关

		New-AzureRmVirtualNetworkGateway -Name $GWName5 -ResourceGroupName $RG5 -Location $Location5 `
		-IpConfigurations $gwipconf5 -GatewayType Vpn -VpnType RouteBased -GatewaySku Standard

### 步骤 7 - 连接网关

在本示例中，由于网关在不同的订阅中，因此我们已将此步骤分作两个 PowerShell 会话，其标记为 [订阅 1] 和 [订阅 5]。

1. **[订阅 1]** 获取订阅 1 的虚拟网络网关

	请确保登录并连接到订阅 1。

		$vnet1gw = Get-AzureRmVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1

	复制以下元素的输出，然后通过电子邮件或其他方法将其发送到订阅 5 的管理员。

		$vnet1gw.Name
		$vnet1gw.Id

	这两个元素的值类似于以下示例输出：

		PS D:> $vnet1gw.Name
		VNet1GW
		PS D:> $vnet1gw.Id
		/subscriptions/b636ca99-6f88-4df4-a7c3-2f8dc4545509/resourceGroupsTestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW

2. **[订阅 5]** 获取订阅 5 的虚拟网络网关

	请确保登录并连接到订阅 5。

		$vnet5gw = Get-AzureRmVirtualNetworkGateway -Name $GWName5 -ResourceGroupName $RG5

	复制以下元素的输出，然后通过电子邮件或其他方法将其发送到订阅 1 的管理员。

		$vnet5gw.Name
		$vnet5gw.Id

	这两个元素的值类似于以下示例输出：

		PS C:\> $vnet5gw.Name
		VNet5GW
		PS C:\> $vnet5gw.Id
		/subscriptions/66c8e4f1-ecd6-47ed-9de7-7e530de23994/resourceGroups/TestRG5/providers/Microsoft.Network/virtualNetworkGateways/VNet5GW

3. **[订阅 1]** 创建 TestVNet1 到 TestVNet5 的连接

	本步骤创建从 TestVNet1 到 TestVNet5 的连接。此处的差别在于无法直接获取 $vnet5gw，因为它位于不同的订阅中。需要使用上述步骤中从订阅 1 传递的值来创建新的 PowerShell 对象。使用下面的示例。将名称、ID 和共享密钥替换为自己的值。共享密钥必须与两个连接匹配，这一点非常重要。创建连接可能需要简短的一段时间才能完成。

	请确保连接到订阅 1。
	
		$vnet5gw = New-Object Microsoft.Azure.Commands.Network.Models.PSVirtualNetworkGateway
		$vnet5gw.Name = "VNet5GW"
		$vnet5gw.Id   = "/subscriptions/66c8e4f1-ecd6-47ed-9de7-7e530de23994/resourceGroups/TestRG5/providers/Microsoft.Network/virtualNetworkGateways/VNet5GW"
		$Connection15 = "VNet1toVNet5"
		New-AzureRmVirtualNetworkGatewayConnection -Name $Connection15 -ResourceGroupName $RG1 -VirtualNetworkGateway1 $vnet1gw -VirtualNetworkGateway2 $vnet5gw -Location $Location1 -ConnectionType Vnet2Vnet -SharedKey 'AzureA1b2C3'

4. **[订阅 5]** 创建 TestVNet5 到 TestVNet1 的连接

	此步骤类似上面的步骤，只不过是创建 TestVNet5 到 TestVNet1 的连接。针对基于从订阅 1 获取的值来创建 PowerShell 对象，该过程也适用于此处。在此步骤中，请确保共享密钥匹配。

	请确保连接到订阅 5。

		$vnet1gw = New-Object Microsoft.Azure.Commands.Network.Models.PSVirtualNetworkGateway
		$vnet1gw.Name = "VNet1GW"
		$vnet1gw.Id = "/subscriptions/b636ca99-6f88-4df4-a7c3-2f8dc4545509/resourceGroups/TestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW "
		New-AzureRmVirtualNetworkGatewayConnection -Name $Connection51 -ResourceGroupName $RG5 -VirtualNetworkGateway1 $vnet5gw -VirtualNetworkGateway2 $vnet1gw -Location $Location5 -ConnectionType Vnet2Vnet -SharedKey 'AzureA1b2C3'

## <a name="verify"></a>如何验证连接


[AZURE.INCLUDE [vpn-gateway-no-nsg-include](../../includes/vpn-gateway-no-nsg-include.md)]


[AZURE.INCLUDE [在 powershell 中验证连接](../../includes/vpn-gateway-verify-connection-ps-rm-include.md)]


## 后续步骤

- 连接完成后，即可将虚拟机添加到虚拟网络。请参阅[创建虚拟机](/documentation/articles/virtual-machines-windows-hero-tutorial/)以获取相关步骤。
- 有关 BGP 的信息，请参阅 [BGP 概述](/documentation/articles/vpn-gateway-bgp-overview/)和[如何配置 BGP](/documentation/articles/vpn-gateway-bgp-resource-manager-ps/)。

<!---HONumber=Mooncake_1031_2016-->