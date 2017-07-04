<properties
    pageTitle="Azure CLI 脚本示例 - 使用 DSC 创建具有 IIS 的 Windows Server 2016 VM | Azure"
    description="Azure CLI 脚本示例 - 使用 DSC 创建具有 IIS 的 Windows Server 2016 VM"
    services="virtual-machines-windows"
    documentationcenter="virtual-machines"
    author="rickstercdn"
    manager="timlt"
    editor="tysonn"
    tags=""
    translationtype="Human Translation" />
<tags
    ms.assetid=""
    ms.service="virtual-machines-Windows"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="vm-windows"
    ms.workload="infrastructure"
    ms.date="02/23/2017"
    wacn.date="04/17/2017"
    ms.author="rclaus"
    ms.sourcegitcommit="e0e6e13098e42358a7eaf3a810930af750e724dd"
    ms.openlocfilehash="95cdd94d8584792364099165970fb5e5b486bdc6"
    ms.lasthandoff="04/06/2017" />

# <a name="create-a-vm-with-iis-using-dsc"></a>使用 DSC 创建具有 IIS 的 VM

此脚本创建一个虚拟机，然后使用 Azure 虚拟机 DSC 自定义脚本扩展安装和配置 IIS。 

必要时，请使用 [Azure CLI 安装指南](https://docs.microsoft.com/zh-cn/cli/azure/install-azure-cli)中的说明安装 Azure CLI，然后运行 `az login` 创建与 Azure 的连接。

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

此示例在 Bash shell 中正常工作。 有关在 Windows 客户端上运行 Azure CLI 脚本的选项，请参阅[在 Windows 中运行 Azure CLI](/documentation/articles/virtual-machines-windows-cli-options/)。

## <a name="sample-script"></a>示例脚本

    #!/bin/bash

    # Update for your admin password
    AdminPassword=ChangeYourAdminPassword1

    # Create a resource group.
    az group create --name myResourceGroup --location chinanorth

    # Create a VM
    az vm create \
        --resource-group myResourceGroup \
        --name myVM \
        --image win2016datacenter \
        --admin-username azureuser \
        --admin-password $AdminPassword \
        --use-unmanaged-disk

    # Start a CustomScript extension to use a simple bash script to update, download and install WordPress and MySQL 
    az vm extension set \
       --name DSC \
       --publisher Microsoft.Powershell \
       --version 2.19 \
       --vm-name myVM \
       --resource-group myResourceGroup \
       --settings '{"ModulesURL":"https://github.com/Azure/azure-quickstart-templates/raw/master/dsc-extension-iis-server-windows-vm/ContosoWebsite.ps1.zip", "configurationFunction": "ContosoWebsite.ps1\\ContosoWebsite", "Properties": {"MachineName": "myVM"} }'

      # open port 80 to allow web traffic to host
      az vm open-port \
        --port 80 \
        --resource-group myResourceGroup \
        --name myVM

## <a name="clean-up-deployment"></a>清理部署 

运行以下命令来删除资源组、VM 和所有相关资源。

    az group delete --name myResourceGroup --yes

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令创建资源组、虚拟机和所有相关资源。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [az group create](https://docs.microsoft.com/zh-cn/cli/azure/group#create) | 创建用于存储所有资源的资源组。 |
| [az vm create](https://docs.microsoft.com/zh-cn/cli/azure/vm#create) | 创建虚拟机并将其连接到网卡、虚拟网络、子网和 NSG。 此命令还指定要使用的虚拟机映像和管理凭据。  |
| [az vm extension set](https://docs.microsoft.com/zh-cn/cli/azure/vm#create) | 将自定义脚本扩展添加到虚拟机，此扩展将调用脚本来安装 IIS。 |
| [az vm open-port](https://docs.microsoft.com/zh-cn/cli/azure/vm#open-port) | 创建网络安全组规则，以允许入站流量。 在此示例中，将为 HTTP 流量打开端口 80。 |
| [az group delete](https://docs.microsoft.com/zh-cn/cli/azure/vm/extension#set) | 删除资源组，包括所有嵌套的资源。 |

## <a name="next-steps"></a>后续步骤

有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](https://docs.microsoft.com/zh-cn/cli/azure/overview)。

可以在 [Azure Windows VM 文档](/documentation/articles/virtual-machines-windows-cli-samples/)中找到其他虚拟机 CLI 脚本示例。