<properties
    pageTitle="使用 Azure CLI 2.0 创建虚拟网络 | Azure"
    description="了解如何使用 Azure CLI 2.0 创建虚拟网络 | Resource Manager"
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
    wacn.date="03/24/2017"
    ms.author="jdial" />  


# 使用 Azure CLI 创建虚拟网络

[AZURE.INCLUDE [virtual-networks-create-vnet-intro](../../includes/virtual-networks-create-vnet-intro-include.md)]

Azure 有两个部署模型：Azure Resource Manager 和经典模型。Azure 建议通过 Resource Manager 部署模型创建资源。若要深入了解这两个模型之间的差异，请阅读[了解 Azure 部署模型](/documentation/articles/resource-manager-deployment-model/)一文。

## 用于完成任务的 CLI 版本
可使用以下 CLI 版本之一完成任务：

- [Azure CLI 1.0](/documentation/articles/virtual-networks-create-vnet-arm-cli-nodejs/)：用于经典部署模型和资源管理部署模型的 CLI
- [Azure CLI 2.0（预览版）](#create-a-virtual-network)- 用于资源管理部署模型（本文）的下一代 CLI

    [AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]
 
    还可以使用其他工具通过 Resource Manager 创建 VNet，或通过从以下列表中选择不同的选项使用经典部署模型创建 VNet：
> [AZURE.SELECTOR]
- [门户](/documentation/articles/virtual-networks-create-vnet-arm-pportal/)
- [PowerShell](/documentation/articles/virtual-networks-create-vnet-arm-ps/)
- [CLI](/documentation/articles/virtual-networks-create-vnet-arm-cli/)
- [门户（经典）](/documentation/articles/virtual-networks-create-vnet-classic-pportal/)
- [PowerShell（经典）](/documentation/articles/virtual-networks-create-vnet-classic-netcfg-ps/)
- [CLI（经典）](/documentation/articles/virtual-networks-create-vnet-classic-cli/)

[AZURE.INCLUDE [virtual-networks-create-vnet-scenario-include](../../includes/virtual-networks-create-vnet-scenario-include.md)]

## <a name="create-a-virtual-network"></a> 创建虚拟网络

若要使用 Azure CLI 2.0 创建虚拟网络，请完成以下步骤：

1. 安装和配置最新的 [Azure CLI 2.0（预览版）](https://docs.microsoft.com/cli/azure/install-az-cli2)并使用 [az login](https://docs.microsoft.com/cli/azure/#login) 登录到 Azure 帐户。

2. 使用 [az group create](https://docs.microsoft.com/cli/azure/group#create) 命令以及 `--name` 和 `--location` 参数为 VNet 创建资源组：

        az group create --name myVNet --location chinaeast

3. 创建 VNet 和子网：

        az network vnet create \
            --name TestVNet \
            --resource-group myVNet \
            --location chinaeast \
            --address-prefix 192.168.0.0/16 \
            --subnet-name FrontEnd \
            --subnet-prefix 192.168.1.0/24

    预期输出：

        {
            "newVNet": {
                "addressSpace": {
                "addressPrefixes": [
                    "192.168.0.0/16"
                ]
                },
                "dhcpOptions": {
                "dnsServers": []
                },
                "provisioningState": "Succeeded",
                "resourceGuid": "<guid>",
                "subnets": [
                {
                    "etag": "W/"<guid>"",
                    "id": "/subscriptions/<guid>/resourceGroups/myVNet/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd",
                    "name": "FrontEnd",
                    "properties": {
                    "addressPrefix": "192.168.1.0/24",
                    "provisioningState": "Succeeded"
                    },
                    "resourceGroup": "myVNet"
                }
                ]
            }
        }

    使用的参数：

    - `--name TestVNet`：要创建的 VNet 的名称。
    - `--resource-group myVNet`：# 资源组名称，用于控制资源。
    - `--location chinaeast`：要部署到的位置。
    - `--address-prefix 192.168.0.0/16`：地址前缀和块。
    - `--subnet-name FrontEnd`：子网的名称。
    - `--subnet-prefix 192.168.1.0/24`：地址前缀和块。

    若要列出在下一命令中使用的基本信息，可以使用[查询筛选器](https://docs.microsoft.com/cli/azure/query-az-cli2)查询 VNet：

        az network vnet list --query '[?name==`TestVNet`].{Where:location,Name:name,Group:resourceGroup}' -o table

    这将生成以下输出：

        Where      Name      Group

        chinaeast  TestVNet  myVNet

4. 创建子网：

        az network vnet subnet create \
            --address-prefix 192.168.2.0/24 \
            --name BackEnd \
            --resource-group myVNet \
            --vnet-name TestVNet

    预期输出：

        {
        "addressPrefix": "192.168.2.0/24",
        "etag": "W/"<guid> "",
        "id": "/subscriptions/<guid>/resourceGroups/myVNet/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/BackEnd",
        "ipConfigurations": null,
        "name": "BackEnd",
        "networkSecurityGroup": null,
        "provisioningState": "Succeeded",
        "resourceGroup": "myVNet",
        "resourceNavigationLinks": null,
        "routeTable": null
        }

    使用的参数：

    - `--address-prefix 192.168.2.0/24`：子网 CIDR 块。
    - `--name BackEnd`：新子网的名称。
    - `--resource-group myVNet`：资源组。
    - `--vnet-name TestVNet`：主控 VNet 的名称。

5. 查询新 VNet 的属性：

        az network vnet show \
        -g myVNET \
        -n TestVNet \
        --query '{Name:name,Where:location,Group:resourceGroup,Status:provisioningState,SubnetCount:subnets | length(@)}' \
        -o table

    预期输出：
   
        Name      Where      Group    Status       SubnetCount

        TestVNet  chinaeast  myVNet   Succeeded              2

6. 查询子网的属性：

        az network vnet subnet list \
        -g myvnet \
        --vnet-name testvnet \
        --query '[].{Name:name,CIDR:addressPrefix,Status:provisioningState}' \
        -o table

    预期输出：

        Name      CIDR            Status

        FrontEnd  192.168.1.0/24  Succeeded
        BackEnd   192.168.2.0/24  Succeeded

## 后续步骤

了解如何连接：

- 通过阅读[创建 Linux VM](/documentation/articles/virtual-machines-linux-quick-create-cli/)一文，将虚拟机 (VM) 连接到虚拟网络。可选择将 VM 连接到现有 VNet 和子网，而不按文章中的步骤创建 VNet 和子网。
- 通过阅读[连接 VNet](/documentation/articles/vpn-gateway-howto-vnet-vnet-resource-manager-portal/) 一文将虚拟网络连接到其他虚拟网络。
- 使用站点到站点虚拟专用网络 (VPN) 或 ExpressRoute 线路将虚拟网络连接到本地网络。通过阅读[使用站点到站点 VPN 将 VNet 连接到本地网络](/documentation/articles/vpn-gateway-howto-multi-site-to-site-resource-manager-portal/)和[将 VNet 链接到 ExpressRoute 线路](/documentation/articles/expressroute-howto-linkvnet-portal-resource-manager/)，了解操作方法。

<!---HONumber=Mooncake_0320_2017-->
<!--Update_Description: change to CLI 2.0-->