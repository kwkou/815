<properties
    pageTitle="管理网络安全组 — Azure CLI 2.0 | Azure"
    description="了解如何使用 Azure 命令行接口 (CLI) 2.0 管理网络安全组。"
    services="virtual-network"
    documentationcenter="na"
    author="jimdial"
    manager="timlt"
    editor=""
    tags="azure-resource-manager" />
<tags
    ms.assetid="ed17d314-07e6-4c7f-bcf1-a8a2535d7c14"
    ms.service="virtual-network"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="02/21/2017"
    wacn.date="03/31/2017"
    ms.author="jdial"
    ms.custom="H1Hack27Feb2017" />  


# 使用 Azure CLI 2.0 管理网络安全组

[AZURE.INCLUDE [virtual-network-manage-arm-selectors-include.md](../../includes/virtual-network-manage-nsg-arm-selectors-include.md)]

## 用于完成任务的 CLI 版本 

可使用以下 CLI 版本之一完成任务：

- [Azure CLI 1.0](/documentation/articles/virtual-network-manage-nsg-cli-nodejs/)：用于经典部署模型和资源管理部署模型的 CLI
- [Azure CLI 2.0](#View-existing-NSGs) - 下一代 CLI，适用于资源管理部署模型（详见本文）

[AZURE.INCLUDE [virtual-network-manage-nsg-intro-include.md](../../includes/virtual-network-manage-nsg-intro-include.md)]

> [AZURE.NOTE]
Azure 具有两种不同的部署模型，用于创建和处理资源：[Resource Manager 模型和经典模型](/documentation/articles/resource-manager-deployment-model/)。本文介绍如何使用 Resource Manager 部署模型。Azure 建议对大多数新的部署使用该模型，而不是经典部署模型。
> 

[AZURE.INCLUDE [virtual-network-manage-nsg-arm-scenario-include.md](../../includes/virtual-network-manage-nsg-arm-scenario-include.md)]

## 先决条件
如果还未安装，请安装和配置最新版 [Azure CLI 2.0](https://docs.microsoft.com/cli/azure/install-az-cli2)，并使用 [az login](https://docs.microsoft.com/cli/azure/#login) 登录到 Azure 帐户。

## <a name="View-existing-NSGs"></a> 查看现有 NSG
若要查看特定资源组中的 NSG 的列表，请使用 `-o table` 输出格式运行 [az network nsg list](https://docs.microsoft.com/cli/azure/network/nsg#list) 命令：

    az network nsg list -g RG-NSG -o table

预期输出：

    Location    Name          ProvisioningState    ResourceGroup    ResourceGuid
    ----------  ------------  -------------------  ---------------  ------------------------------------
    chinaeast   NSG-BackEnd   Succeeded            RG-NSG           <guid>
    chinaeast   NSG-FrontEnd  Succeeded            RG-NSG           <guid>

## 列出 NSG 的所有规则
若要查看名为 **NSG-FrontEnd** 的 NSG 规则，请使用 [JMESPATH 查询筛选器](https://docs.microsoft.com/cli/azure/query-az-cli2)和 `-o table` 输出格式运行 [az network nsg show](https://docs.microsoft.com/cli/azure/network/nsg#show) 命令：

        az network nsg show \
        --resource-group RG-NSG \
        --name NSG-FrontEnd \
        --query '[defaultSecurityRules[],securityRules[]][].{Name:name,Desc:description,Access:access,Direction:direction,DestPortRange:destinationPortRange,DestAddrPrefix:destinationAddressPrefix,SrcPortRange:sourcePortRange,SrcAddrPrefix:sourceAddressPrefix}' \
        -o table

预期输出：

    Name                           Desc                                                    Access    Direction    DestPortRange    DestAddrPrefix    SrcPortRange    SrcAddrPrefix
    -----------------------------  ------------------------------------------------------  --------  -----------  ---------------  ----------------  --------------  -----------------
    AllowVnetInBound               Allow inbound traffic from all VMs in VNET              Allow     Inbound      *                VirtualNetwork    *               VirtualNetwork
    AllowAzureLoadBalancerInBound  Allow inbound traffic from azure load balancer          Allow     Inbound      *                *                 *               AzureLoadBalancer
    DenyAllInBound                 Deny all inbound traffic                                Deny      Inbound      *                *                 *               *
    AllowVnetOutBound              Allow outbound traffic from all VMs to all VMs in VNET  Allow     Outbound     *                VirtualNetwork    *               VirtualNetwork
    AllowInternetOutBound          Allow outbound traffic from all VMs to Internet         Allow     Outbound     *                Internet          *               *
    DenyAllOutBound                Deny all outbound traffic                               Deny      Outbound     *                *                 *               *
    rdp-rule                                                                               Allow     Inbound      3389             *                 *               Internet
    web-rule                                                                               Allow     Inbound      80               *                 *               Internet
> [AZURE.NOTE]
还可以使用 [az network nsg rule list](https://docs.microsoft.com/cli/azure/network/nsg/rule#list) 仅列出 NSG 中的自定义规则。
>

## <a name="View-NSGs-associations"></a>查看 NSG 关联项

若要查看与 **NSG-FrontEnd** NSG 关联的资源，请运行 `az network nsg show` 命令，如下所示。

    az network nsg show -g RG-NSG -n nsg-frontend --query '[subnets,networkInterfaces]'

查找 **NetworkInterfaces** 和 **Subnets** 属性，如下所示：

    [
      [
        {
          "addressPrefix": null,
          "etag": null,
          "id": "/subscriptions/<guid>/resourceGroups/RG-NSG/providers/Microsoft.Network/virtualNetworks/TestVNET/subnets/FrontEnd",
          "ipConfigurations": null,
          "name": null,
          "networkSecurityGroup": null,
          "provisioningState": null,
          "resourceGroup": "RG-NSG",
          "resourceNavigationLinks": null,
          "routeTable": null
        }
      ],
      null
    ]

在上述示例中，NSG 不与任何网络接口 (NIC) 关联，而是与名为 **FrontEnd** 的子网关联。

## 添加规则
若要向 **NSG-FrontEnd** NSG 添加允许来自任何计算机的**入站**流量流入端口 **443** 的规则，请输入以下命令：

    az network nsg rule create  \
    --resource-group RG-NSG \
    --nsg-name NSG-FrontEnd  \
    --name allow-https \
    --description "Allow access to port 443 for HTTPS" \
    --access Allow \
    --protocol Tcp  \
    --direction Inbound \
    --priority 102 \
    --source-address-prefix "*"  \
    --source-port-range "*"  \
    --destination-address-prefix "*" \
    --destination-port-range "443"

预期输出：

    {
      "access": "Allow",
      "description": "Allow access to port 443 for HTTPS",
      "destinationAddressPrefix": "*",
      "destinationPortRange": "443",
      "direction": "Inbound",
      "etag": "W/\"<guid>\"",
      "id": "/subscriptions/<guid>/resourceGroups/RG-NSG/providers/Microsoft.Network/networkSecurityGroups/NSG-FrontEnd/securityRules/allow-https",
      "name": "allow-https",
      "priority": 102,
      "protocol": "Tcp",
      "provisioningState": "Succeeded",
      "resourceGroup": "RG-NSG",
      "sourceAddressPrefix": "*",
      "sourcePortRange": "*"
    }

## 更改规则
若要将上面创建的规则更改为仅允许来自 **Internet** 的入站流量，请运行 [az network nsg rule update](https://docs.microsoft.com/cli/azure/network/nsg/rule#update) 命令：

    az network nsg rule update \
    --resource-group RG-NSG \
    --nsg-name NSG-FrontEnd \
    --name allow-https \
    --source-address-prefix Internet

预期输出：

    {
    "access": "Allow",
    "description": "Allow access to port 443 for HTTPS",
    "destinationAddressPrefix": "*",
    "destinationPortRange": "443",
    "direction": "Inbound",
    "etag": "W/\"<guid>\"",
    "id": "/subscriptions/<guid>/resourceGroups/RG-NSG/providers/Microsoft.Network/networkSecurityGroups/NSG-FrontEnd/securityRules/allow-https",
    "name": "allow-https",
    "priority": 102,
    "protocol": "Tcp",
    "provisioningState": "Succeeded",
    "resourceGroup": "RG-NSG",
    "sourceAddressPrefix": "Internet",
    "sourcePortRange": "*"
    }

## 删除规则
若要删除上面创建的规则，请运行以下命令：

    az network nsg rule delete \
    --resource-group RG-NSG \
    --nsg-name NSG-FrontEnd \
    --name allow-https

## 将 NSG 关联到 NIC
若要将 **NSG-FrontEnd** NSG 关联到 **TestNICWeb1** NIC，请使用 [az network nic update](https://docs.microsoft.com/cli/azure/network/nic#update) 命令：

    az network nic update \
    --resource-group RG-NSG \
    --name TestNICWeb1 \
    --network-security-group NSG-FrontEnd    

预期输出：

    {
      "dnsSettings": {
        "appliedDnsServers": [],
        "dnsServers": [],
        "internalDnsNameLabel": null,
        "internalDomainNameSuffix": "k0wkaguidnqrh0ud.gx.internal.chinacloudapp.cn",
        "internalFqdn": null
      },
      "enableAcceleratedNetworking": false,
      "enableIpForwarding": false,
      "etag": "W/\"<guid>\"",
      "id": "/subscriptions/<guid>/resourceGroups/RG-NSG/providers/Microsoft.Network/networkInterfaces/TestNICWeb1",
      "ipConfigurations": [
        {
          "applicationGatewayBackendAddressPools": null,
          "etag": "W/\"<guid>\"",
          "id": "/subscriptions/<guid>/resourceGroups/RG-NSG/providers/Microsoft.Network/networkInterfaces/TestNICWeb1/ipConfigurations/ipconfig1",
          "loadBalancerBackendAddressPools": null,
          "loadBalancerInboundNatRules": null,
          "name": "ipconfig1",
          "primary": true,
          "privateIpAddress": "192.168.1.6",
          "privateIpAddressVersion": "IPv4",
          "privateIpAllocationMethod": "Static",
          "provisioningState": "Succeeded",
          "publicIpAddress": null,
          "resourceGroup": "RG-NSG",
          "subnet": {
            "addressPrefix": null,
            "etag": null,
            "id": "/subscriptions/<guid>/resourceGroups/RG-NSG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd",
            "ipConfigurations": null,
            "name": null,
            "networkSecurityGroup": null,
            "provisioningState": null,
            "resourceGroup": "RG-NSG",
            "resourceNavigationLinks": null,
            "routeTable": null
          }
        }
      ],
      "location": "chinaeast",
      "macAddress": "00-0D-3A-91-A9-60",
      "name": "TestNICWeb1",
      "networkSecurityGroup": {
        "defaultSecurityRules": null,
        "etag": null,
        "id": "/subscriptions/<guid>/resourceGroups/RG-NSG/providers/Microsoft.Network/networkSecurityGroups/NSG-FrontEnd",
        "location": null,
        "name": null,
        "networkInterfaces": null,
        "provisioningState": null,
        "resourceGroup": "RG-NSG",
        "resourceGuid": null,
        "securityRules": null,
        "subnets": null,
        "tags": null,
        "type": null
      },
      "primary": null,
      "provisioningState": "Succeeded",
      "resourceGroup": "RG-NSG",
      "resourceGuid": "<guid>",
      "tags": {},
      "type": "Microsoft.Network/networkInterfaces",
      "virtualMachine": null
    }

## <a name="Dissociate-an-NSG-from-a-NIC"></a>取消 NSG 与 NIC 之间的关联

若要从 **TestNICWeb1** NIC 取消关联 **NSG-FrontEnd** NSG，请再次运行 [az network nsg rule update](https://docs.microsoft.com/cli/azure/network/nsg/rule#update) 命令，但将 `--network-security-group` 参数替换为一个空字符串 (`""`)。

    az network nic update --resource-group RG-NSG --name TestNICWeb3 --network-security-group ""

在输出中，`networkSecurityGroup` 项设置为 null。

## <a name="Dissociate-an-NSG-from-a-subnet"></a>取消 NSG 与子网之间的关联
若要从 **FrontEnd** 子网取消关联 **NSG-FrontEnd** NSG，请再次运行 [az network nsg rule update](https://docs.microsoft.com/cli/azure/network/nsg/rule#update) 命令，但将 `--network-security-group` 参数替换为一个空字符串 (`""`)。

    az network vnet subnet update \
    --resource-group RG-NSG \
    --vnet-name testvnet \
    --name FrontEnd \
    --network-security-group ""

在输出中，`networkSecurityGroup` 项设置为 null。

## 将 NSG 关联到子网
若要再次将 **NSG-FrontEnd** NSG 关联到 **FrontEnd** 子网，请运行以下命令：

    az network vnet subnet update \
    --resource-group RG-NSG \
    --vnet-name testvnet \
    --name FrontEnd \
    --network-security-group NSG-FrontEnd

在输出中，`networkSecurityGroup` 项具有类似的值：

    "networkSecurityGroup": {
        "defaultSecurityRules": null,
        "etag": null,
        "id": "/subscriptions/0e220bf6-5caa-4e9f-8383-51f16b6c109f/resourceGroups/RG-NSG/providers/Microsoft.Network/networkSecurityGroups/NSG-FrontEnd",
        "location": null,
        "name": null,
        "networkInterfaces": null,
        "provisioningState": null,
        "resourceGroup": "RG-NSG",
        "resourceGuid": null,
        "securityRules": null,
        "subnets": null,
        "tags": null,
        "type": null
      }

## 删除 NSG
仅当 NSG 不与任何资源关联时，才能删除 NSG。若要删除 NSG，请按照以下步骤进行操作。

1. 若要检查与 NSG 关联的资源，请运行 `azure network nsg show`，如[查看 NSG 关联项](#View-NSGs-associations)中所示。
2. 如果 NSG 关联到任意 NIC，请为每个 NIC 运行 `azure network nic set`，如[取消 NSG 与 NIC 之间的关联](#Dissociate-an-NSG-from-a-NIC)中所示。
3. 如果 NSG 关联到任意子网，请为每个子网运行 `azure network vnet subnet set`，如[取消 NSG 与子网之间的关联](#Dissociate-an-NSG-from-a-subnet)中所示。
4. 若要删除 NSG，请运行以下命令：

        az network nsg delete --resource-group RG-NSG --name NSG-FrontEnd

## 后续步骤
* 为 NSG [启用日志记录](/documentation/articles/virtual-network-nsg-manage-log/)。

<!---HONumber=Mooncake_0327_2017-->
<!--Update_Description: change from CLI 1.0 to CLI 2.0-->