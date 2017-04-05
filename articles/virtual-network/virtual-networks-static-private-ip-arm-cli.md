<properties
    pageTitle="为 VM 配置专用 IP 地址 - Azure CLI 2.0 | Azure"
    description="了解如何使用 Azure 命令行接口 (CLI) 2.0 为虚拟机配置专用 IP 地址。"
    services="virtual-network"
    documentationcenter="na"
    author="jimdial"
    manager="timlt"
    editor="tysonn"
    tags="azure-resource-manager" />
<tags
    ms.assetid="40b03a1a-ea00-454c-b716-7574cea49ac0"
    ms.service="virtual-network"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="02/16/2017"
    wacn.date="03/31/2017"
    ms.author="jdial"
    ms.custom="H1Hack27Feb2017" />  


# 使用 Azure CLI 2.0 为虚拟机配置专用 IP 地址

[AZURE.INCLUDE [virtual-networks-static-private-ip-selectors-arm-include](../../includes/virtual-networks-static-private-ip-selectors-arm-include.md)]

## 用于完成任务的 CLI 版本 

可使用以下 CLI 版本之一完成任务：

- [Azure CLI 1.0](/documentation/articles/virtual-networks-static-private-ip-cli-nodejs/)：用于经典部署模型和资源管理部署模型的 CLI
- [Azure CLI 2.0](#specify-a-static-private-ip-address-when-creating-a-vm) - 下一代 CLI，适用于资源管理部署模型（详见本文）

[AZURE.INCLUDE [virtual-networks-static-private-ip-intro-include](../../includes/virtual-networks-static-private-ip-intro-include.md)]

[AZURE.INCLUDE [azure-arm-classic-important-include](../../includes/azure-arm-classic-important-include.md)]

本文介绍 Resource Manager 部署模型。你还可以[管理经典部署模型中的静态专用 IP 地址](/documentation/articles/virtual-networks-static-private-ip-classic-cli/)。

[AZURE.INCLUDE [virtual-networks-static-ip-scenario-include](../../includes/virtual-networks-static-ip-scenario-include.md)]

> [AZURE.NOTE]
下面的 Azure CLI 2.0 命令示例需要一个已创建好的简单环境。若要按本文档所示运行命令，首先需要构建[创建 VNet](/documentation/articles/virtual-networks-create-vnet-arm-cli/) 中所述的测试环境。

## <a name="specify-a-static-private-ip-address-when-creating-a-vm"></a> 在创建 VM 时指定静态专用 IP 地址

若要在名为 *TestVNet* 的 VNet 的 *FrontEnd* 子网中使用静态专用 IP *192.168.1.101* 创建名为 *DNS01* 的 VM，请按照以下步骤进行操作：

1. 如果还未安装，请安装和配置最新版 [Azure CLI 2.0](https://docs.microsoft.com/cli/azure/install-az-cli2)，并使用 [az login](https://docs.microsoft.com/cli/azure/#login) 登录到 Azure 帐户。

2. 通过 [azure network public-ip create](https://docs.microsoft.com/cli/azure/network/public-ip#create) 命令为该 VM 创建公共 IP。在输出后显示的列表说明了所用的参数。

    > [AZURE.NOTE]
    根据你的环境，可能需在该步骤及后续步骤中使用不同的参数值。

        az network public-ip create \
        --name TestPIP \
        --resource-group TestRG \
        --location chinaeast \
        --allocation-method Static

    预期输出：

        {
            "publicIp": {
                "idleTimeoutInMinutes": 4,
                "ipAddress": "52.176.43.167",
                "provisioningState": "Succeeded",
                "publicIPAllocationMethod": "Static",
                "resourceGuid": "79e8baa3-33ce-466a-846c-37af3c721ce1"
            }
         }

   * `--resource-group`：将在其中创建公共 IP 的资源组的名称。
   * `--name`：公共 IP 的名称。
   * `--location`：将在其中创建公共 IP 的 Azure 区域。

3. 运行 [az network nic create](https://docs.microsoft.com/cli/azure/network/nic#create) 命令，创建具有静态专用 IP 的 NIC。在输出后显示的列表说明了所用的参数。

        az network nic create \
        --resource-group TestRG \
        --name TestNIC \
        --location chinaeast \
        --subnet FrontEnd \
        --private-ip-address 192.168.1.101 \
        --vnet-name TestVNet

    预期输出：

        {
            "newNIC": {
                "dnsSettings": {
                "appliedDnsServers": [],
                "dnsServers": []
                },
                "enableIPForwarding": false,
                "ipConfigurations": [
                {
                    "etag": "W/"<guid>"",
                    "id": "/subscriptions/<guid>/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/TestNIC/ipConfigurations/ipconfig1",
                    "name": "ipconfig1",
                    "properties": {
                    "primary": true,
                    "privateIPAddress": "192.168.1.101",
                    "privateIPAllocationMethod": "Static",
                    "provisioningState": "Succeeded",
                    "subnet": {
                        "id": "/subscriptions/<guid>/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd",
                        "resourceGroup": "TestRG"
                    }
                    },
                    "resourceGroup": "TestRG"
                }
                ],
                "provisioningState": "Succeeded",
                "resourceGuid": "<guid>"
            }
        }

    参数：

    * `--private-ip-address`：NIC 的静态专用 IP 地址。
    * `--vnet-name`：要在其中创建 NIC 的 VNet 的名称。
    * `--subnet`：要在其中创建 NIC 的子网的名称。

4. 运行 [azure vm create](https://docs.microsoft.com/cli/azure/vm/nic#create) 命令，使用上面创建的公共 IP 和 NIC 创建 VM。输出后显示的列表阐释了所用参数。

        az vm create \
        --resource-group TestRG \
        --name DNS01 \
        --location chinaeast \
        --image Debian \
        --admin-username adminuser \
        --ssh-key-value ~/.ssh/id_rsa.pub \
        --nics TestNIC

    预期输出：

        {
            "fqdns": "",
            "id": "/subscriptions/<guid>/resourceGroups/TestRG/providers/Microsoft.Compute/virtualMachines/DNS01",
            "location": "chinaeast",
            "macAddress": "00-0D-3A-92-C1-66",
            "powerState": "VM running",
            "privateIpAddress": "192.168.1.101",
            "publicIpAddress": "",
            "resourceGroup": "TestRG"
        }

   基本 [az vm create](https://docs.microsoft.com/cli/azure/vm#create) 参数之外的参数。

   * `--nics`：VM 附加到的 NIC 的名称。
   
## 检索 VM 的静态专用 IP 地址信息

若要查看所创建的静态专用 IP 地址，请运行以下 Azure CLI 命令并观察 *Private IP alloc-method* 和 *Private IP address* 的值：

    az vm show -g TestRG -n DNS01 --show-details --query 'privateIps'

预期输出：

    "192.168.1.101"

若要显示该 VM 的 NIC 的具体 IP 信息，请具体查询 NIC：

    az network nic show \
    -g testrg \
    -n testnic \
    --query 'ipConfigurations[0].{PrivateAddress:privateIpAddress,IPVer:privateIpAddressVersion,IpAllocMethod:p
    rivateIpAllocationMethod,PublicAddress:publicIpAddress}'

输出类似于：

    {
        "IPVer": "IPv4",
        "IpAllocMethod": "Static",
        "PrivateAddress": "192.168.1.101",
        "PublicAddress": null
    }

## 从 VM 中删除静态专用 IP 地址

无法在用于 Resource Manager 部署的 Azure CLI 中删除 NIC 的静态专用 IP 地址。必须具备以下条件：
- 创建使用动态 IP 的新 NIC
- 在新建的 NIC 的 VM 上设置 NIC。

若要更改上述命令使用的 VM NIC，请执行以下步骤。

1. 运行 **azure network nic create** 命令，通过具有新 IP 地址的动态 IP 分配创建新 NIC。请注意，由于未指定任何 IP 地址，因此分配方法为**动态**。

        az network nic create     \
        --resource-group TestRG     \
        --name TestNIC2     \
        --location chinaeast     \
        --subnet FrontEnd    \
        --vnet-name TestVNet

    预期输出：

        {
            "newNIC": {
                "dnsSettings": {
                "appliedDnsServers": [],
                "dnsServers": []
                },
                "enableIPForwarding": false,
                "ipConfigurations": [
                {
                    "etag": "W/"<guid>"",
                    "id": "/subscriptions/<guid>/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/TestNIC2/ipConfigurations/ipconfig1",
                    "name": "ipconfig1",
                    "properties": {
                    "primary": true,
                    "privateIPAddress": "192.168.1.4",
                    "privateIPAllocationMethod": "Dynamic",
                    "provisioningState": "Succeeded",
                    "subnet": {
                        "id": "/subscriptions/<guid>/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd",
                        "resourceGroup": "TestRG"
                    }
                    },
                    "resourceGroup": "TestRG"
                }
                ],
                "provisioningState": "Succeeded",
                "resourceGuid": "0808a61c-476f-4d08-98ee-0fa83671b010"
            }
        }

2. 运行 **azure vm set** 命令以更改 VM 使用的 NIC。

        azure vm set -g TestRG -n DNS01 -N TestNIC2

    预期输出：

        [
            {
                "id": "/subscriptions/0e220bf6-5caa-4e9f-8383-51f16b6c109f/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/TestNIC3",
                "primary": true,
                "resourceGroup": "TestRG"
            }
        ]

    > [AZURE.NOTE]
    如果 VM 的空间足以拥有多个 NIC，请运行 **azure network nic delete** 命令删除旧版 NIC。
   
## 后续步骤
* 了解[保留公共 IP](/documentation/articles/virtual-networks-reserved-public-ip/) 地址。
* 了解[实例层级公共 IP (ILPIP)](/documentation/articles/virtual-networks-instance-level-public-ip/) 地址。
* 查阅[保留 IP REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn722420.aspx)。

<!---HONumber=Mooncake_0327_2017-->
<!--Update_Description: change from CLI 1.0 to CLI 2.0-->