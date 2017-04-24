<properties
    pageTitle="在 Azure CLI 2.0 中使用内部 DNS 进行 VM 名称解析 | Azure"
    description="如何使用 Azure CLI 2.0 在 Azure 中创建虚拟网络接口卡以及使用内部 DNS 进行 VM 名称解析"
    services="virtual-machines-linux"
    documentationcenter=""
    author="vlivech"
    manager="timlt"
    editor=""
    tags="azure-resource-manager"
    translationtype="Human Translation" />
<tags
    ms.assetid=""
    ms.service="virtual-machines-linux"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-linux"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/16/2017"
    wacn.date="04/24/2017"
    ms.author="v-livech"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="bc3fba0ca7a6313477f6e465ddcb2dfd2e2f6819"
    ms.lasthandoff="04/14/2017" />

# <a name="create-virtual-network-interface-cards-and-use-internal-dns-for-vm-name-resolution-on-azure"></a>创建虚拟网络接口卡，以及在 Azure 上使用内部 DNS 进行 VM 名称解析
本文介绍了如何通过 Azure CLI 2.0 使用虚拟网络接口卡 (vNic) 和 DNS 标签名称为 Linux VM 设置静态内部 DNS 名称。 也可以使用 [Azure CLI 1.0](/documentation/articles/virtual-machines-linux-static-dns-name-resolution-for-linux-on-azure-nodejs/) 执行这些步骤。 静态 DNS 名称用于永久基础结构服务，如本文档所使用的 Jenkins 生成服务器或 Git 服务器。

要求包括：

* [一个 Azure 帐户](/pricing/1rmb-trial/)
* [SSH 公钥和私钥文件](/documentation/articles/virtual-machines-linux-mac-create-ssh-keys/)

## <a name="quick-commands"></a>快速命令
如果需要快速完成任务，以下部分详细介绍所需的命令。 本文档的余下部分（[从此处开始](#detailed-walkthrough)）提供了每个步骤的更详细信息和上下文。 若要执行这些步骤，需要安装最新的 [Azure CLI 2.0](https://docs.microsoft.com/zh-cn/cli/azure/install-az-cli2)，并使用 [az login](https://docs.microsoft.com/zh-cn/cli/azure/#login) 登录到 Azure 帐户。

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

先决条件：资源组、虚拟网络和子网、带 SSH 入站的网络安全组。

### <a name="create-a-virtual-network-interface-card-with-a-static-internal-dns-name"></a>使用静态内部 DNS 名称创建虚拟网络接口卡
使用 [az network nic create](https://docs.microsoft.com/zh-cn/cli/azure/network/nic#create) 创建 vNic。 `--internal-dns-name` CLI 标志用于设置 DNS 标签，该标签为虚拟网络接口卡 (vNic) 提供静态 DNS 名称。 以下示例创建名为 `myNic` 的 vNic，将其连接到 `myVnet` 虚拟网络，并创建名为 `jenkins` 的内部 DNS 名称记录：

    az network nic create \
        --resource-group myResourceGroup \
        --name myNic \
        --vnet-name myVnet \
        --subnet mySubnet \
        --internal-dns-name jenkins

### <a name="deploy-a-vm-and-connect-the-vnic"></a>部署 VM 并连接 vNic
使用 [az vm create](https://docs.microsoft.com/zh-cn/cli/azure/vm#create)创建 VM。 在部署到 Azure 期间，`--nics` 标志将 VNic 连接到 VM。 以下示例使用 Azure 非托管磁盘创建名为 `myVM` 的 VM，并附加上一步中名为 `myNic` 的 vNic：

    az vm create \
        --resource-group myResourceGroup \
        --name myVM \
        --nics myNic \
        --image UbuntuLTS \
        --admin-username azureuser \
        --ssh-key-value ~/.ssh/id_rsa.pub \
        --use-unmanaged-disk

## <a name="detailed-walkthrough"></a>详细演练

Azure 上的完整持续集成和持续部署 (CiCd) 基础结构需要某些服务器是静态服务器或长期存在的服务器。 建议选择静态的、长期存在的且部署频率极低的资源作为 Azure 资产，例如虚拟网络和网络安全组。 部署虚拟网络后，新部署可以重复使用它，而不会对基础结构产生任何负面影响。 以后可在开发或测试环境中添加 Git 存储库服务器或 Jenkins 自动化服务器，将 CiCd 提供给此虚拟网络。  

内部 DNS 名称仅在 Azure 虚拟网络内可解析。 由于 DNS 名称是内部类，因此它们无法解析到外部 Internet，从而为基础结构提供了附加安全性。

在以下示例中，请将示例参数名称替换为你自己的值。 示例参数名称包括 `myResourceGroup`、`myNic` 和 `myVM`。

## <a name="create-the-resource-group"></a>创建资源组
首先，使用 [az group create](https://docs.microsoft.com/zh-cn/cli/azure/group#create) 创建资源组。 以下示例在 `chinanorth` 位置创建名为 `myResourceGroup` 的资源组：

    az group create --name myResourceGroup --location chinanorth

## <a name="create-the-virtual-network"></a>创建虚拟网络

下一步是构建虚拟网络，以便在其中启动 VM。 该虚拟网络包含本演练中使用的一个子网。 有关 Azure 虚拟网络的详细信息，请参阅[使用 Azure CLI 创建虚拟网络](/documentation/articles/virtual-networks-create-vnet-arm-cli/)。 

使用 [az network vnet create](https://docs.microsoft.com/zh-cn/cli/azure/network/vnet#create) 创建虚拟网络。 以下示例创建一个名为 `myVnet` 的虚拟网络和名为 `mySubnet` 的子网：

    az network vnet create \
        --resource-group myResourceGroup \
        --name myVnet \
        --address-prefix 192.168.0.0/16 \
        --subnet-name mySubnet \
        --subnet-prefix 192.168.1.0/24

## <a name="create-the-network-security-group"></a>创建网络安全组
Azure 网络安全组相当于网络层防火墙。 有关网络安全组的详细信息，请参阅[如何在 Azure CLI 中创建 NSG](/documentation/articles/virtual-networks-create-nsg-arm-cli/)。 

使用 [az network nsg create](https://docs.microsoft.com/zh-cn/cli/azure/network/nsg#create)创建网络安全组。 以下示例创建名为 `myNetworkSecurityGroup`的网络安全组：

    az network nsg create \
        --resource-group myResourceGroup \
        --name myNetworkSecurityGroup

## <a name="add-an-inbound-rule-to-allow-ssh"></a>添加入站规则以允许 SSH
使用 [az network nsg rule create](https://docs.microsoft.com/zh-cn/cli/azure/network/nsg/rule#create)为网络安全组添加入站规则。 以下示例创建名为 `myRuleAllowSSH`的规则：

    az network nsg rule create \
        --resource-group myResourceGroup \
        --nsg-name myNetworkSecurityGroup \
        --name myRuleAllowSSH \
        --protocol tcp \
        --direction inbound \
        --priority 1000 \
        --source-address-prefix '*' \
        --source-port-range '*' \
        --destination-address-prefix '*' \
        --destination-port-range 22 \
        --access allow

## <a name="associate-the-subnet-with-the-network-security-group"></a>将子网与网络安全组相关联
若要将子网与网络安全组相关联，请使用 [az network vnet subnet update](https://docs.microsoft.com/zh-cn/cli/azure/network/vnet/subnet#update)。 以下示例将名为 `mySubnet` 的子网与名为 `myNetworkSecurityGroup` 的网络安全组相关联：

    az network vnet subnet update \
        --resource-group myResourceGroup \
        --vnet-name myVnet \
        --name mySubnet \
        --network-security-group myNetworkSecurityGroup

## <a name="create-the-virtual-network-interface-card-and-static-dns-names"></a>创建虚拟网络接口卡和静态 DNS 名称
Azure 非常灵活，但若要使用 DNS 名称进行 VM 名称解析，则需要创建包含 DNS 标签的虚拟网络接口卡 (vNic)。 可通过基础结构生命周期将 VNic 连接到不同的 VM 供用户重复使用，因此 VNic 很重要。 通过这种方法可将 vNic 用作静态资源，而 VM 可能是临时性的。 在 VNic 上使用 DNS 标签可从 VNet 中的其他 VM 启用简单名称解析。 使用可解析名称可使其他 VM 能够通过 DNS 名称 `Jenkins` 或作为 `gitrepo` 的 Git 服务器访问自动化服务器。  

使用 [az network nic create](https://docs.microsoft.com/zh-cn/cli/azure/network/nic#create) 创建 vNic。 以下示例创建名为 `myNic` 的 vNic，将其连接到名为 `myVnet` 的 `myVnet` 虚拟网络，并创建名为 `jenkins` 的内部 DNS 名称记录：

    az network nic create \
        --resource-group myResourceGroup \
        --name myNic \
        --vnet-name myVnet \
        --subnet mySubnet \
        --internal-dns-name jenkins

## <a name="deploy-the-vm-into-the-virtual-network-infrastructure"></a>将 VM 部署到虚拟网络基础结构
现在，我们已有一个虚拟网络和子网、一个充当防火墙的网络安全组（可以通过阻止所有入站流量（用于 SSH 的端口 22 除外）来保护子网），以及一个 vNic。 现在，可在此现有网络基础结构中部署 VM。

使用 [az vm create](https://docs.microsoft.com/zh-cn/cli/azure/vm#create) 创建 VM。 以下示例使用 Azure 非托管磁盘创建名为 `myVM` 的 VM，并附加上一步中名为 `myNic` 的 vNic：

    az vm create \
        --resource-group myResourceGroup \
        --name myVM \
        --nics myNic \
        --image UbuntuLTS \
        --admin-username azureuser \
        --ssh-key-value ~/.ssh/id_rsa.pub \
        --use-unmanaged-disk

使用 CLI 标志调用现有资源是为了指示 Azure 将 VM 部署到现有网络中。 重述一遍，VNet 和子网一经部署，便可在 Azure 区域内保留为静态或永久资源。  

## <a name="next-steps"></a>后续步骤

* [使用 Azure Resource Manager 模板创建特定部署](/documentation/articles/virtual-machines-linux-cli-deploy-templates/)
* [直接使用 Azure CLI 命令创建自定义的 Linux VM 环境](/documentation/articles/virtual-machines-linux-create-cli-complete/)
* [使用模板在 Azure 上创建 Linux VM](/documentation/articles/virtual-machines-linux-create-ssh-secured-vm-from-template/)
<!--Update_Description: wording update-->