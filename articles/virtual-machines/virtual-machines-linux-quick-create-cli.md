<properties
    pageTitle="Azure 快速入门 - 创建 VM CLI | Azure"
    description="快速了解如何使用 Azure CLI 创建虚拟机。"
    services="virtual-machines-linux"
    documentationcenter="virtual-machines"
    author="neilpeterson"
    manager="timlt"
    editor="tysonn"
    tags="azure-resource-manager"
    translationtype="Human Translation" />
<tags
    ms.assetid=""
    ms.service="virtual-machines-linux"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="vm-linux"
    ms.workload="infrastructure"
    ms.date="03/10/2017"
    wacn.date="04/24/2017"
    ms.author="nepeters"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="b227647cb35c6d8c5ba4b4f53843c8375534fbd8"
    ms.lasthandoff="04/14/2017" />

# <a name="create-a-linux-virtual-machine-with-the-azure-cli"></a>使用 Azure CLI 创建 Linux 虚拟机

Azure CLI 用于从命令行或脚本创建和管理 Azure 资源。 本指南详细介绍了如何使用 Azure CLI 部署运行 Ubuntu 16.04 LTS 的虚拟机。

在开始之前，请确保已安装 Azure CLI。 有关详细信息，请参阅 [Azure CLI 安装指南](https://docs.microsoft.com/zh-cn/cli/azure/install-azure-cli)。 

## <a name="log-in-to-azure"></a>登录 Azure 

使用 [az login](https://docs.microsoft.com/zh-cn/cli/azure/#login) 命令登录到 Azure 订阅，并按照屏幕上的说明进行操作。

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

    az login

## <a name="create-a-resource-group"></a>创建资源组

使用 [az group create](https://docs.microsoft.com/zh-cn/cli/azure/group#create) 命令创建资源组。 Azure 资源组是在其中部署和管理 Azure 资源的逻辑容器。 

以下示例在 `chinanorth` 位置创建名为 `myResourceGroup` 的资源组。

    az group create --name myResourceGroup --location chinanorth

## <a name="create-virtual-machine"></a>创建虚拟机

使用 [az vm create](https://docs.microsoft.com/zh-cn/cli/azure/vm#create) 命令创建 VM。 

下面的示例创建一个名为 `myVM` 的 VM，并且在默认密钥位置中不存在 SSH 密钥时创建这些密钥。 若要使用特定的一组密钥，请使用 `--ssh-key-value` 选项。  

    az vm create --resource-group myResourceGroup --name myVM --image UbuntuLTS --generate-ssh-keys --use-unmanaged-disk

创建 VM 后，Azure CLI 将显示类似于以下示例的信息。 记下公共 IP 地址。 此地址用于访问 VM。

    {
      "fqdns": "",
      "id": "/subscriptions/d5b9d4b7-6fc1-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM",
      "location": "chinanorth",
      "macAddress": "00-0D-3A-23-9A-49",
      "powerState": "VM running",
      "privateIpAddress": "10.0.0.4",
      "publicIpAddress": "52.174.34.95",
      "resourceGroup": "myResourceGroup"
    }

## <a name="connect-to-virtual-machine"></a>连接到虚拟机

使用以下命令创建与虚拟机的 SSH 会话。 将 IP 地址替换为你的虚拟机的公共 IP 地址。

    ssh <Public IP Address>

## <a name="delete-virtual-machine"></a>删除虚拟机

如果不再需要资源组、VM 和所有相关的资源，可以使用以下命令将其删除。

    az group delete --name myResourceGroup

## <a name="next-steps"></a>后续步骤

[创建高可用性虚拟机教程](/documentation/articles/virtual-machines-linux-create-cli-complete/)

[浏览 VM 部署 CLI 示例](/documentation/articles/virtual-machines-linux-cli-samples/)
<!--Update_Description: wording update-->