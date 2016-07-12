<!-- ARM: tested -->

<properties 
   pageTitle="在资源管理器中使用 Azure CLI 控制路由和使用虚拟设备 | Microsoft Azure"
   description="了解如何使用 Azure CLI 控制路由和使用虚拟设备"
   services="virtual-network"
   documentationCenter="na"
   authors="telmosampaio"
   manager="carolz"
   editor=""
   tags="azure-resource-manager"
/>
<tags
	ms.service="virtual-network"
	ms.date="03/15/2016"
	wacn.date="06/29/2016"/>

#在 Azure CLI 中创建用户定义的路由 (UDR)

[AZURE.INCLUDE [virtual-network-create-udr-arm-selectors-include.md](../includes/virtual-network-create-udr-arm-selectors-include.md)]

[AZURE.INCLUDE [arm-api-version-cli](../includes/arm-api-version-cli.md)]

[AZURE.INCLUDE [virtual-network-create-udr-intro-include.md](../includes/virtual-network-create-udr-intro-include.md)]

>[AZURE.IMPORTANT]在使用 Azure 资源之前，请务必了解 Azure 当前使用两种部署模型：资源管理器部署模型和经典部署模型。在使用任何 Azure 资源之前，请确保你了解[部署模型和工具](/documentation/articles/azure-classic-rm/)。可以通过单击本文顶部的选项卡来查看不同工具的文档。这篇文章涵盖了资源管理器部署模型。你还可以[在经典部署模型中创建 UDR](/documentation/articles/virtual-network-create-udr-classic-cli/)。

[AZURE.INCLUDE [virtual-network-create-udr-scenario-include.md](../includes/virtual-network-create-udr-scenario-include.md)]

下面的示例 Azure CLI 命令需要一个已经基于上述方案创建的简单环境。如果你想要运行本文档中所显示的命令，首先通过部署[此模板](http://github.com/telmosampaio/azure-templates/tree/master/IaaS-NSG-UDR-Before)构建测试环境。下载该模板，并作必要的修改，然后通过 Azure CLI 发布。

>[AZURE.NOTE] 你从 GitHub 仓库 "azure-quickstart-templates" 中下载的模板，需要做一些修改才能适用于 Azure 中国云环境。例如，替换一些终结点 -- "blob.core.windows.net" 替换成 "blob.core.chinacloudapi.cn"，"cloudapp.azure.com" 替换成 "chinacloudapp.cn"；改掉一些不支持的 VM 映像，还有，改掉一些不支持的 VM 大小。

[AZURE.INCLUDE [azure-cli-prerequisites-include.md](../includes/azure-cli-prerequisites-include.md)]

## 为前端子网创建 UDR
若要根据上述方案为前端子网创建所需的路由表和路由，请按照下面的步骤操作。

3. 运行 **`azure network route-table create`** 命令为前端子网创建路由表。

		azure network route-table create -g TestRG -n UDR-FrontEnd -l chinanorth

	输出：

		info:    Executing command network route-table create
		info:    Looking up route table "UDR-FrontEnd"
		info:    Creating route table "UDR-FrontEnd"
		info:    Looking up route table "UDR-FrontEnd"
		data:    Id                              : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/
		routeTables/UDR-FrontEnd
		data:    Name                            : UDR-FrontEnd
		data:    Type                            : Microsoft.Network/routeTables
		data:    Location                        : chinanorth
		data:    Provisioning state              : Succeeded
		info:    network route-table create command OK

	参数：- **-g（或 --resource-group）**。要创建 NSG 所在的资源组的名称。对于我们的方案，为 *TestRG*。- **-l（或 --location）**。要创建新 NSG 所在的 Azure 区域。对于我们的方案，为 *chinanorth*。- **-n（或 --name）**。新 NSG 的名称。对于我们的方案，为 *NSG-FrontEnd*。

4. 运行 **`azure network route-table route create`** 命令，在上面创建的路由表中创建路由，以将目标至后端子网 (192.168.2.0/24) 的所有流量发送到 **FW1** VM (192.168.0.4)。

		azure network route-table route create -g TestRG -r UDR-FrontEnd -n RouteToBackEnd -a 192.168.2.0/24 -y VirtualAppliance -p 192.168.0.4

	输出：

		info:    Executing command network route-table route create
		info:    Looking up route "RouteToBackEnd" in route table "UDR-FrontEnd"
		info:    Creating route "RouteToBackEnd" in a route table "UDR-FrontEnd"
		info:    Looking up route "RouteToBackEnd" in route table "UDR-FrontEnd"
		data:    Id                              : /subscriptions/628dad04-b5d1-4f10-b3a4-dc61d88cf97c/resourceGroups/TestRG/providers/Microsoft.Network/
		routeTables/UDR-FrontEnd/routes/RouteToBackEnd
		data:    Name                            : RouteToBackEnd
		data:    Provisioning state              : Succeeded
		data:    Next hop type                   : VirtualAppliance
		data:    Next hop IP address             : 192.168.0.4
		data:    Address prefix                  : 192.168.2.0/24
		info:    network route-table route create command OK

	参数：- **-r（或 --route-table-name）**。要添加路由的路由表的名称。对于我们的方案，为 *UDR-FrontEnd*。- **-a（或 --address-prefix）**。数据包的目标子网的地址前缀。对于我们的方案，为 *192.168.2.0/24*。- **-y（或 --next-hop-type）**。要发送的对象流量的类型。可能的值为 *VirtualAppliance*、*VirtualNetworkGateway*、*VNETLocal*、*Internet* 或 *None*。- **-p（或 --next-hop-ip-address**）。下一个跃点的 IP 地址。对于我们的方案，为 *192.168.0.4*。

5. 运行 **`azure network vnet subnet set`** 命令，将上面创建的路由表与**前端**子网关联。

		azure network vnet subnet set -g TestRG -e TestVNet -n FrontEnd -r UDR-FrontEnd

	输出：

		info:    Executing command network vnet subnet set
		info:    Looking up the subnet "FrontEnd"
		info:    Looking up route table "UDR-FrontEnd"
		info:    Setting subnet "FrontEnd"
		info:    Looking up the subnet "FrontEnd"
		data:    Id                              : /subscriptions/628dad04-b5d1-4f10-b3a4-dc61d88cf97c/resourceGroups/TestRG/providers/Microsoft.Network/
		virtualNetworks/TestVNet/subnets/FrontEnd
		data:    Type                            : Microsoft.Network/virtualNetworks/subnets
		data:    ProvisioningState               : Succeeded
		data:    Name                            : FrontEnd
		data:    Address prefix                  : 192.168.1.0/24
		data:    Network security group          : [object Object]
		data:    Route Table                     : /subscriptions/628dad04-b5d1-4f10-b3a4-dc61d88cf97c/resourceGroups/TestRG/providers/Microsoft.Network/
		routeTables/UDR-FrontEnd
		data:    IP configurations:
		data:      /subscriptions/628dad04-b5d1-4f10-b3a4-dc61d88cf97c/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/NICWEB1/ipConf
		igurations/ipconfig1
		data:      /subscriptions/628dad04-b5d1-4f10-b3a4-dc61d88cf97c/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/NICWEB2/ipConf
		igurations/ipconfig1
		data:    
		info:    network vnet subnet set command OK

	参数：
	 - **-e（或 --vnet-name）**。要查找子网所在的 VNet 的名称。对于我们的方案，为 *TestVNet*。
 
## 为后端子网创建 UDR
若要根据上述方案为后端子网创建所需的路由表和路由，请按照下面的步骤操作。

1. 运行 **`azure network route-table create`** 命令为后端子网创建路由表。

		azure network route-table create -g TestRG -n UDR-BackEnd -l chinanorth

4. 运行 **`azure network route-table route create`** 命令，在上面创建的路由表中创建路由，以将目标至前端子网 (192.168.1.0/24) 的所有流量发送到 **FW1** VM (192.168.0.4)。

		azure network route-table route create -g TestRG -r UDR-BackEnd -n RouteToFrontEnd -a 192.168.1.0/24 -y VirtualAppliance -p 192.168.0.4

5. 运行 **`azure network vnet subnet set`** 命令，将上面创建的路由表与**后端**子网关联。

		azure network vnet subnet set -g TestRG -e TestVNet -n BackEnd -r UDR-BackEnd

## 在 FW1 上启用 IP 转发
若要在 **FW1** 使用的 NIC 上启用 IP 转发，请按照下面的步骤操作。

1. 运行 **`azure network nic show`** 命令，并注意**启用 IP 转发**的值。应将它设置为 *false*。

		azure network nic show -g TestRG -n NICFW1

	输出：
		
		info:    Executing command network nic show
		info:    Looking up the network interface "NICFW1"
		data:    Id                              : /subscriptions/628dad04-b5d1-4f10-b3a4-dc61d88cf97c/resourceGroups/TestRG/providers/Microsoft.Network/
		networkInterfaces/NICFW1
		data:    Name                            : NICFW1
		data:    Type                            : Microsoft.Network/networkInterfaces
		data:    Location                        : chinanorth
		data:    Provisioning state              : Succeeded
		data:    MAC address                     : 00-0D-3A-30-95-B3
		data:    Enable IP forwarding            : false
		data:    Tags                            : displayName=NetworkInterfaces - DMZ
		data:    Virtual machine                 : /subscriptions/628dad04-b5d1-4f10-b3a4-dc61d88cf97c/resourceGroups/TestRG/providers/Microsoft.Compute/
		virtualMachines/FW1
		data:    IP configurations:
		data:      Name                          : ipconfig1
		data:      Provisioning state            : Succeeded
		data:      Public IP address             : /subscriptions/628dad04-b5d1-4f10-b3a4-dc61d88cf97c/resourceGroups/TestRG/providers/Microsoft.Network/
		publicIPAddresses/PIPFW1
		data:      Private IP address            : 192.168.0.4
		data:      Private IP Allocation Method  : Static
		data:      Subnet                        : /subscriptions/628dad04-b5d1-4f10-b3a4-dc61d88cf97c/resourceGroups/TestRG/providers/Microsoft.Network/
		virtualNetworks/TestVNet/subnets/DMZ
		data:    
		info:    network nic show command OK

2. 运行 **`azure network nic set`** 命令以启用 IP 转发。

		azure network nic set -g TestRG -n NICFW1 -f true

	输出：

		info:    Executing command network nic set
		info:    Looking up the network interface "NICFW1"
		info:    Updating network interface "NICFW1"
		info:    Looking up the network interface "NICFW1"
		data:    Id                              : /subscriptions/628dad04-b5d1-4f10-b3a4-dc61d88cf97c/resourceGroups/TestRG/providers/Microsoft.Network/
		networkInterfaces/NICFW1
		data:    Name                            : NICFW1
		data:    Type                            : Microsoft.Network/networkInterfaces
		data:    Location                        : chinanorth
		data:    Provisioning state              : Succeeded
		data:    MAC address                     : 00-0D-3A-30-95-B3
		data:    Enable IP forwarding            : true
		data:    Tags                            : displayName=NetworkInterfaces - DMZ
		data:    Virtual machine                 : /subscriptions/628dad04-b5d1-4f10-b3a4-dc61d88cf97c/resourceGroups/TestRG/providers/Microsoft.Compute/
		virtualMachines/FW1
		data:    IP configurations:
		data:      Name                          : ipconfig1
		data:      Provisioning state            : Succeeded
		data:      Public IP address             : /subscriptions/628dad04-b5d1-4f10-b3a4-dc61d88cf97c/resourceGroups/TestRG/providers/Microsoft.Network/
		publicIPAddresses/PIPFW1
		data:      Private IP address            : 192.168.0.4
		data:      Private IP Allocation Method  : Static
		data:      Subnet                        : /subscriptions/628dad04-b5d1-4f10-b3a4-dc61d88cf97c/resourceGroups/TestRG/providers/Microsoft.Network/
		virtualNetworks/TestVNet/subnets/DMZ
		data:    
		info:    network nic set command OK

	参数：

	- **-f（或 --enable-ip-forwarding）**。*true* 或 *false*。

<!---HONumber=79-->