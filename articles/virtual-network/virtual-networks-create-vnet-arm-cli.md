<properties
    pageTitle="使用 Azure CLI 创建虚拟网络 | Azure"
    description="了解如何使用 Azure CLI 创建虚拟网络 | Resource Manager。"
    services="virtual-network"
    documentationcenter=""
    author="jimdial"
    manager="carmonm"
    editor=""
    tags="azure-resource-manager" />  

<tags
    ms.assetid="75966bcc-0056-4667-8482-6f08ca38e77a"
    ms.service="virtual-network"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="03/15/2016"
    wacn.date="12/26/2016"
    ms.author="jdial" />  


# 使用 Azure CLI 创建虚拟网络

[AZURE.INCLUDE [virtual-networks-create-vnet-intro](../../includes/virtual-networks-create-vnet-intro-include.md)]

Azure 有两个部署模型：Azure Resource Manager 和经典模型。Azure 建议通过 Resource Manager 部署模型创建资源。若要深入了解这两个模型之间的差异，请阅读[了解 Azure 部署模型](/documentation/articles/resource-manager-deployment-model/)一文。
 
本文介绍如何使用 Azure 命令行接口 (CLI) 通过 Resource Manager 部署模型创建 VNet。还可以使用其他工具通过 Resource Manager 创建 VNet，或通过从以下列表中选择不同的选项使用经典部署模型创建 VNet：
> [AZURE.SELECTOR]
- [门户](/documentation/articles/virtual-networks-create-vnet-arm-pportal/)
- [PowerShell](/documentation/articles/virtual-networks-create-vnet-arm-ps/)
- [CLI](/documentation/articles/virtual-networks-create-vnet-arm-cli/)
- [门户（经典）](/documentation/articles/virtual-networks-create-vnet-classic-pportal/)
- [PowerShell（经典）](/documentation/articles/virtual-networks-create-vnet-classic-netcfg-ps/)
- [CLI（经典）](/documentation/articles/virtual-networks-create-vnet-classic-cli/)

[AZURE.INCLUDE [virtual-networks-create-vnet-scenario-include](../../includes/virtual-networks-create-vnet-scenario-include.md)]

## 创建虚拟网络

若要使用 Azure CLI 创建虚拟网络，请完成以下步骤：

1. 按照[安装和配置 Azure CLI](/documentation/articles/xplat-cli-install/) 文章中的步骤安装和配置 Azure CLI。

2. 运行以下命令创建 VNet 和子网：

        azure network vnet create --vnet TestVNet -e 192.168.0.0 -i 16 -n FrontEnd -p 192.168.1.0 -r 24 -l "China North"

    预期输出：
   
            info:    Executing command network vnet create
            + Looking up network configuration
            + Looking up locations
            + Setting network configuration
            info:    network vnet create command OK

	使用的参数：

   * **--vnet**。要创建的 VNet 的名称。对于我们的方案，为 *TestVNet*
   * **-e（或 --address-space）**。VNet 地址空间。对于我们的方案，为 *192.168.0.0*
   * **-i（或 -cidr）**。采用 CIDR 格式的网络掩码。对于我们的方案，为 *16* 。
   * **-n（或 --subnet-name）**。第一个子网的名称。对于我们的方案，为 *FrontEnd* 。
   * **-p（或 --subnet-start-ip）**。子网或子网地址空间的起始 IP 地址。对于我们的方案，为 *192.168.1.0* 。
   * **-r（或 --subnet-cidr）**。子网的网络掩码（采用 CIDR 格式）。对于我们的方案，为 *24* 。
   * **-l（或 --location）**。要在其中创建 VNet 的 Azure 区域 。在我们的方案中为中国北部。
3. 运行以下命令创建子网：

        azure network vnet subnet create -t TestVNet -n BackEnd -a 192.168.2.0/24

    预期输出：

            info:    Executing command network vnet subnet create
            + Looking up network configuration
            + Creating subnet "BackEnd"
            + Setting network configuration
            + Looking up the subnet "BackEnd"
            + Looking up network configuration
            data:    Name                            : BackEnd
            data:    Address prefix                  : 192.168.2.0/24
            info:    network vnet subnet create command OK

	使用的参数：

   * **-t（或 --vnet-name）**。将在其中创建子网的 VNet 的名称。对于我们的方案，为 *TestVNet* 。
   * **-n（或 --name）**。新子网的名称。对于我们的方案，为 *BackEnd* 。
   * **-a（或 --address-prefix）**。子网 CIDR 块。对于我们的方案，为 *192.168.2.0/24* 。
4. 运行以下命令查看新 VNet 的属性：

        azure network vnet show

    预期输出：
   
            info:    Executing command network vnet show
            Virtual network name: TestVNet
            + Looking up the virtual network sites
            data:    Name                            : TestVNet
            data:    Location                        : China North
            data:    State                           : Created
            data:    Address space                   : 192.168.0.0/16
            data:    Subnets:
            data:      Name                          : FrontEnd
            data:      Address prefix                : 192.168.1.0/24
            data:
            data:      Name                          : BackEnd
            data:      Address prefix                : 192.168.2.0/24
            data:
            info:    network vnet show command OK

## 后续步骤

了解如何连接：

- 通过阅读[创建 Linux VM](/documentation/articles/virtual-machines-linux-quick-create-cli/)一文，将虚拟机 (VM) 连接到虚拟网络。可选择将 VM 连接到现有 VNet 和子网，而不按文章中的步骤创建 VNet 和子网。
- 通过阅读[连接 VNet](/documentation/articles/vpn-gateway-howto-vnet-vnet-resource-manager-portal/) 一文将虚拟网络连接到其他虚拟网络。
- 使用站点到站点虚拟专用网络 (VPN) 或 ExpressRoute 线路将虚拟网络连接到本地网络。通过阅读[使用站点到站点 VPN 将 VNet 连接到本地网络](/documentation/articles/vpn-gateway-howto-multi-site-to-site-resource-manager-portal/)和[将 VNet 链接到 ExpressRoute 线路](/documentation/articles/expressroute-howto-linkvnet-portal-resource-manager/)文章，了解操作方法。

<!---HONumber=Mooncake_1219_2016-->