<properties
    pageTitle="使用 Azure CLI 管理 Linux 虚拟机 | Azure"
    description="教程 - 使用 Azure CLI 管理 Linux 虚拟机"
    services="virtual-machines-linux"
    documentationcenter="virtual-machines"
    author="neilpeterson"
    manager="timlt"
    editor="tysonn"
    tags="azure-service-management" />
<tags
    ms.assetid=""
    ms.service="virtual-machines-linux"
    ms.devlang="azurecli"
    ms.topic="article"
    ms.tgt_pltfrm="vm-linux"
    ms.workload="infrastructure"
    ms.date="03/28/2017"
    wacn.date="05/15/2017"
    ms.author="nepeters"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="a2084798dc02ec48962e8f31cfcc70d04ed399fa"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="manage-linux-virtual-machines-with-the-azure-cli"></a>使用 Azure CLI 管理 Linux 虚拟机

本教程将创建一个虚拟机并执行常见的管理任务，例如添加磁盘、自动化软件安装以及创建虚拟机快照。 

若要完成本教程，请确保已安装最新的 [Azure CLI 2.0](https://docs.microsoft.com/zh-cn/cli/azure/install-azure-cli)。

## <a name="step-1---log-in-to-azure"></a>步骤 1 - 登录 Azure

首先，使用 [az login](https://docs.microsoft.com/zh-cn/cli/azure/#login) 命令打开终端并登录 Azure 订阅。

    az login

## <a name="step-2---create-resource-group"></a>步骤 2 - 创建资源组

使用 [az group create](https://docs.microsoft.com/zh-cn/cli/azure/group#create) 命令创建资源组。 

Azure 资源组是在其中部署和管理 Azure 资源的逻辑容器。 必须在创建虚拟机前创建资源组。 在此示例中，在 `chinanorth` 区域中创建了名为 `myResourceGroup` 的资源组。 

    az group create --name myResourceGroup --location chinanorth

## <a name="step-3---prepare-configuration"></a>步骤 3 - 准备配置文件

部署虚拟机时，可使用 **cloud-init** 自动执行配置，如安装软件包、创建文件和运行脚本。 在本教程中，有两个项目为自动配置：

- 安装 NGINX Web 服务器
- 在 VM 上预配第二个磁盘

由于 **cloud-init** 配置在 VM 部署时间进行，因此需在创建虚拟机前定义 **cloud-init** 配置。

创建名为 `cloud-init.txt` 的文件，并将以下内容复制到其中。 此配置将安装 NGINX 包，并将命令运行到共振峰，安装第二个磁盘。

    #cloud-config
    package_upgrade: true
    packages:
      - nginx
    runcmd:
      - (echo n; echo p; echo 1; echo ; echo ; echo w) | sudo fdisk /dev/sdc
      - sudo mkfs -t ext4 /dev/sdc1
      - sudo mkdir /datadrive
      - sudo mount /dev/sdc1 /datadrive

## <a name="step-4---create-virtual-machine"></a>步骤 4 - 创建虚拟机

使用 [az vm create](https://docs.microsoft.com/zh-cn/cli/azure/vm#create) 命令创建虚拟机。 

创建虚拟机时，可使用多个选项，例如操作系统映像、磁盘大小调整和管理凭据。 在此示例中，创建了一个名为 `myVM` 的运行 Ubuntu 的虚拟机。`--custom-data` 参数接受 cloud-init 配置，并将其暂存在 VM 上。 最后，如果不存在 SSH 密钥，则会进行创建。

    az vm create \
      --resource-group myResourceGroup \
      --name myVM \
      --image Canonical:UbuntuServer:14.04.3-LTS:latest \
      --generate-ssh-keys \
      --use-unmanaged-disk \
      --custom-data cloud-init.txt

创建 VM 后，Azure CLI 会输出以下信息。 记下公共 IP 地址，访问虚拟机时需使用该地址。 

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

部署 VM 后，可能需要几分钟才能完成 **cloud-init** 配置。 

## <a name="step-5---configure-firewall"></a>步骤 5 - 配置防火墙

Azure [网络安全组](/documentation/articles/virtual-networks-nsg/) (NSG) 控制一个或多个虚拟机的入站和出站流量。 网络安全组规则允许或拒绝特定端口上或端口范围内的网络流量。 这些规则还可包括源地址前缀，这样只有源自预定义源的流量才可与虚拟机进行通信。

在上一节中，已安装了 NGINX Web 服务器。 如果没有网络安全组规则允许端口 80 上的入站流量，则无法在 Internet 上访问 Web 服务器。 此步骤将介绍如何创建 NSG 规则，允许端口 80 上的入站连接。

### <a name="create-nsg-rule"></a>创建 NSG 规则

若要创建入站 NSG 规则，请使用 [az vm open-port](https://docs.microsoft.com/zh-cn/cli/azure/vm#open-port) 命令。 以下示例将打开虚拟机的 `80` 端口。

    az vm open-port --port 80 --resource-group myResourceGroup --name myVM 

现在浏览到虚拟机的公共 IP 地址。 如果具有 NSG 规则，将显示默认 NGINX 站点。

![NGINX 默认站点](./media/virtual-machines-linux-tutorial-manage-vm/nginx.png)  

## <a name="step-7---management-tasks"></a>步骤 6 - 管理任务

在虚拟机生命周期中，你可能需要运行管理任务，例如启动、停止或删除虚拟机。 此外，可能还需要创建脚本来自动执行重复或复杂的任务。 使用 Azure CLI，可从命令行或脚本运行许多常见的管理任务。 

### <a name="get-ip-address"></a>获取 IP 地址

此命令返回虚拟机的私有 IP 地址和公共 IP 地址。  

    az vm list-ip-addresses --resource-group myResourceGroup --name myVM

### <a name="resize-virtual-machine"></a>调整虚拟机大小

若要调整 Azure 虚拟机的大小，需要知道所选 Azure 区域中可用的大小的名称。 可使用 [az vm list-sizes](https://docs.microsoft.com/zh-cn/cli/azure/vm#list-sizes) 命令找到这些大小。

    az vm list-sizes --location chinanorth --output table

可使用 [az vm resize](https://docs.microsoft.com/zh-cn/cli/azure/vm#resize) 命令调整虚拟机大小。 

    az vm resize -g myResourceGroup -n myVM --size Standard_F4s

### <a name="stop-virtual-machine"></a>停止虚拟机

    az vm stop --resource-group myResourceGroup --name myVM

### <a name="start-virtual-machine"></a>启动虚拟机

    az vm start --resource-group myResourceGroup --name myVM

### <a name="delete-resource-group"></a>删除资源组

删除资源组会删除其包含的所有资源。

    az group delete --name myResourceGroup

## <a name="next-steps"></a>后续步骤
本教程使用单个 Azure 资源创建单个虚拟机。 下一个教程基于这些概念，创建高度可用的应用程序，这些应用程序负载均衡且具有针对维护事件的弹性。 继续学习下一个教程 - [在 Azure 中的 Linux 虚拟机上生成负载均衡的高可用性应用程序](/documentation/articles/virtual-machines-linux-tutorial-load-balance-nodejs/)。

示例 - [Azure CLI 示例脚本](/documentation/articles/virtual-machines-windows-cli-samples/)