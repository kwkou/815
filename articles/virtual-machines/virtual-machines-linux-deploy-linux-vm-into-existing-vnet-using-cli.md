<properties
    pageTitle="使用 Azure CLI 2.0（预览版）将 Linux VM 部署到现有网络中 | Azure"
    description="了解如何使用 Azure CLI 2.0（预览版）将 Linux 虚拟机部署到现有虚拟网络中"
    services="virtual-machines-linux"
    documentationcenter="virtual-machines"
    author="iainfoulds"
    manager="timlt"
    editor=""
    tags="azure-resource-manager" />
<tags 
    ms.assetid=""
    ms.service="virtual-machines-linux"
    ms.workload="infrastructure"
    ms.tgt_pltfrm="vm-linux"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/31/2017"
    wacn.date="04/10/2017"
    ms.author="iainfou" />

# 使用 Azure CLI 2.0（预览版）将 Linux VM 部署到现有虚拟网络

本文说明如何使用 Azure CLI 2.0（预览版）将虚拟机 (VM) 部署到现有虚拟网络。要求包括：

- [一个 Azure 帐户](/pricing/1rmb-trial/)
- [SSH 公钥和私钥文件](/documentation/articles/virtual-machines-linux-mac-create-ssh-keys/)

## 用于完成任务的 CLI 版本
可使用以下 CLI 版本之一完成任务：

- [Azure CLI 1.0](/documentation/articles/virtual-machines-linux-deploy-linux-vm-into-existing-vnet-using-cli-nodejs/)：用于经典部署模型和资源管理部署模型的 CLI
- [Azure CLI 2.0（预览版）](#quick-commands)：用于资源管理部署模型（本文）的下一代 CLI

## <a name="quick-commands"></a> 快速命令
如果需要快速完成任务，以下部分详细介绍所需的命令。本文档的余下部分（[从此处开始](#detailed-walkthrough)）提供了每个步骤的更详细信息和上下文。

若要创建此自定义环境，需要安装最新的 [Azure CLI 2.0（预览版）](https://docs.microsoft.com/cli/azure/install-az-cli2)并使用 [az login](https://docs.microsoft.com/cli/azure/#login) 登录到 Azure 帐户。

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

在以下示例中，请将示例参数名称替换为你自己的值。示例参数名称包括 `myResourceGroup`、`myVnet` 和 `myVM`。

**先决条件：**Azure 资源组、虚拟网络和子网、使用 SSH 入站连接的网络安全组，以及虚拟网络接口卡。

### 将 VM 部署到虚拟网络基础结构中

    az vm create \
        --resource-group myResourceGroup \
        --name myVM \
        --image Debian \
        --admin-username azureuser \
        --ssh-key-value ~/.ssh/id_rsa.pub \
        --nics myNic \
        --use-unmanaged-disk

## <a name="detailed-walkthrough"></a> 详细演练

建议选择静态的、长期存在的且部署频率极低的资源作为 Azure 资产，例如虚拟网络和网络安全组。部署虚拟网络后，新部署可以重复使用它，而不会对基础结构产生任何负面影响。可将虚拟网络想像为传统硬件网络交换机 - 不需要为全新的硬件交换机配置每个部署。正确配置虚拟网络后，在该虚拟网络的整个生命周期中，我们可以持续反复将新 VM 部署到该虚拟网络，只需做很少的更改（如果有）。

若要创建此自定义环境，需要安装最新的 [Azure CLI 2.0（预览版）](https://docs.microsoft.com/cli/azure/install-az-cli2)并使用 [az login](https://docs.microsoft.com/cli/azure/#login) 登录到 Azure 帐户。

在以下示例中，请将示例参数名称替换为你自己的值。示例参数名称包括 `myResourceGroup`、`myVnet` 和 `myVM`。

## 创建资源组

首先创建 Azure 资源组，以便组织在本演练中创建的所有内容。有关资源组的详细信息，请参阅 [Azure Resource Manager 概述](/documentation/articles/resource-group-overview/)。使用 [az group create](https://docs.microsoft.com/cli/azure/group#create) 创建资源组。以下示例在 `chinanorth` 位置创建一个名为 `myResourceGroup` 的资源组：

    az group create \
        --name myResourceGroup \
        --location chinanorth

## 创建虚拟网络

现在创建一个 Azure 虚拟网络，以便在其中启动 VM。有关虚拟网络的详细信息，请参阅[使用 Azure CLI 创建虚拟网络](/documentation/articles/virtual-networks-create-vnet-arm-cli/)。使用 [az network vnet create](https://docs.microsoft.com/cli/azure/network/vnet#create) 创建虚拟网络。以下示例创建名为 `myVnet` 的虚拟网络和名为 `mySubnet` 的子网：

    az network vnet create \
        --resource-group myResourceGroup \
        --location chinanorth \
        --name myVnet \
        --address-prefix 10.10.0.0/16 \
        --subnet-name mySubnet \
        --subnet-prefix 10.10.1.0/24

## 创建网络安全组

Azure 网络安全组相当于网络层防火墙。有关网络安全组的详细信息，请参阅[如何在 Azure CLI 中创建网络安全组](/documentation/articles/virtual-networks-create-nsg-arm-cli/)。使用 [az network nsg create](https://docs.microsoft.com/cli/azure/network/nsg#create) 创建网络安全组。以下示例创建名为 `myNetworkSecurityGroup` 的网络安全组：

    az network nsg create \
        --resource-group myResourceGroup \
        --location chinanorth \
        --name myNetworkSecurityGroup

## 添加入站 SSH 允许规则

Linux VM 需要从 Internet 访问，因此需要允许通过网络将入站端口 22 流量传递到 Linux VM 上的端口 22 的规则。使用 [az network nsg rule create](https://docs.microsoft.com/cli/azure/network/nsg/rule#create) 为网络安全组添加入站规则。以下示例创建名为 `myNetworkSecurityGroupRuleSSH` 的规则：

    az network nsg rule create \
        --resource-group myResourceGroup \
        --nsg-name myNetworkSecurityGroup \
        --name myNetworkSecurityGroupRuleSSH \
        --protocol tcp \
        --direction inbound \
        --priority 1000 \
        --source-address-prefix '*' \
        --source-port-range '*' \
        --destination-address-prefix '*' \
        --destination-port-range 22 \
        --access allow

## 将子网附加到网络安全组

可将网络安全组规则应用到子网或特定的虚拟网络接口。下面我们将网络安全组附加到子网。使用 [az network vnet subnet update](https://docs.microsoft.com/cli/azure/network/vnet/subnet#update) 将子网附加到网络安全组：

    az network vnet subnet update \
        --resource-group myResourceGroup \
        --vnet-name myVnet \
        --name mySubnet \
        --network-security-group myNetworkSecurityGroup

## 将虚拟网络接口卡添加到子网

虚拟网络接口卡 (VNic) 非常重要，可将它们连接到不同的 VM，因此可以重复使用。VNic 的这种可重复使用的特点使其可以用作静态资源，而 VM 可能是临时性的。使用 [az network nic create](https://docs.microsoft.com/cli/azure/network/nic#create) 创建 VNic 并将其与子网关联。以下示例创建名为 `myNic` 的 VNic：

    az network nic create \
        --resource-group myResourceGroup \
        --location chinanorth \
        --name myNic \
        --vnet-name myVnet \
        --subnet mySubnet

## 将 VM 部署到虚拟网络基础结构中

现在，我们已有一个虚拟网络、一个子网，以及一个充当防火墙的网络安全组，该网络安全组可以通过阻止所有入站流量（用于 SSH 的端口 22 除外）来保护子网。现在，可将 VM 部署在这个现有的网络基础结构内。

使用 [az vm create](https://docs.microsoft.com/cli/azure/vm#create) 创建 VM。有关可在 Azure CLI 2.0（预览版）中用来部署完整 VM 的标志的详细信息，请参阅[使用 Azure CLI 创建完整的 Linux 环境](/documentation/articles/virtual-machines-linux-create-cli-complete/)。

以下示例使用 Azure 非托管磁盘创建 VM。

    az vm create \
        --resource-group myResourceGroup \
        --name myVM \
        --image Debian \
        --admin-username azureuser \
        --ssh-key-value ~/.ssh/id_rsa.pub \
        --nics myNic \
        --use-unmanaged-disk \
        --storage-account mystorageaccount

通过使用 CLI 标志调用现有资源，我们指示 Azure 将 VM 部署在现有网络内部。如前所述，部署虚拟网络和子网后，可将其作为静态资源或永久资源保留在 Azure 区域中。在此示例中，我们没有为 VNic 创建并分配公共 IP 地址，因此，无法通过 Internet 公开访问该 VM。有关详细信息，请参阅[在 Azure CLI 中创建使用静态公共 IP 的 VM](/documentation/articles/virtual-network-deploy-static-pip-arm-cli/)。

## 后续步骤
若要详细了解在 Azure 中创建虚拟机的各种方法，请参阅以下资源：

* [Use an Azure Resource Manager template to create a specific deployment（使用 Azure Resource Manager 模板创建特定部署）](/documentation/articles/virtual-machines-linux-cli-deploy-templates/)
* [Create your own custom environment for a Linux VM using Azure CLI commands directly（直接使用 Azure CLI 命令为 Linux VM 创建用户自己的自定义环境）](/documentation/articles/virtual-machines-linux-create-cli-complete/)
* [使用模板在 Azure 上创建 Linux VM](/documentation/articles/virtual-machines-linux-create-ssh-secured-vm-from-template/)

<!---HONumber=Mooncake_0320_2017-->