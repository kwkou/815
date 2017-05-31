<properties
    pageTitle="创建网络安全组 - Azure CLI 2.0 | Azure"
    description="了解如何使用 Azure CLI 2.0 创建和部署网络安全组。"
    services="virtual-network"
    documentationcenter="na"
    author="jimdial"
    manager="timlt"
    editor="tysonn"
    tags="azure-resource-manager" />
<tags
    ms.assetid="9ea82c09-f4a6-4268-88bc-fc439db40c48"
    ms.service="virtual-network"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="02/17/2017"
    wacn.date="03/31/2017"
    ms.author="jdial"
    ms.custom="H1Hack27Feb2017" />  


# 使用 Azure CLI 2.0 创建网络安全组

[AZURE.INCLUDE [virtual-networks-create-nsg-selectors-arm-include](../../includes/virtual-networks-create-nsg-selectors-arm-include.md)]

## 用于完成任务的 CLI 版本 

可使用以下 CLI 版本之一完成任务：

- [Azure CLI 1.0](/documentation/articles/virtual-networks-create-nsg-cli-nodejs/)：用于经典部署模型和资源管理部署模型的 CLI
- [Azure CLI 2.0](#Create-the-nsg-for-the-front-end-subnet) - 下一代 CLI，适用于资源管理部署模型（详见本文）

[AZURE.INCLUDE [virtual-networks-create-nsg-intro-include](../../includes/virtual-networks-create-nsg-intro-include.md)]

[AZURE.INCLUDE [virtual-networks-create-nsg-scenario-include](../../includes/virtual-networks-create-nsg-scenario-include.md)]

下面的 Azure CLI 2.0 命令示例需要一个已基于上述方案创建的简单环境。请下载模板，执行一些必要的修改，然后使用 Azure CLI 进行部署。

>[AZURE.NOTE] 必须修改从 GitHub 存储库“azure-quickstart-templates”下载的模板，以适应 Azure 中国云环境。例如，替换某些终结点（将“blob.core.windows.net”替换为“blob.core.chinacloudapi.cn”，将“cloudapp.azure.com”替换为“chinacloudapp.cn”）；更改某些不受支持的 VM 映像；更改某些不受支持的 VM 大小。

## <a name="Create-the-nsg-for-the-front-end-subnet"></a> 为`FrontEnd`子网创建 NSG

若要基于上述方案创建名为 *NSG-FrontEnd* 的 NSG，请执行下面的步骤。

1. 如果还未安装，请安装和配置最新版 [Azure CLI 2.0](https://docs.microsoft.com/cli/azure/install-az-cli2)，并使用 [az login](https://docs.microsoft.com/cli/azure/#login) 登录到 Azure 帐户。

2. 运行 [az network nsg create](https://docs.microsoft.com/cli/azure/network/nsg#create) 命令创建 NSG。

        az network nsg create \
        --resource-group testrg \
        --name NSG-FrontEnd \
        --location chinaeast 

    参数：
   
   * `--resource-group`：创建 NSG 的资源组的名称。对于我们的方案，为 *TestRG*。
   * `--location`：创建新 NSG 的 Azure 区域。对于我们的方案，为 *chinanorth*。
   * `--name`：新 NSG 的名称。对于我们的方案，为 *NSG-FrontEnd*。

    预期输出包含很多信息，例如所有默认规则的列表等。以下示例显示了采用 `table` 输出格式且使用 JMESPATH 查询筛选器的默认规则：

        az network nsg show \
        -g testrg \
        -n nsg-frontend \
        --query 'defaultSecurityRules[].{Access:access,Desc:description,DestPortRange:destinationPortRange,Direction:direction,Priority:priority}' \
        -o table

   输出：

        Access    Desc                                                    DestPortRange    Direction      Priority

        Allow     Allow inbound traffic from all VMs in VNET              *                Inbound           65000
        Allow     Allow inbound traffic from azure load balancer          *                Inbound           65001
        Deny      Deny all inbound traffic                                *                Inbound           65500
        Allow     Allow outbound traffic from all VMs to all VMs in VNET  *                Outbound          65000
        Allow     Allow outbound traffic from all VMs to Internet         *                Outbound          65001
        Deny      Deny all outbound traffic                               *                Outbound          65500

3. 通过 [az network nsg rule create](https://docs.microsoft.com/cli/azure/network/nsg/rule#create) 命令，创建允许从 Internet 访问端口 3389 (RDP) 的规则。

    > [AZURE.NOTE]
    根据所使用的命令行界面，可能需要修改以下参数中的 `*` 字符，防止在执行前展开参数。

        az network nsg rule create \
        --resource-group testrg \
        --nsg-name NSG-FrontEnd \
        --name rdp-rule \
        --access Allow \
        --protocol Tcp \
        --direction Inbound \
        --priority 100 \
        --source-address-prefix Internet \
        --source-port-range "*" \
        --destination-address-prefix "*" \
        --destination-port-range 3389

    预期输出：

        {
            "access": "Allow",
            "description": null,
            "destinationAddressPrefix": "*",
            "destinationPortRange": "3389",
            "direction": "Inbound",
            "etag": "W/\"<guid>\"",
            "id": "/subscriptions/<guid>/resourceGroups/testrg/providers/Microsoft.Network/networkSecurityGroups/NSG-FrontEnd/securityRules/rdp-rule",
            "name": "rdp-rule",
            "priority": 100,
            "protocol": "Tcp",
            "provisioningState": "Succeeded",
            "resourceGroup": "testrg",
            "sourceAddressPrefix": "Internet",
            "sourcePortRange": "*"
        }

    参数：

    * `--resource-group testrg`：要使用的资源组。请注意，该组不区分大小写。
    * `--nsg-name NSG-FrontEnd`：在其中创建规则的 NSG 的名称。
    * `--name rdp-rule`：新规则的名称。
    * `--access Allow`：规则的访问级别（拒绝或允许）。
    * `--protocol Tcp`：协议（Tcp、Udp 或 *）。
    * `--direction Inbound`：连接方向（入站或出站）。
    * `--priority 100`：规则的优先级。
    * `--source-address-prefix Internet`：CIDR 中的源地址前缀或者使用默认标记。
    * `--source-port-range "*"`：源端口或端口范围。打开连接的端口。
    * `--destination-address-prefix "*"`：CIDR 中的目标地址前缀或者使用默认标记。
    * `--destination-port-range 3389`：目标端口或端口范围。接收连接请求的端口。

4. 通过 **azure network nsg rule create** 命令，创建允许从 Internet 访问端口 80 (HTTP) 的规则。

        az network nsg rule create \
        --resource-group testrg \
        --nsg-name NSG-FrontEnd \
        --name web-rule \
        --access Allow \
        --protocol Tcp \
        --direction Inbound \
        --priority 200 \
        --source-address-prefix Internet \
        --source-port-range "*" \
        --destination-address-prefix "*" \
        --destination-port-range 80

    预期输出：

        {
            "access": "Allow",
            "description": null,
            "destinationAddressPrefix": "*",
            "destinationPortRange": "80",
            "direction": "Inbound",
            "etag": "W/\"<guid>\"",
            "id": "/subscriptions/<guid>/resourceGroups/testrg/providers/Microsoft.Network/networkSecurityGroups/NSG-FrontEnd/securityRules/web-rule",
            "name": "web-rule",
            "priority": 200,
            "protocol": "Tcp",
            "provisioningState": "Succeeded",
            "resourceGroup": "testrg",
            "sourceAddressPrefix": "Internet",
            "sourcePortRange": "*"
        }

5. 通过 [az network vnet subnet update](https://docs.microsoft.com/cli/azure/network/vnet/subnet#update) 命令，将 NSG 绑定到**前端**子网。

        az network vnet subnet update \
        --vnet-name TestVNET \
        --name FrontEnd \
        --resource-group testrg \
        --network-security-group NSG-FrontEnd

    预期输出：

        {
            "addressPrefix": "192.168.1.0/24",
            "etag": "W/\"<guid>\"",
            "id": "/subscriptions/<guid>/resourceGroups/testrg/providers/Microsoft.Network/virtualNetworks/TestVNET/subnets/FrontEnd",
            "ipConfigurations": [
                {
                "etag": null,
                "id": "/subscriptions/<guid>/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/TestNIC/ipConfigurations/ipconfig1",
                "name": null,
                "privateIpAddress": null,
                "privateIpAllocationMethod": null,
                "provisioningState": null,
                "publicIpAddress": null,
                "resourceGroup": "TestRG",
                "subnet": null
                }
            ],
            "name": "FrontEnd",
            "networkSecurityGroup": {
                "defaultSecurityRules": null,
                "etag": null,
                "id": "/subscriptions/<guid>f/resourceGroups/testrg/providers/Microsoft.Network/networkSecurityGroups/NSG-FrontEnd",
                "location": null,
                "name": null,
                "networkInterfaces": null,
                "provisioningState": null,
                "resourceGroup": "testrg",
                "resourceGuid": null,
                "securityRules": null,
                "subnets": null,
                "tags": null,
                "type": null
            },
            "provisioningState": "Succeeded",
            "resourceGroup": "testrg",
            "resourceNavigationLinks": null,
            "routeTable": null
        }

## 为`BackEnd`子网创建 NSG
若要基于上述方案创建名为 *NSG-BackEnd* 的 NSG，请执行下面的步骤。

1. 使用 **az network nsg create** 创建 `NSG-BackEnd` NSG。

        az network nsg create \
        --resource-group testrg \
        --name NSG-BackEnd \
        --location chinaeast

    如之前的步骤 2 所示，预期输出相当大，包括默认规则。
   
2. 通过 **az network nsg rule create** 命令，创建允许从`FrontEnd`子网访问端口 1433 (SQL) 的规则。

        az network nsg rule create \
        --resource-group testrg \
        --nsg-name NSG-BackEnd \
        --name sql-rule \
        --access Allow \
        --protocol Tcp \
        --direction Inbound \
        --priority 100 \
        --source-address-prefix 192.168.1.0/24 \
        --source-port-range "*" \
        --destination-address-prefix "*" \
        --destination-port-range 1433

    预期输出：

        {
        "access": "Allow",
        "description": null,
        "destinationAddressPrefix": "*",
        "destinationPortRange": "1433",
        "direction": "Inbound",
        "etag": "W/\"<guid>\"",
        "id": "/subscriptions/<guid>/resourceGroups/testrg/providers/Microsoft.Network/networkSecurityGroups/NSG-BackEnd/securityRules/sql-rule",
        "name": "sql-rule",
        "priority": 100,
        "protocol": "Tcp",
        "provisioningState": "Succeeded",
        "resourceGroup": "testrg",
        "sourceAddressPrefix": "192.168.1.0/24",
        "sourcePortRange": "*"
        }

3. 通过 **azure network nsg rule create** 命令，创建拒绝访问 Internet 的规则。

        az network nsg rule create \
        --resource-group testrg \
        --nsg-name NSG-BackEnd \
        --name web-rule \
        --access Deny \
        --protocol Tcp  \
        --direction Outbound  \
        --priority 200 \
        --source-address-prefix "*" \
        --source-port-range "*" \
        --destination-address-prefix "*" \
        --destination-port-range "*"

    预期输出：

        {
        "access": "Deny",
        "description": null,
        "destinationAddressPrefix": "*",
        "destinationPortRange": "*",
        "direction": "Outbound",
        "etag": "W/\"<guid>\"",
        "id": "/subscriptions/<guid>/resourceGroups/testrg/providers/Microsoft.Network/networkSecurityGroups/NSG-BackEnd/securityRules/web-rule",
        "name": "web-rule",
        "priority": 200,
        "protocol": "Tcp",
        "provisioningState": "Succeeded",
        "resourceGroup": "testrg",
        "sourceAddressPrefix": "*",
        "sourcePortRange": "*"
        }

4. 通过 **az network vnet subnet set** 命令，将 NSG 绑定到`BackEnd`子网。

        az network vnet subnet update \
        --vnet-name TestVNET \
        --name BackEnd \
        --resource-group testrg \
        --network-security-group NSG-BackEnd

    预期输出：

        {
        "addressPrefix": "192.168.2.0/24",
        "etag": "W/\"<guid>\"",
        "id": "/subscriptions/<guid>/resourceGroups/testrg/providers/Microsoft.Network/virtualNetworks/TestVNET/subnets/BackEnd",
        "ipConfigurations": null,
        "name": "BackEnd",
        "networkSecurityGroup": {
            "defaultSecurityRules": null,
            "etag": null,
            "id": "/subscriptions/<guid>/resourceGroups/testrg/providers/Microsoft.Network/networkSecurityGroups/NSG-BackEnd",
            "location": null,
            "name": null,
            "networkInterfaces": null,
            "provisioningState": null,
            "resourceGroup": "testrg",
            "resourceGuid": null,
            "securityRules": null,
            "subnets": null,
            "tags": null,
            "type": null
        },
        "provisioningState": "Succeeded",
        "resourceGroup": "testrg",
        "resourceNavigationLinks": null,
        "routeTable": null
        }

<!---HONumber=Mooncake_0327_2017-->
<!--Update_Description: change from CLI 1.0 to CLI 2.0-->