<properties
    pageTitle="Azure CLI 脚本示例 - 安装 IIS | Azure"
    description="Azure CLI 脚本示例 - 安装 IIS"
    services="virtual-machines-Windows"
    documentationcenter="virtual-machines"
    author="neilpeterson"
    manager="timlt"
    editor="tysonn"
    tags=""
    translationtype="Human Translation" />
<tags
    ms.assetid=""
    ms.service="virtual-machines-Windows"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="vm-Windows"
    ms.workload="infrastructure"
    ms.date="02/28/2017"
    wacn.date="04/17/2017"
    ms.author="nepeters"
    ms.sourcegitcommit="e0e6e13098e42358a7eaf3a810930af750e724dd"
    ms.openlocfilehash="9a3fec28b7ca3ad832123c1a7c1f6ae8618ed739"
    ms.lasthandoff="04/06/2017" />

# <a name="quick-create-a-virtual-machine-with-the-azure-cli"></a>使用 Azure CLI 快速创建虚拟机

此脚本创建一个运行 Windows Server 2016 的 Azure 虚拟机，然后使用 Azure 虚拟机自定义脚本扩展安装 IIS。 运行此脚本后，可通过虚拟机的公共 IP 地址访问默认 IIS 网站。

必要时，请使用 [Azure CLI 安装指南](https://docs.microsoft.com/zh-cn/cli/azure/install-azure-cli)中的说明安装 Azure CLI，然后运行 `az login` 创建与 Azure 的连接。

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

此示例在 Bash Shell 中正常工作。 有关在 Windows 上运行 Azure CLI 脚本的选项，请参阅[在 Windows 中运行 Azure CLI](/documentation/articles/virtual-machines-windows-cli-options/)。

## <a name="sample-script"></a>示例脚本

    #!/bin/bash

    # Update for your admin password
    AdminPassword=ChangeYourAdminPassword1

    # Create a resource group.
    az group create --name myResourceGroup --location chinanorth

    # Create a virtual machine. 
    az vm create \
        --resource-group myResourceGroup \
        --name myVM \
        --image win2016datacenter \
        --admin-username azureuser \
        --admin-password $AdminPassword \
        --use-unmanaged-disk

    # Open port 80 to allow web traffic to host.
    az vm open-port --port 80 --resource-group myResourceGroup --name myVM 

    # Use CustomScript extension to install Apache.
    az vm extension set \
      --publisher Microsoft.Compute \
      --version 1.8 \
      --name CustomScriptExtension \
      --vm-name myVM \
      --resource-group myResourceGroup \
      --settings '{"commandToExecute":"powershell.exe Install-WindowsFeature -Name Web-Server"}'

## <a name="clean-up-deployment"></a>清理部署 

运行以下命令来删除资源组、VM 和所有相关资源。

    az group delete --name myResourceGroup --yes

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令创建资源组、虚拟机和所有相关资源。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [az group create](https://docs.microsoft.com/zh-cn/cli/azure/group#create) | 创建用于存储所有资源的资源组。 |
| [az vm create](https://docs.microsoft.com/zh-cn/cli/azure/vm#create) | 创建虚拟机并将其连接到网卡、虚拟网络、子网和网络安全组。 此命令还指定要使用的虚拟机映像和管理凭据。  |
| [az vm open-port](https://docs.microsoft.com/zh-cn/cli/azure/network/nsg/rule#create) | 创建网络安全组规则，以允许入站流量。 在此示例中，将为 HTTP 流量打开端口 80。 |
| [azure vm extension set](https://docs.microsoft.com/zh-cn/cli/azure/vm/extension#set) | 将虚拟机扩展添加到 VM 并运行该扩展。 在此示例中，使用自定义脚本扩展来安装 IIS。|
| [az group delete](https://docs.microsoft.com/zh-cn/cli/azure/vm/extension#set) | 删除资源组，包括所有嵌套的资源。 |

## <a name="next-steps"></a>后续步骤

有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](https://docs.microsoft.com/zh-cn/cli/azure/overview)。

可以在 [Azure Windows VM 文档](/documentation/articles/virtual-machines-windows-cli-samples/)中找到其他虚拟机 CLI 脚本示例。