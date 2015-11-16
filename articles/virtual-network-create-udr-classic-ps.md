<properties 
   pageTitle="在经典部署模型中使用 PowerShell 控制路由和使用虚拟设备 | Windows Azure"
   description="了解如何在典型部署模型中使用 PowerShell 控制 Vnet 中的路由"
   services="virtual-network"
   documentationCenter="na"
   authors="telmosampaio"
   manager="carolz"
   editor=""
   tags="azure-service-management"
/>
<tags
	ms.service="virtual-network"
	ms.date="10/06/2015"
	wacn.date="11/12/2015"/>

#使用 PowerShell 控制路由和使用虚拟设备（经典）

[AZURE.INCLUDE [virtual-network-create-udr-classic-selectors-include.md](../includes/virtual-network-create-udr-classic-selectors-include.md)]

[AZURE.INCLUDE [virtual-network-create-udr-intro-include.md](../includes/virtual-network-create-udr-intro-include.md)]

[AZURE.INCLUDE [azure-arm-classic-important-include](../includes/azure-arm-classic-important-include.md)]本文介绍经典部署模型。您还可以在此处输入为 ARM 输入操作。

[AZURE.INCLUDE [virtual-network-create-udr-scenario-include.md](../includes/virtual-network-create-udr-scenario-include.md)]

下面的示例 Azure CLI 命令需要一个已经基于上述方案创建的简单环境。

[AZURE.INCLUDE [azure-ps-prerequisites-include.md](../includes/azure-ps-prerequisites-include.md)]

## 为前端子网创建 UDR
若要根据上述方案为前端子网创建所需的路由表和路由，请按照下面的步骤操作。

3. 运行 **`New-AzureRouteTable`** cmdlet 为前端子网创建路由表。

		```powershell
		New-AzureRouteTable -Name UDR-FrontEnd `
			-Location uswest `
			-Label "Route table for front end subnet"
		```

	输出：

		Name         Location   Label                          
		----         --------   -----                          
		UDR-FrontEnd China North    Route table for front end subnet

4. 运行 **`Set-AzureRoute`** cmdlet，在上面创建的路由表中创建路由，以将目标至后端子网 (192.168.2.0/24) 的所有流量发送到 **FW1** VM (192.168.0.4)。
	
		```powershell
		Get-AzureRouteTable UDR-FrontEnd `
			|Set-AzureRoute -RouteName RouteToBackEnd -AddressPrefix 192.168.2.0/24 `
			-NextHopType VirtualAppliance `
			-NextHopIpAddress 192.168.0.4
		```

	输出：

		Name     : UDR-FrontEnd
		Location : China North
		Label    : Route table for frontend subnet
		Routes   : 
		           Name                 Address Prefix    Next hop type        Next hop IP address
		           ----                 --------------    -------------        -------------------
		           RouteToBackEnd       192.168.2.0/24    VirtualAppliance     192.168.0.4  

5. 运行 **`Set-AzureSubnetRouteTable`** cmdlet 将上面创建的路由表与**前端**子网关联。

		```powershell
		Set-AzureSubnetRouteTable -VirtualNetworkName TestVNet `
			-SubnetName FrontEnd `
			-RouteTableName UDR-FrontEnd
		```
 
## 为后端子网创建 UDR
若要根据上述方案为后端子网创建所需的路由表和路由，请按照下面的步骤操作。

3. 运行 **`New-AzureRouteTable`** cmdlet 为后端子网创建路由表。

		```powershell
		New-AzureRouteTable -Name UDR-BackEnd `
			-Location uswest `
			-Label "Route table for back end subnet"
		```

4. 运行 **`Set-AzureRoute`** cmdlet，在上面创建的路由表中创建路由，以将目标至前端子网 (192.168.1.0/24) 的所有流量发送到 **FW1** VM (192.168.0.4)。

		```powershell
		Get-AzureRouteTable UDR-BackEnd `
			|Set-AzureRoute -RouteName RouteToFrontEnd -AddressPrefix 192.168.1.0/24 `
			-NextHopType VirtualAppliance `
			-NextHopIpAddress 192.168.0.4
		```

5. 运行 **`Set-AzureSubnetRouteTable`** cmdlet 将上面创建的路由表与**后端**子网关联。

		```powershell
		Set-AzureSubnetRouteTable -VirtualNetworkName TestVNet `
			-SubnetName FrontEnd `
			-RouteTableName UDR-FrontEnd
		```
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

<!---HONumber=79-->