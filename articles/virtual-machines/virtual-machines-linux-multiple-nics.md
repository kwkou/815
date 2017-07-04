<properties
    pageTitle="使用 Azure CLI 2.0 创建具有多个 NIC 的 Linux VM | Azure"
    description="了解如何使用 Azure CLI 2.0 或 Resource Manager 模板创建具有多个 NIC 的 Linux VM。"
    services="virtual-machines-linux"
    documentationcenter=""
    author="iainfoulds"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="5d2d04d0-fc62-45fa-88b1-61808a2bc691"
    ms.service="virtual-machines-linux"
    ms.devlang="azurecli"
    ms.topic="article"
    ms.tgt_pltfrm="vm-linux"
    ms.workload="infrastructure"
    ms.date="02/10/2017"
    wacn.date="05/15/2017"
    ms.author="iainfou"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="65e056610b6edec126672aec1956e1ec8ac93621"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="create-a-linux-vm-with-multiple-nics"></a>创建具有多个 NIC 的 Linux VM
可以在 Azure 中创建附有多个虚拟网络接口 (NIC) 的虚拟机 (VM)。 一种常见方案是为前端和后端连接使用不同的子网，或者为监视或备份解决方案使用一个专用网络。 本文提供用于创建附有多个 NIC 的 VM 的快速命令。 有关详细信息（包括如何在自己的 Bash 脚本中创建多个 NIC），请阅读[部署具有多个 NIC 的 VM](/documentation/articles/virtual-network-deploy-multinic-arm-cli/)。 不同的 [VM 大小](/documentation/articles/virtual-machines-linux-sizes/)支持不同数目的 NIC，因此请相应地调整 VM 的大小。

本文详述了如何使用 Azure CLI 2.0 创建具有多个 NIC 的 VM。 还可以使用 [Azure CLI 1.0](/documentation/articles/virtual-machines-linux-multiple-nics-nodejs/) 执行这些步骤。

## <a name="create-supporting-resources"></a>创建支持资源
安装最新的 [Azure CLI 2.0](https://docs.microsoft.com/zh-cn/cli/azure/install-az-cli2) 并使用 [az login](https://docs.microsoft.com/zh-cn/cli/azure/#login) 登录到 Azure 帐户。

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

在以下示例中，请将示例参数名称替换为自己的值。 示例参数名称包括 `myResourceGroup`、`mystorageaccount` 和 `myVM`。

首先，使用 [az group create](https://docs.microsoft.com/zh-cn/cli/azure/group#create) 创建资源组。 以下示例在 `ChinaNorth` 位置创建名为 `myResourceGroup` 的资源组：

    az group create --name myResourceGroup --location chinanorth

使用 [az network vnet create](https://docs.microsoft.com/zh-cn/cli/azure/network/vnet#create) 创建虚拟网络。 以下示例创建一个名为 `myVnet` 的虚拟网络和名为 `mySubnetFrontEnd` 的子网：

    az network vnet create --resource-group myResourceGroup --name myVnet \
      --address-prefix 192.168.0.0/16 --subnet-name mySubnetFrontEnd --subnet-prefix 192.168.1.0/24

使用 [az network vnet subnet create](https://docs.microsoft.com/zh-cn/cli/azure/network/vnet/subnet#create) 为后端通信流创建子网。 以下示例创建名为 `mySubnetBackEnd`的子网：

    az network vnet subnet create --resource-group myResourceGroup --vnet-name myVnet \
        --name mySubnetBackEnd --address-prefix 192.168.2.0/24

## <a name="create-and-configure-multiple-nics"></a>创建和配置多个 NIC
详细了解如何[使用 Azure CLI 部署多个 NIC](/documentation/articles/virtual-network-deploy-multinic-arm-cli/)，包括如何编写循环创建所有 NIC 的过程脚本。

通常会创建[网络安全组](/documentation/articles/virtual-networks-nsg/)或[负载均衡器](/documentation/articles/load-balancer-overview/)来帮助管理流量以及跨 VM 分布流量。 使用 [az network nsg create](https://docs.microsoft.com/zh-cn/cli/azure/network/nsg#create) 创建网络安全组。 以下示例创建名为 `myNetworkSecurityGroup`的网络安全组：

    az network nsg create --resource-group myResourceGroup \
      --name myNetworkSecurityGroup

使用 [az network nic create](https://docs.microsoft.com/zh-cn/cli/azure/network/nic#create) 创建两个 NIC。 以下示例将创建两个 NIC（名为 `myNic1` 和 `myNic2`），并连接到网络安全组，其中一个 NIC 连接到每个子网：

    az network nic create --resource-group myResourceGroup --name myNic1 \
      --vnet-name myVnet --subnet mySubnetFrontEnd \
      --network-security-group myNetworkSecurityGroup
    az network nic create --resource-group myResourceGroup --name myNic2 \
      --vnet-name myVnet --subnet mySubnetBackEnd \
      --network-security-group myNetworkSecurityGroup

## <a name="create-a-vm-and-attach-the-nics"></a>创建 VM 并附加 NIC
创建 VM 时，请使用 `--nics`指定所创建的 NIC。 还需要谨慎选择 VM 的大小。 可添加到 VM 的 NIC 数目有限制。 详细了解 [Linux VM 大小](/documentation/articles/virtual-machines-linux-sizes/)。 

使用 [az vm create](https://docs.microsoft.com/zh-cn/cli/azure/vm#create) 创建 VM。 以下示例使用 Azure 非托管磁盘创建一个名为 `myVM` 的 VM：

    az vm create \
        --resource-group myResourceGroup \
        --name myVM \
        --image UbuntuLTS \
        --size Standard_DS2_v2 \
        --admin-username azureuser \
        --ssh-key-value ~/.ssh/id_rsa.pub \
        --nics myNic1 myNic2 \
        --use-unmanaged-disk

## <a name="create-multiple-nics-using-resource-manager-templates"></a>使用 Resource Manager 模板创建多个 NIC
Azure Resource Manager 模板使用声明性 JSON 文件来定义环境。 可以阅读 [Azure Resource Manager 概述](/documentation/articles/resource-group-overview/)。 Resource Manager 模板可让你在部署期间创建资源的多个实例，例如，创建多个 NIC。 使用 *copy* 指定要创建的实例数：

    "copy": {
        "name": "multiplenics"
        "count": "[parameters('count')]"
    }

阅读有关[使用 *copy* 创建多个实例](/documentation/articles/resource-group-create-multiple/)的详细信息。 

也可以使用 `copyIndex()` 并在资源名称中追加一个数字，来创建 `myNic1`、`myNic2`，等等。下面显示了追加索引值的示例：

    "name": "[concat('myNic', copyIndex())]", 

可以阅读[使用 Resource Manager 模板创建多个 NIC](/documentation/articles/virtual-network-deploy-multinic-arm-template/) 的完整示例。

## <a name="next-steps"></a>后续步骤
尝试创建具有多个 NIC 的 VM 时，请查看 [Lnux VM 大小](/documentation/articles/virtual-machines-linux-sizes/)。 注意每个 VM 大小支持的 NIC 数目上限。

<!--Update_Description: wording update-->