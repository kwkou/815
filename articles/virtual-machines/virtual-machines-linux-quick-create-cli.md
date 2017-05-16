<properties
    pageTitle="Azure 快速入门 - 创建 VM CLI | Azure"
    description="快速了解如何使用 Azure CLI 创建虚拟机。"
    services="virtual-machines-linux"
    documentationcenter="virtual-machines"
    author="neilpeterson"
    manager="timlt"
    editor="tysonn"
    tags="azure-resource-manager" />
<tags
    ms.assetid=""
    ms.service="virtual-machines-linux"
    ms.devlang="azurecli"
    ms.topic="hero-article"
    ms.tgt_pltfrm="vm-linux"
    ms.workload="infrastructure"
    ms.date="04/03/2017"
    wacn.date="05/15/2017"
    ms.author="nepeters"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="4c9cf54d89a91e980d749eaa80c4e235cda7ef25"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="create-a-linux-virtual-machine-with-the-azure-cli"></a>使用 Azure CLI 创建 Linux 虚拟机

Azure CLI 用于从命令行或脚本创建和管理 Azure 资源。 本指南详细介绍了如何使用 Azure CLI 部署运行 Ubuntu 16.04 LTS 的虚拟机。 部署服务器后，可通过 SSH 登录到 VM 以安装 NGINX。 

如果没有 Azure 订阅，可在开始前创建一个[试用帐户](/pricing/1rmb-trial/)。

另请确保已安装 Azure CLI。 有关详细信息，请参阅 [Azure CLI 安装指南](https://docs.microsoft.com/zh-cn/cli/azure/install-azure-cli)。 

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

创建 VM 后，Azure CLI 将显示类似于以下示例的信息。 记下 `publicIpAddress`。 此地址用于访问 VM。

    {
      "fqdns": "",
      "id": "/subscriptions/d5b9d4b7-6fc1-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM",
      "location": "chinanorth",
      "macAddress": "00-0D-3A-23-9A-49",
      "powerState": "VM running",
      "privateIpAddress": "10.0.0.4",
      "publicIpAddress": "40.68.254.142",
      "resourceGroup": "myResourceGroup"
    }

## <a name="open-port-80-for-web-traffic"></a>为 Web 流量打开端口 80 

默认情况下，仅允许通过 SSH 连接登录到 Azure 中部署的 Linux 虚拟机。 如果此 VM 将用作 Web 服务器，则需要从 Internet 打开端口 80。  需要使用单个命令打开所需的端口。  

    az vm open-port --port 80 --resource-group myResourceGroup --name myVM

## <a name="ssh-into-your-vm"></a>通过 SSH 连接到你的 VM

使用以下命令创建与虚拟机的 SSH 会话。 确保将 `<publicIpAddress>` 替换为你的虚拟机的相应公共 IP 地址。  在上例中，我们的 IP 地址为 `40.68.254.142`。

    ssh <publicIpAddress>

## <a name="install-nginx"></a>安装 NGINX

使用以下 bash 脚本更新包源并安装最新的 NGINX 包。 

    #!/bin/bash

    # update package source
    apt-get -y update

    # install NGINX
    apt-get -y install nginx

## <a name="view-the-ngix-welcome-page"></a>查看 NGIX 欢迎页

NGINX 已安装，并且现在已从 Internet 打开 VM 上的端口 80 - 可以使用所选的 Web 浏览器查看默认的 NGINX 欢迎页。 请务必使用前面记录的 `publicIpAddress` 访问默认页面。 

![NGINX 默认站点](./media/virtual-machines-linux-quick-create-cli/nginx.png) 

## <a name="delete-virtual-machine"></a>删除虚拟机

如果不再需要资源组、VM 和所有相关的资源，可以使用以下命令将其删除。

    az group delete --name myResourceGroup

## <a name="next-steps"></a>后续步骤

[创建高可用性虚拟机教程](/documentation/articles/virtual-machines-linux-create-cli-complete/)

[浏览 VM 部署 CLI 示例](/documentation/articles/virtual-machines-linux-cli-samples/)

<!--Update_Description: add "opening port 80" and "installing NGINX" -->