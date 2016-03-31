<properties 
   pageTitle="在经典部署模型中使用 Azure CLI 控制路由和使用虚拟设备 | Azure"
   description="了解如何在典型部署模型中使用 Azure CLI 控制 Vnet 中的路由"
   services="virtual-network"
   documentationCenter="na"
   authors="telmosampaio"
   manager="carolz"
   editor=""
   tags="azure-service-management"
/>
<tags
	ms.service="virtual-network"
	ms.date="12/11/2015"
	wacn.date="01/14/2016"/>

#使用 Azure CLI 控制路由和使用虚拟设备（经典）

[AZURE.INCLUDE [virtual-network-create-udr-classic-selectors-include.md](../includes/virtual-network-create-udr-classic-selectors-include.md)]

[AZURE.INCLUDE [virtual-network-create-udr-intro-include.md](../includes/virtual-network-create-udr-intro-include.md)]

[AZURE.INCLUDE [virtual-network-create-udr-scenario-include.md](../includes/virtual-network-create-udr-scenario-include.md)]

下面的示例 Azure CLI 命令需要一个已经基于上述方案创建的简单环境。如果你想要运行本文档中所显示的命令，首先需要构建[使用 Azure CLI 创建 VNet](/documentation/articles/virtual-networks-create-vnet-classic-cli) 中所述的环境。

[AZURE.INCLUDE [azure-cli-prerequisites-include.md](../includes/azure-cli-prerequisites-include.md)]

## 为前端子网创建 UDR
若要根据上述方案为前端子网创建所需的路由表和路由，请按照下面的步骤操作。

1. 运行 **`azure config mode`** 以切换到经典模式。

		azure config mode asm

	输出：

		info:    New mode is asm

3. 运行 **`azure network route-table create`** 命令为前端子网创建路由表。

		azure network route-table create -n UDR-FrontEnd -l uswest

	输出：

		info:    Executing command network route-table create
		info:    Creating route table "UDR-FrontEnd"
		info:    Getting route table "UDR-FrontEnd"
		data:    Name                            : UDR-FrontEnd
		data:    Location                        : China North
		info:    network route-table create command OK

	参数：- **-l （或 --location）**。要创建新 NSG 所在的 Azure 区域。对于我们的方案，为 *chinanorth*。- **-n（或 --name）**。新 NSG 的名称。对于我们的方案，为 *NSG-FrontEnd*。

4. 运行 **`azure network route-table route set`** 命令，在上面创建的路由表中创建路由，以将目标至后端子网 (192.168.2.0/24) 的所有流量发送到 **FW1** VM (192.168.0.4)。

		azure network route-table route set -r UDR-FrontEnd -n RouteToBackEnd -a 192.168.2.0/24 -t VirtualAppliance -p 192.168.0.4

	输出：

		info:    Executing command network route-table route set
		info:    Getting route table "UDR-FrontEnd"
		info:    Setting route "RouteToBackEnd" in a route table "UDR-FrontEnd"
		info:    network route-table route set command OK

	参数：- **-r（或 --route-table-name）**。要添加路由的路由表的名称。对于我们的方案，为 *UDR-FrontEnd*。- **-a（或 --address-prefix）**。数据包的目标子网的地址前缀。对于我们的方案，为 *192.168.2.0/24*。- **-t（或 --next-hop-type）**。要发送的对象流量的类型。可能的值为 *VirtualAppliance*、*VirtualNetworkGateway*、*VNETLocal*、*Internet* 或 *None*。- **-p（或 --next-hop-ip-address**）。下一个跃点的 IP 地址。对于我们的方案，为 *192.168.0.4*。

5. 运行 **`azure network vnet subnet route-table add`** 命令，将上面创建的路由表与**前端**子网关联。

		azure network vnet subnet route-table add -t TestVNet -n FrontEnd -r UDR-FrontEnd

	输出：

		info:    Executing command network vnet subnet route-table add
		info:    Looking up the subnet "FrontEnd"
		info:    Looking up network configuration
		info:    Looking up network gateway route tables in virtual network "TestVNet" subnet "FrontEnd"
		info:    Associating route table "UDR-FrontEnd" and subnet "FrontEnd"
		info:    Looking up network gateway route tables in virtual network "TestVNet" subnet "FrontEnd"
		data:    Route table name                : UDR-FrontEnd
		data:      Location                      : China North
		data:      Routes:
		info:    network vnet subnet route-table add command OK	

	参数：
	 - **-t（或 --vnet-name）**。要查找子网所在的 VNet 的名称。对于我们的方案，为 *TestVNet*.
	 -  - **-n（或 --subnet-name）**。将在其中添加路由表的子网的名称。对于我们的方案，为 *FrontEnd*。
 
## 为后端子网创建 UDR
若要根据上述方案为后端子网创建所需的路由表和路由，请按照下面的步骤操作。

3. 运行 **`azure network route-table create`** 命令为后端子网创建路由表。

		azure network route-table create -n UDR-BackEnd -l uswest

4. 运行 **`azure network route-table route set`** 命令，在上面创建的路由表中创建路由，以将目标至前端子网 (192.168.1.0/24) 的所有流量发送到 **FW1** VM (192.168.0.4)。

		azure network route-table route set -r UDR-BackEnd -n RouteToFrontEnd -a 192.168.1.0/24 -t VirtualAppliance -p 192.168.0.4

5. 运行 **`azure network vnet subnet route-table add`** 命令，将上面创建的路由表与**后端**子网关联。

		azure network vnet subnet route-table add -t TestVNet -n BackEnd -r UDR-BackEnd

<!---HONumber=79-->