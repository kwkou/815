<properties
    pageTitle="在经典部署模型中使用 Azure CLI 控制路由和使用虚拟设备 | Azure"
    description="了解如何在典型部署模型中使用 Azure CLI 控制 Vnet 中的路由"
    services="virtual-network"
    documentationcenter="na"
    author="jimdial"
    manager="carmonm"
    editor=""
    tags="azure-service-management" />  

<tags
    ms.assetid="ca2b4638-8777-4d30-b972-eb790a7c804f"
    ms.service="virtual-network"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="03/15/2016"
    wacn.date="12/26/2016"
    ms.author="jdial" />  


# 使用 Azure CLI 控制路由和使用虚拟设备（经典）
> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/virtual-network-create-udr-arm-ps/)
- [Azure CLI](/documentation/articles/virtual-network-create-udr-arm-cli/)
- [模板](/documentation/articles/virtual-network-create-udr-arm-template/)
- [PowerShell（经典）](/documentation/articles/virtual-network-create-udr-classic-ps/)
- [CLI（经典）](/documentation/articles/virtual-network-create-udr-classic-cli/)

[AZURE.INCLUDE [virtual-network-create-udr-intro-include.md](../../includes/virtual-network-create-udr-intro-include.md)]

>[AZURE.IMPORTANT]在使用 Azure 资源之前，请务必了解 Azure 当前使用两种部署模型：资源管理器部署模型和经典部署模型。在使用任何 Azure 资源之前，请确保你了解[部署模型和工具](/documentation/articles/azure-classic-rm/)。可以通过单击本文顶部的选项卡来查看不同工具的文档。本文介绍经典部署模型。你还可以[在资源管理器部署模型中控制路由和使用虚拟设备](/documentation/articles/virtual-network-create-udr-arm-cli/)。

[AZURE.INCLUDE [virtual-network-create-udr-scenario-include.md](../../includes/virtual-network-create-udr-scenario-include.md)]

下面的示例 Azure CLI 命令需要一个已经基于上述方案创建的简单环境。如果你想要运行本文档中所显示的命令，首先需要构建[使用 Azure CLI 创建 VNet](/documentation/articles/virtual-networks-create-vnet-classic-cli/) 中所述的环境。

[AZURE.INCLUDE [azure-cli-prerequisites-include.md](../../includes/azure-cli-prerequisites-include.md)]

## 为前端子网创建 UDR
若要根据上述方案为前端子网创建所需的路由表和路由，请按照下面的步骤操作。

1. 运行以下命令，切换到经典模式：

        azure config mode asm

    输出：

        info:    New mode is asm

2. 运行以下命令，为前端子网创建路由表：

        azure network route-table create -n UDR-FrontEnd -l chinanorth

    输出：
   
        info:    Executing command network route-table create
        info:    Creating route table "UDR-FrontEnd"
        info:    Getting route table "UDR-FrontEnd"
        data:    Name                            : UDR-FrontEnd
        data:    Location                        : China North
        info:    network route-table create command OK
   
    参数：
   
   * **-l（或 --location）**。要创建新 NSG 所在的 Azure 区域。对于我们的方案，为 *chinanorth* 。
   * **-n（或 --name）**。新 NSG 的名称。对于我们的方案，为 *NSG-FrontEnd* 。
3. 运行以下命令，在路由表中创建路由，将流向后端子网 (192.168.2.0/24) 的所有流量发送到 **FW1** VM (192.168.0.4)：

        azure network route-table route set -r UDR-FrontEnd -n RouteToBackEnd -a 192.168.2.0/24 -t VirtualAppliance -p 192.168.0.4

    输出：
   
        info:    Executing command network route-table route set
        info:    Getting route table "UDR-FrontEnd"
        info:    Setting route "RouteToBackEnd" in a route table "UDR-FrontEnd"
        info:    network route-table route set command OK
   
    参数：
   
   * **-r（或 --route-table-name）**。要添加路由的路由表的名称。对于我们的方案，为 *UDR-FrontEnd* 。
   * **-a（或 --address-prefix）**。数据包目标子网的地址前缀。对于我们的方案，为 *192.168.2.0/24* 。
   * **-t（或 --next-hop-type）**。要将流量发送到的对象的类型。可能的值为 *VirtualAppliance* 、 *VirtualNetworkGateway* 、 *VNETLocal* 、 *Internet* 或 *None* 。
   * **-p（或 --next-hop-ip-address**）。下一个跃点的 IP 地址。对于我们的方案，为 *192.168.0.4* 。
4. 运行以下命令，将创建的路由表与 **FrontEnd** 子网关联：

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
   
   * **-t（或 --vnet-name）**。子网所在的 VNet 的名称。对于我们的方案，为 *TestVNet* 。
   * **-n（或 --subnet-name）**。将在其中添加路由表的子网的名称。对于我们的方案，为 *FrontEnd* 。

## 为后端子网创建 UDR
若要根据方案为后端子网创建所需的路由表和路由，请完成以下步骤：

1. 运行以下命令，为后端子网创建路由表：

        azure network route-table create -n UDR-BackEnd -l chinanorth

2. 运行以下命令，在路由表中创建路由，将流向前端子网 (192.168.1.0/24) 的所有流量发送到 **FW1** VM (192.168.0.4)：

        azure network route-table route set -r UDR-BackEnd -n RouteToFrontEnd -a 192.168.1.0/24 -t VirtualAppliance -p 192.168.0.4

3. 运行以下命令，将路由表与 **BackEnd** 子网关联：

        azure network vnet subnet route-table add -t TestVNet -n BackEnd -r UDR-BackEnd

<!---HONumber=Mooncake_1219_2016-->