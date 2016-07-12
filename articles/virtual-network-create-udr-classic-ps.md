<properties 
   pageTitle="在经典部署模型中使用 PowerShell 控制路由和使用虚拟设备 | Azure"
   description="了解如何在典型部署模型中使用 PowerShell 控制 Vnet 中的路由"
   services="virtual-network"
   documentationCenter="na"
   authors="telmosampaio"
   manager="carmonm"
   editor=""
   tags="azure-service-management"
/>
<tags
	ms.service="virtual-network"
	ms.date="02/02/2016"
	wacn.date="03/17/2016"/>

#使用 PowerShell 控制路由和使用虚拟设备（经典）

[AZURE.INCLUDE [virtual-network-create-udr-classic-selectors-include.md](../includes/virtual-network-create-udr-classic-selectors-include.md)]

[AZURE.INCLUDE [virtual-network-create-udr-intro-include.md](../includes/virtual-network-create-udr-intro-include.md)]

>[AZURE.IMPORTANT]在使用 Azure 资源之前，请务必了解 Azure 当前使用两种部署模型：资源管理器部署模型和经典部署模型。在使用任何 Azure 资源之前，请确保你了解[部署模型和工具](/documentation/articles/azure-classic-rm/)。可以通过单击本文顶部的选项卡来查看不同工具的文档。本文介绍经典部署模型。你还可以[在资源管理器部署模型中创建 UDR](/documentation/articles/virtual-network-create-udr-arm-ps/)。

[AZURE.INCLUDE [virtual-network-create-udr-scenario-include.md](../includes/virtual-network-create-udr-scenario-include.md)]

下面的示例 Azure PowerShell 命令需要一个已经基于上述方案创建的简单环境。如果你想要运行本文档中所显示的命令，首先需要构建[使用 PowerShell 创建 VNet](/documentation/articles/virtual-networks-create-vnet-classic-netcfg-ps/) 中所述的环境。

[AZURE.INCLUDE [azure-ps-prerequisites-include.md](../includes/azure-ps-prerequisites-include.md)]

## 为前端子网创建 UDR
若要根据上述方案为前端子网创建所需的路由表和路由，请按照下面的步骤操作。

3. 运行 **`New-AzureRouteTable`** cmdlet 为前端子网创建路由表。

		New-AzureRouteTable -Name UDR-FrontEnd `
			-Location uswest `
			-Label "Route table for front end subnet"

	输出：

		Name         Location   Label                          
		----         --------   -----                          
		UDR-FrontEnd China North    Route table for front end subnet

4. 运行 **`Set-AzureRoute`** cmdlet，在上面创建的路由表中创建路由，以将目标至后端子网 (192.168.2.0/24) 的所有流量发送到 **FW1** VM (192.168.0.4)。
	
		Get-AzureRouteTable UDR-FrontEnd `
			|Set-AzureRoute -RouteName RouteToBackEnd -AddressPrefix 192.168.2.0/24 `
			-NextHopType VirtualAppliance `
			-NextHopIpAddress 192.168.0.4

	输出：

		Name     : UDR-FrontEnd
		Location : China North
		Label    : Route table for frontend subnet
		Routes   : 
		           Name                 Address Prefix    Next hop type        Next hop IP address
		           ----                 --------------    -------------        -------------------
		           RouteToBackEnd       192.168.2.0/24    VirtualAppliance     192.168.0.4  

5. 运行 **`Set-AzureSubnetRouteTable`** cmdlet 将上面创建的路由表与**前端**子网关联。

		Set-AzureSubnetRouteTable -VirtualNetworkName TestVNet `
			-SubnetName FrontEnd `
			-RouteTableName UDR-FrontEnd
 
## 为后端子网创建 UDR
若要根据上述方案为后端子网创建所需的路由表和路由，请按照下面的步骤操作。

3. 运行 **`New-AzureRouteTable`** cmdlet 为后端子网创建路由表。

		New-AzureRouteTable -Name UDR-BackEnd `
			-Location uswest `
			-Label "Route table for back end subnet"

4. 运行 **`Set-AzureRoute`** cmdlet，在上面创建的路由表中创建路由，以将目标至前端子网 (192.168.1.0/24) 的所有流量发送到 **FW1** VM (192.168.0.4)。

		Get-AzureRouteTable UDR-BackEnd `
			|Set-AzureRoute -RouteName RouteToFrontEnd -AddressPrefix 192.168.1.0/24 `
			-NextHopType VirtualAppliance `
			-NextHopIpAddress 192.168.0.4

5. 运行 **`Set-AzureSubnetRouteTable`** cmdlet 将上面创建的路由表与**后端**子网关联。

		Set-AzureSubnetRouteTable -VirtualNetworkName TestVNet `
			-SubnetName FrontEnd `
			-RouteTableName UDR-FrontEnd

## 在 FW1 VM 上启用 IP 转发
若要在 FW1 VM 虚拟机中启用 IP 转发，请按照下面的步骤操作。

1. 运行 **`Get-AzureIPForwarding`** cmdlet 检查 IP 转发的状态

		Get-AzureVM -Name FW1 -ServiceName TestRGFW `
			| Get-AzureIPForwarding

	输出：

		Disabled

2. 运行 **`Set-AzureIPForwarding`** 命令以对 *FW1* VM 启用 IP 转发。

		Get-AzureVM -Name FW1 -ServiceName TestRGFW `
			| Set-AzureIPForwarding -Enable

<!---HONumber=Mooncake_0307_2016-->