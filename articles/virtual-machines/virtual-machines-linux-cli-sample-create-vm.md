<properties
    pageTitle="Azure CLI 脚本示例 - 创建 Linux VM | Azure"
    description="Azure CLI 脚本示例 - 创建 Linux VM"
    services="virtual-machines-linux"
    documentationcenter="virtual-machines"
    author="neilpeterson"
    manager="timlt"
    editor="tysonn"
    tags="azure-service-management"
    translationtype="Human Translation" />
<tags
    ms.assetid=""
    ms.service="virtual-machines-linux"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="vm-linux"
    ms.workload="infrastructure"
    ms.date="02/27/2017"
    wacn.date="04/17/2017"
    ms.author="nepeters"
    ms.sourcegitcommit="e0e6e13098e42358a7eaf3a810930af750e724dd"
    ms.openlocfilehash="2ab082d6ce513a44f206c0bde2241955c0efe078"
    ms.lasthandoff="04/06/2017" />

# <a name="create-a-fully-configured-virtual-machine"></a>创建完全配置的虚拟机

此脚本使用 Ubuntu 操作系统创建 Azure 虚拟机。 运行脚本后，可通过 SSH 访问虚拟机。

必要时，请使用 [Azure CLI 安装指南](https://docs.microsoft.com/zh-cn/cli/azure/install-azure-cli)中的说明安装 Azure CLI，然后运行 `az login` 创建与 Azure 的连接。

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

此示例在 Bash shell 中正常工作。 有关在 Windows 客户端上运行 Azure CLI 脚本的选项，请参阅[在 Windows 中运行 Azure CLI](/documentation/articles/virtual-machines-windows-cli-options/)。

## <a name="sample-script"></a>示例脚本

    #!/bin/bash

    # Create a resource group.
    az group create --name myResourceGroup --location chinanorth

    # Create a virtual network.
    az network vnet create --resource-group myResourceGroup --name myVnet --subnet-name mySubnet

    # Create a public IP address.
    az network public-ip create --resource-group myResourceGroup --name myPublicIP

    # Create a network security group.
    az network nsg create --resource-group myResourceGroup --name myNetworkSecurityGroup

    # Create a virtual network card and associate with public IP address and NSG.
    az network nic create \
      --resource-group myResourceGroup \
      --name myNic \
      --vnet-name myVnet \
      --subnet mySubnet \
      --network-security-group myNetworkSecurityGroup \
      --public-ip-address myPublicIP

    # Create a new virtual machine, this creates SSH keys if not present.
    az vm create --resource-group myResourceGroup --name myVM --nics myNic --image UbuntuLTS --generate-ssh-keys --use-unmanaged-disk

    # Open port 22 to allow SSh traffic to host.
    az vm open-port --port 22 --resource-group myResourceGroup --name myVM

## <a name="clean-up-deployment"></a>清理部署 

运行以下命令来删除资源组、VM 和所有相关资源。

    az group delete --name myResourceGroup

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令创建资源组、虚拟机和所有相关资源。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [az group create](https://docs.microsoft.com/zh-cn/cli/azure/group#create) | 创建用于存储所有资源的资源组。 |
| [az network vnet create](https://docs.microsoft.com/zh-cn/cli/azure/network/vnet#create) | 创建 Azure 虚拟网络和子网。 |
| [az network public-ip create](https://docs.microsoft.com/zh-cn/cli/azure/network/public-ip#create) | 使用静态 IP 地址和关联的 DNS 名称创建公共 IP 地址。 |
| [az network nsg create](https://docs.microsoft.com/zh-cn/cli/azure/network/nsg#create) | 创建网络安全组 (NSG)，这是 Internet 和虚拟机之间的安全边界。 |
| [az network nsg rule create](https://docs.microsoft.com/zh-cn/cli/azure/network/nsg/rule#create) | 创建 NSG 规则以允许入站流量。 在此示例中，将为 SSH 流量打开端口 22。 |
| [az network nic create](https://docs.microsoft.com/zh-cn/cli/azure/network/nic#create) | 创建虚拟网卡并将其连接到虚拟网络、子网和 NSG。 |
| [az vm create](https://docs.microsoft.com/zh-cn/cli/azure/vm#create) | 创建虚拟机并将其连接到网卡、虚拟网络、子网和 NSG。 此命令还指定要使用的虚拟机映像和管理凭据。  |
| [az group delete](https://docs.microsoft.com/zh-cn/cli/azure/vm/extension#set) | 删除资源组，包括所有嵌套的资源。 |

## <a name="next-steps"></a>后续步骤

有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](https://docs.microsoft.com/zh-cn/cli/azure/overview)。

可以在 [Azure Linux VM 文档](/documentation/articles/virtual-machines-linux-cli-samples/)中找到其他虚拟机 CLI 脚本示例。