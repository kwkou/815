<properties
    pageTitle="在 Azure 上使用内部 DNS 进行 VM 名称解析 | Azure"
    description="在 Azure 上使用内部 DNS 进行 VM 名称解析。"
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
    ms.date="12/05/2016"
    wacn.date="04/24/2017"
    ms.author="v-livech"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="347cbaae43020b0957c9a0198a972f5bb3d32a3f"
    ms.lasthandoff="04/14/2017" />

# <a name="using-internal-dns-for-vm-name-resolution-on-azure"></a>在 Azure 上使用内部 DNS 进行 VM 名称解析

本文说明如何使用虚拟 NIC 卡 (VNic) 和 DNS 标签名称为 Linux VM 设置静态内部 DNS 名称。 静态 DNS 名称用于永久基础结构服务，如本文档所使用的 Jenkins 生成服务器或 Git 服务器。

要求包括：

* [一个 Azure 帐户](/pricing/1rmb-trial/)
* [SSH 公钥和私钥文件](/documentation/articles/virtual-machines-linux-mac-create-ssh-keys/)

## <a name="cli-versions-to-complete-the-task"></a>用于完成任务的 CLI 版本
可使用以下 CLI 版本之一完成任务：

- [Azure CLI 1.0](#quick-commands) - 适用于经典部署模型和资源管理部署模型（本文）的 CLI
- [Azure CLI 2.0](/documentation/articles/virtual-machines-linux-static-dns-name-resolution-for-linux-on-azure/) - 适用于资源管理部署模型的下一代 CLI

## <a name="quick-commands"></a> 快速命令

如果需要快速完成任务，以下部分详细介绍所需的命令。 本文档的余下部分（ [从此处开始](#detailed-walkthrough)）提供了每个步骤的更详细信息和上下文。  

前提条件： 资源组、VNet、将 SSH 入站的 NSG、子网。

### <a name="create-a-vnic-with-a-static-internal-dns-name"></a>使用静态内部 DNS 名称创建 VNic

`-r` cli 标志用于设置 DNS 标签，该标签为 VNic 提供静态 DNS 名称。

    azure network nic create jenkinsVNic \
    -g myResourceGroup \
    -l chinanorth \
    -m myVNet \
    -k mySubNet \
    -r jenkins

### <a name="deploy-the-vm-into-the-vnet-nsg-and-connect-the-vnic"></a>将 VM 部署到 VNet、NSG 中并连接 VNic

在部署到 Azure 期间， `-N` 将 VNic 连接到新 VM。

    azure vm create jenkins \
    -g myResourceGroup \
    -l chinanorth \
    -y linux \
    -Q Debian \
    -o myStorageAcct \
    -u myAdminUser \
    -M ~/.ssh/id_rsa.pub \
    -F myVNet \
    -j mySubnet \
    -N jenkinsVNic

## <a name="detailed-walkthrough"></a> 详细演练

Azure 上的完整持续集成和持续部署 (CiCd) 基础结构需要某些服务器是静态服务器或长期存在的服务器。  建议虚拟网络 (VNet) 和网络安全组 (NSG) 等 Azure 资产应该是静态的、很少部署的长期存在资源。  部署 VNet 后，新部署可以重复使用它，而不会对基础结构产生任何负面影响。  向此静态网络添加 Git 存储库服务器和 Jenkins 自动化服务器可将 CiCd 提供给开发或测试环境。  

内部 DNS 名称仅在 Azure 虚拟网络内可解析。  由于 DNS 名称是内部类，因此它们无法解析到外部 Internet，从而为基础结构提供了附加安全性。

_将任何示例替换为你自己的命名。_

## <a name="create-the-resource-group"></a>创建资源组

需要使用资源组来组织在本演练中创建的所有内容。  有关 Azure 资源组的详细信息，请参阅 [Azure Resource Manager 概述](/documentation/articles/resource-group-overview/)。

    azure group create myResourceGroup \
    --location chinanorth

## <a name="create-the-vnet"></a>创建 VNet

第一步是生成用于在其中启动 VM 的 VNet。  该 VNet 包含本演练所用的一个子网。  有关 Azure VNet 的详细信息，请参阅[使用 Azure CLI 创建虚拟网络](/documentation/articles/virtual-networks-create-vnet-arm-cli/)

    azure network vnet create myVNet \
    --resource-group myResourceGroup \
    --address-prefixes 10.10.0.0/24 \
    --location chinanorth

## <a name="create-the-nsg"></a>创建 NSG

子网在现有网络安全组后面构建，因此我们在构建子网之前先构建 NSG。  Azure NSG 相当于网络层防火墙。  有关 Azure NSG 的详细信息，请参阅[如何在 Azure CLI 中创建 NSG](/documentation/articles/virtual-networks-create-nsg-arm-cli/)

    azure network nsg create myNSG \
    --resource-group myResourceGroup \
    --location chinanorth

## <a name="add-an-inbound-ssh-allow-rule"></a>添加入站 SSH 允许规则

Linux VM 需要从 Internet 访问，因此需要允许通过网络将入站端口 22 流量传递到 Linux VM 上的端口 22 的规则。

    azure network nsg rule create inboundSSH \
    --resource-group myResourceGroup \
    --nsg-name myNSG \
    --access Allow \
    --protocol Tcp \
    --direction Inbound \
    --priority 100 \
    --source-address-prefix * \
    --source-port-range * \
    --destination-address-prefix 10.10.0.0/24 \
    --destination-port-range 22

## <a name="add-a-subnet-to-the-vnet"></a>将子网添加到 VNet

VNet 中的 VM 必须位于一个子网中。  每个 VNet 可以有多个子网。  创建子网并将子网与 NSG 相关联，以便将防火墙添加到子网。

    azure network vnet subnet create mySubNet \
    --resource-group myResourceGroup \
    --vnet-name myVNet \
    --address-prefix 10.10.0.0/26 \
    --network-security-group-name myNSG

现在，子网已添加到 VNet 中，并已与 NSG 和 NSG 规则相关联。

## <a name="creating-static-dns-names"></a>创建静态 DNS 名称

Azure 非常灵活，但若要使用 DNS 名称进行 VM 名称解析，需要使用 DNS 标签将它们创建为虚拟网卡 (VNic)。  VNic 很重要，因为用户可以通过将它们连接到不同的 VM 来重新使用它们，这使 VNic 保持作为静态资源，而 VM 可以是临时 VM。  通过在 VNic 上使用 DNS 标签，我们将能够从 VNet 中的其他 VM 启用简单名称解析。  使用可解析名称可使其他 VM 能够通过 DNS 名称 `Jenkins` 或作为 `gitrepo` 的Git 服务器访问自动化服务器。  创建 VNic 并将其与上一步中创建的子网相关联。

    azure network nic create jenkinsVNic \
    -g myResourceGroup \
    -l chinanorth \
    -m myVNet \
    -k mySubNet \
    -r jenkins

## <a name="deploy-the-vm-into-the-vnet-and-nsg"></a>将 VM 部署到 VNet 和 NSG 中

我们现在已有一个 VNet、该 VNet 中的一个子网和一个 NSG，该 NSG 通过阻止 SSH 端口 22 以外的所有入站流量来充当防火墙保护我们的子网。  现在，可将 VM 部署在这个现有的网络基础结构内。

使用 Azure CLI 和 `azure vm create` 命令，将 Linux VM 部署到现有的 Azure 资源组、VNet、子网和 VNic 中。  若要详细了解如何使用 CLI 部署完整的 VM，请参阅[使用 Azure CLI 创建完整的 Linux 环境](/documentation/articles/virtual-machines-linux-create-cli-complete/)

    azure vm create jenkins \
    --resource-group myResourceGroup myVM \
    --location chinanorth \
    --os-type linux \
    --image-urn Debian \
    --storage-account-name mystorageaccount \
    --admin-username myAdminUser \
    --ssh-publickey-file ~/.ssh/id_rsa.pub \
    --vnet-name myVNet \
    --vnet-subnet-name mySubnet \
    --nic-name jenkinsVNic

通过使用 CLI 标志调用现有资源，我们指示 Azure 将 VM 部署在现有网络内部。  重述一遍，VNet 和子网一经部署，便可在 Azure 区域内保留为静态或永久资源。  

## <a name="next-steps"></a>后续步骤

* [使用 Azure Resource Manager 模板创建特定部署](/documentation/articles/virtual-machines-linux-cli-deploy-templates/)
* [直接使用 Azure CLI 命令创建自定义的 Linux VM 环境](/documentation/articles/virtual-machines-linux-create-cli-complete/)
* [使用模板在 Azure 上创建 Linux VM](/documentation/articles/virtual-machines-linux-create-ssh-secured-vm-from-template/)
<!--Update_Description: wording update-->