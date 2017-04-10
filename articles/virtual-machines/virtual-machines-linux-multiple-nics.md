<properties
    pageTitle="使用 Azure CLI 2.0（预览版）创建具有多个 NIC 的 VM | Azure"
    description="了解如何使用 Azure CLI 2.0（预览版）或 Resource Manager 模板创建具有多个 NIC 的 Linux VM。"
    services="virtual-machines-linux"
    documentationcenter=""
    author="iainfoulds"
    manager="timlt"
    editor="" />
<tags 
    ms.assetid="5d2d04d0-fc62-45fa-88b1-61808a2bc691"
    ms.service="virtual-machines-linux"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="vm-linux"
    ms.workload="infrastructure"
    ms.date="02/10/2017"
    wacn.date="04/10/2017"
    ms.author="iainfou" />

# 使用 Azure CLI 2.0（预览版）创建具有多个 NIC 的 VM
可以在 Azure 中创建附有多个虚拟网络接口 (NIC) 的虚拟机 (VM)。一种常见方案是为前端和后端连接使用不同的子网，或者为监视或备份解决方案使用一个专用网络。本文提供用于创建附有多个 NIC 的 VM 的快速命令。有关详细信息，包括如何在自己的 Bash 脚本中创建多个 NIC，请阅读 [Deploying multi-NIC VMs](/documentation/articles/virtual-network-deploy-multinic-arm-cli/)（部署具有多个 NIC 的 VM）。不同的 [VM 大小](/documentation/articles/virtual-machines-linux-sizes/)支持不同数目的 NIC，因此请相应地调整 VM 的大小。

> [AZURE.WARNING]
必须在创建 VM 时附加多个 NIC - 不能将 NIC 添加到现有 VM。可以[基于原始虚拟磁盘创建 VM](/documentation/articles/virtual-machines-linux-copy-vm/)，并在部署 VM 时创建多个 NIC。

## 用于完成任务的 CLI 版本
可使用以下 CLI 版本之一完成任务：

- [Azure CLI 1.0](/documentation/articles/virtual-machines-linux-multiple-nics-nodejs/)：用于经典部署模型和资源管理部署模型的 CLI
- [Azure CLI 2.0（预览版）](#create-supporting-resources)：用于资源管理部署模型（本文）的下一代 CLI

## <a name="create-supporting-resources"></a> 创建支持资源
安装最新的 [Azure CLI 2.0（预览版）](https://docs.microsoft.com/cli/azure/install-az-cli2)并使用 [az login](https://docs.microsoft.com/cli/azure/#login) 登录到 Azure 帐户。

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

在以下示例中，请将示例参数名称替换为你自己的值。示例参数名称包括 `myResourceGroup`、`mystorageaccount` 和 `myVM`。

首先，使用 [az group create](https://docs.microsoft.com/cli/azure/group#create) 创建资源组。以下示例在 `ChinaNorth` 位置创建名为 `myResourceGroup` 的资源组：

    az group create --name myResourceGroup --location chinanorth

使用 [az network vnet create](https://docs.microsoft.com/cli/azure/network/vnet#create) 创建虚拟网络。以下示例创建名为 `myVnet` 的虚拟网络和名为 `mySubnetFrontEnd` 的子网：

    az network vnet create --resource-group myResourceGroup --name myVnet \
      --address-prefix 192.168.0.0/16 --subnet-name mySubnetFrontEnd --subnet-prefix 192.168.1.0/24

使用 [az network vnet subnet create](https://docs.microsoft.com/cli/azure/network/vnet/subnet#create) 创建适用于后端流量的子网。以下示例创建名为 `mySubnetBackEnd` 的子网：

    az network vnet subnet create --resource-group myResourceGroup --vnet-name myVnet \
        --name mySubnetBackEnd --address-prefix 192.168.2.0/24

## 创建和配置多个 NIC
详细了解如何[使用 Azure CLI 部署多个 NIC](/documentation/articles/virtual-network-deploy-multinic-arm-cli/)，包括如何编写轮流创建所有 NIC 的过程脚本。

通常，我们会创建[网络安全组](/documentation/articles/virtual-networks-nsg/)或[负载均衡器](/documentation/articles/load-balancer-overview/)来帮助管理流量以及跨 VM 分布流量。使用 [az network nsg create](https://docs.microsoft.com/cli/azure/network/nsg#create) 创建网络安全组。以下示例创建名为 `myNetworkSecurityGroup` 的网络安全组：

    az network nsg create --resource-group myResourceGroup \
      --name myNetworkSecurityGroup

使用 [az network nic create](https://docs.microsoft.com/cli/azure/network/nic#create) 创建两个 NIC。以下示例创建两个名为 `myNic1` 和 `myNic2` 的 NIC（连接网络安全组），其中一个 NIC 将连接到每个子网：

    az network nic create --resource-group myResourceGroup --name myNic1 \
      --vnet-name myVnet --subnet mySubnetFrontEnd \
      --network-security-group myNetworkSecurityGroup
    az network nic create --resource-group myResourceGroup --name myNic2 \
      --vnet-name myVnet --subnet mySubnetBackEnd \
      --network-security-group myNetworkSecurityGroup

## 创建 VM 并附加 NIC
创建 VM 时，请使用 `--nics` 指定所创建的 NIC。还需要谨慎选择 VM 的大小。可添加到 VM 的 NIC 数目有限制。详细了解 [Linux VM 大小](/documentation/articles/virtual-machines-linux-sizes/)。

使用 [az vm create](https://docs.microsoft.com/cli/azure/vm#create) 创建 VM。以下示例使用 Azure 非托管磁盘创建名为 `myVM` 的 VM：

    az vm create \
        --resource-group myResourceGroup \
        --name myVM \
        --image UbuntuLTS \
        --size Standard_DS2_v2 \
        --admin-username azureuser \
        --ssh-key-value ~/.ssh/id_rsa.pub \
        --nics myNic1 myNic2 \
        --use-unmanaged-disk

## 使用 Resource Manager 模板创建多个 NIC
Azure Resource Manager 模板使用声明性 JSON 文件来定义环境。阅读 [Azure Resource Manager 概述](/documentation/articles/resource-group-overview/)。Resource Manager 模板可让你在部署期间创建资源的多个实例，例如，创建多个 NIC。使用 *copy* 指定要创建的实例数：

    "copy": {
        "name": "multiplenics"
        "count": "[parameters('count')]"
    }

阅读有关[使用 *copy* 创建多个实例](/documentation/articles/resource-group-create-multiple/)的详细信息。

也可以使用 `copyIndex()` 并在资源名称中追加一个数字，来创建 `myNic1`、`myNic2`，等等。下面显示了追加索引值的示例：

    "name": "[concat('myNic', copyIndex())]", 

阅读[使用 Resource Manager 模板创建多个 NIC](/documentation/articles/virtual-network-deploy-multinic-arm-template/) 的完整示例。

## 后续步骤
尝试创建具有多个 NIC 的 VM 时，请务必查看 [Linux VM sizes](/documentation/articles/virtual-machines-linux-sizes/)（Linux VM 大小）。注意每个 VM 大小支持的 NIC 数目上限。

请记住，不能将其他 NIC 添加到现有 VM，而必须在部署 VM 时创建所有 NIC。仔细规划部署，确保从一开始就建立了全部所需的网络连接。

<!---HONumber=Mooncake_0320_2017-->