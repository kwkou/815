<properties
    pageTitle="将 Linux VM 部署到现有网络 - Azure CLI | Azure"
    description="使用 CLI 将 Linux VM 部署到现有虚拟网络。"
    services="virtual-machines-linux"
    documentationcenter="virtual-machines"
    author="vlivech"
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
    ms.date="12/05/2016"
    wacn.date="04/06/2017"
    ms.author="v-livech" />  


# 使用 CLI 将 Linux VM 部署到现有 Azure 虚拟网络

本文说明如何使用 CLI 标志将 VM 部署到现有的虚拟网络 (VNet)。要求包括：

- [一个 Azure 帐户](/pricing/1rmb-trial/)
- [SSH 公钥和私钥文件](/documentation/articles/virtual-machines-linux-mac-create-ssh-keys/)

## 用于完成任务的 CLI 版本
可使用以下 CLI 版本之一完成任务：

- [Azure CLI 1.0](#quick-commands)：用于经典部署模型和资源管理部署模型（本文）的 CLI
- [Azure CLI 2.0（预览版）](/documentation/articles/virtual-machines-linux-deploy-linux-vm-into-existing-vnet-using-cli/)：用于资源管理部署模型的下一代 CLI

## <a name="quick-commands"></a> 快速命令

如果需要快速完成任务，以下部分详细介绍所需的命令。本文档的余下部分（[从此处开始](/documentation/articles/virtual-machines-linux-deploy-linux-vm-into-existing-vnet-using-cli/#detailed-walkthrough)）提供了每个步骤的更详细信息和上下文。

前提条件：资源组、VNet、将 SSH 入站的 NSG、子网。将任何示例替换为你自己的设置。

### 将 VM 部署到虚拟网络基础结构中

    azure vm create myVM \
    -g myResourceGroup \
    -l chinanorth \
    -y linux \
    -Q Debian \
    -o myStorageAcct \
    -u myAdminUser \
    -M ~/.ssh/id_rsa.pub \
    -n myVM \
    -F myVNet \
    -j mySubnet \
    -N myVNic

## <a name="detailed-walkthrough"></a> 详细演练

建议 VNet 和 NSG 等 Azure 资产应该是静态的、很少部署的长期存在资源。部署 VNet 后，新部署可以重复使用它，而不会对基础结构产生任何负面影响。考虑一下作为传统硬件网络交换机的 VNet 的情况。不需每个部署都配置全新的硬件交换机。正确配置 VNet 后，在该 VNet 的整个生命周期中，我们可以继续反复将新的服务器部署到该 VNet，只需做很少的更改（如果有）。

## 创建资源组

首先，我们创建资源组，以便组织在本演练中创建的所有内容。有关 Azure 资源组的详细信息，请参阅 [Azure Resource Manager 概述](/documentation/articles/resource-group-overview/)

    azure group create myResourceGroup \
    --location chinanorth

## 创建 VNet

第一步是生成用于在其中启动 VM 的 VNet。该 VNet 包含本演练所用的一个子网。有关 Azure VNet 的详细信息，请参阅[使用 Azure CLI 创建虚拟网络](/documentation/articles/virtual-networks-create-vnet-arm-cli/)

    azure network vnet create myVNet \
    --resource-group myResourceGroup \
    --address-prefixes 10.10.0.0/24 \
    --location chinanorth

## 创建 NSG

子网在现有网络安全组后面构建，因此我们在构建子网之前先构建 NSG。Azure NSG 相当于网络层防火墙。有关 Azure NSG 的详细信息，请参阅[如何在 Azure CLI 中创建 NSG](/documentation/articles/virtual-networks-create-nsg-arm-cli/)

    azure network nsg create myNSG \
    --resource-group myResourceGroup \
    --location chinanorth

## 添加入站 SSH 允许规则

Linux VM 需要从 Internet 访问，因此需要允许通过网络将入站端口 22 流量传递到 Linux VM 上的端口 22 的规则。

    azure network nsg rule create inboundSSH \
    --resource-group myResourceGroup \
    --nsg-name myNSG \
    --access Allow \
    --protocol Tcp \
    --direction Inbound \
    --priority 100 \
    --source-address-prefix Internet \
    --source-port-range 22 \
    --destination-address-prefix 10.10.0.0/24 \
    --destination-port-range 22

## 将子网添加到 VNet

VNet 中的 VM 必须位于一个子网中。每个 VNet 可以有多个子网。创建子网并将子网与 NSG 相关联，以便将防火墙添加到子网。

    azure network vnet subnet create mySubNet \
    --resource-group myResourceGroup \
    --vnet-name myVNet \
    --address-prefix 10.10.0.0/26 \
    --network-security-group-name myNSG

现在，子网已添加到 VNet 中，并已与 NSG 和 NSG 规则相关联。

## 将 VNic 添加到子网

虚拟网卡 (VNic) 很重要，因为用户可以通过将它们连接到不同的 VM 来重新使用它们，这使 VNic 保持作为静态资源，而 VM 可以是临时 VM。创建 VNic 并将其与上一步中创建的子网相关联。

    azure network nic create myVNic \
    -g myResourceGroup \
    -l chinanorth \
    -m myVNet \
    -k mySubNet

## 将 VM 部署到 VNet 和 NSG 中

我们现在已有一个 VNet、该 VNet 中的一个子网和一个 NSG，该 NSG 通过阻止 SSH 端口 22 以外的所有入站流量来充当防火墙保护我们的子网。现在，可将 VM 部署在这个现有的网络基础结构内。

使用 Azure CLI 和 `azure vm create` 命令，将 Linux VM 部署到现有的 Azure 资源组、VNet、子网和 VNic 中。有关使用 CLI 部署完整的 VM 的详细信息，请参阅[使用 Azure CLI 创建完整的 Linux 环境](/documentation/articles/virtual-machines-linux-create-cli-complete/)

    azure vm create myVM \
    --resource-group myResourceGroup \
    --location chinanorth \
    --os-type linux \
    --image-urn Debian \
    --storage-account-name mystorageaccount \
    --admin-username myAdminUser \
    --ssh-publickey-file ~/.ssh/id_rsa.pub \
    --vnet-name myVNet \
    --vnet-subnet-name mySubnet \
    --nic-name myVNic

通过使用 CLI 标志调用现有资源，我们指示 Azure 将 VM 部署在现有网络内部。重述一遍，VNet 和子网一经部署，便可在 Azure 区域内保留为静态或永久资源。

## 后续步骤

* [Use an Azure Resource Manager template to create a specific deployment（使用 Azure Resource Manager 模板创建特定部署）](/documentation/articles/virtual-machines-linux-cli-deploy-templates/)
* [Create your own custom environment for a Linux VM using Azure CLI commands directly（直接使用 Azure CLI 命令为 Linux VM 创建用户自己的自定义环境）](/documentation/articles/virtual-machines-linux-create-cli-complete/)
* [使用模板在 Azure 上创建 Linux VM](/documentation/articles/virtual-machines-linux-create-ssh-secured-vm-from-template/)

<!---HONumber=Mooncake_0313_2017-->