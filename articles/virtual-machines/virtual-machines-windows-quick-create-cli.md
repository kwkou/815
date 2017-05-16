<properties
    pageTitle="Azure 快速入门 - 创建 Windows VM CLI | Azure"
    description="快速了解如何使用 Azure CLI 创建 Windows 虚拟机。"
    services="virtual-machines-windows"
    documentationcenter="virtual-machines"
    author="neilpeterson"
    manager="timlt"
    editor="tysonn"
    tags="azure-resource-manager" />
<tags
    ms.assetid=""
    ms.service="virtual-machines-windows"
    ms.devlang="na"
    ms.topic="hero-article"
    ms.tgt_pltfrm="vm-windows"
    ms.workload="infrastructure"
    ms.date="04/03/2017"
    wacn.date="05/15/2017"
    ms.author="nepeters"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="a6221ad9eac6ec67ac02874e3172f8a6bbd07159"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="create-a-windows-virtual-machine-with-the-azure-cli"></a>使用 Azure CLI 创建 Windows 虚拟机

Azure CLI 用于从命令行或脚本创建和管理 Azure 资源。 本指南详细介绍如何使用 Azure CLI 部署运行 Windows Server 2016 的虚拟机。 部署完成后，我们将连接到服务器并安装 IIS。

如果没有 Azure 订阅，可在开始前创建一个[试用帐户](/pricing/1rmb-trial/)。

另请确保已安装 Azure CLI。 有关详细信息，请参阅 [Azure CLI 安装指南](https://docs.microsoft.com/zh-cn/cli/azure/install-azure-cli)。

## <a name="log-in-to-azure"></a>登录 Azure 

使用 [az login](https://docs.microsoft.com/zh-cn/cli/azure/#login) 命令登录到 Azure 订阅，并按照屏幕上的说明进行操作。

    az login

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

## <a name="create-a-resource-group"></a>创建资源组

使用 [az group create](https://docs.microsoft.com/zh-cn/cli/azure/group#create) 创建资源组。 Azure 资源组是在其中部署和管理 Azure 资源的逻辑容器。 

以下示例在 `chinanorth` 位置创建名为 `myResourceGroup` 的资源组。

    az group create --name myResourceGroup --location chinanorth

## <a name="create-virtual-machine"></a>创建虚拟机

使用 [az vm create](https://docs.microsoft.com/zh-cn/cli/azure/vm#create) 创建 VM。 

以下示例创建一个名为 `myVM` 的 VM。 此示例使用 `azureuser` 作为管理用户名，使用 ` myPassword12` 作为密码。 更新这些值，使其适用于你的环境。 创建与虚拟机的连接时，需要这些值。

    az vm create --resource-group myResourceGroup --name myVM --image win2016datacenter --admin-username azureuser --admin-password myPassword12 --use-unmanaged-disk

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

## <a name="open-port-80-for-web-traffic"></a>为 Web 流量打开端口 80 

默认情况下，仅允许通过 RDP 连接登录到 Azure 中部署的 Windows 虚拟机。 如果此 VM 将用作 Web 服务器，则需要从 Internet 打开端口 80。  需要使用单个命令打开所需的端口。  

    az vm open-port --port 80 --resource-group myResourceGroup --name myVM

## <a name="connect-to-virtual-machine"></a>连接到虚拟机

使用以下命令创建与虚拟机的远程桌面会话。 将 IP 地址替换为你的虚拟机的公共 IP 地址。 出现提示时，输入创建虚拟机时使用的凭据。

    mstsc /v:<Public IP Address>

## <a name="install-iis-using-powershell"></a>使用 PowerShell 安装 IIS

现在，你已登录到 Azure VM，可以使用单行 PowerShell 安装 IIS，并启用本地防火墙规则以允许 Web 流量。  打开 PowerShell 提示符并运行以下命令：

    Install-WindowsFeature -name Web-Server -IncludeManagementTools

## <a name="view-the-iis-welcome-page"></a>查看 IIS 欢迎页

IIS 已安装，并且现在已从 Internet 打开 VM 上的端口 80 - 你可以使用所选的 Web 浏览器查看默认的 IIS 欢迎页。 请务必使用前面记录的 `publicIpAddress` 访问默认页面。 

![IIS 默认站点](./media/virtual-machines-windows-quick-create-powershell/default-iis-website.png) 
## <a name="delete-virtual-machine"></a>删除虚拟机

如果不再需要资源组、VM 和所有相关的资源，可以使用以下命令将其删除。

    az group delete --name myResourceGroup

## <a name="next-steps"></a>后续步骤

[安装角色和配置防火墙教程](/documentation/articles/virtual-machines-windows-hero-role/)

[浏览 VM 部署 CLI 示例](/documentation/articles/virtual-machines-windows-cli-samples/)

<!--Update_Description: add "opening port 80" and "installing IIS"-->