<properties
    pageTitle="Azure CLI 脚本示例 - 使用内部和外部 NSG 创建两个 VM | Azure"
    description="Azure CLI 脚本示例 - 使用内部和外部 NSG 创建两个 VM"
    services="virtual-machines-windows"
    documentationcenter="virtual-machines"
    author="rickstercdn"
    manager="timlt"
    editor="tysonn"
    tags=""
    translationtype="Human Translation" />
<tags
    ms.assetid=""
    ms.service="virtual-machines-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="vm-windows"
    ms.workload="infrastructure"
    ms.date="02/23/2017"
    wacn.date="04/17/2017"
    ms.author="rclaus"
    ms.sourcegitcommit="e0e6e13098e42358a7eaf3a810930af750e724dd"
    ms.openlocfilehash="25fac68683b45d7d0c9fd3a7ea8198b6388ac291"
    ms.lasthandoff="04/06/2017" />

# <a name="secure-network-traffic-between-virtual-machines"></a>保护虚拟机之间的网络流量

此脚本创建两个虚拟机，并保护这两个虚拟机的传入流量。 一个虚拟机可在 Internet 上访问，其网络安全组 (NSG) 配置为允许端口 3389 和端口 80 上的流量。 第二个虚拟机无法在 Internet 上访问，其 NSG 配置为仅允许来自第一个虚拟机的流量。 

必要时，请使用 [Azure CLI 安装指南](https://docs.microsoft.com/zh-cn/cli/azure/install-azure-cli)中的说明安装 Azure CLI，然后运行 `az login` 创建与 Azure 的连接。

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

此示例在 Bash shell 中正常工作。 有关在 Windows 上运行 Azure CLI 脚本的选项，请参阅[在 Windows 中运行 Azure CLI](/documentation/articles/virtual-machines-windows-cli-options/)。

## <a name="sample-script"></a>示例脚本

    #!/bin/bash

    # Update for your admin password
    AdminPassword=ChangeYourAdminPassword1

    # Create a resource group.
    az group create --name myResourceGroup --location chinanorth

    # Create a virtual network and subnet (front end).
    az network vnet create --resource-group myResourceGroup --name myVnet --address-prefix 192.168.0.0/16 \
    --subnet-name mySubnetFrontEnd --subnet-prefix 192.168.1.0/24

    # Create a subnet (back end) and associate with virtual network. 
    az network vnet subnet create --resource-group myResourceGroup --vnet-name myVnet \
      --name mySubnetBackEnd --address-prefix 192.168.2.0/24

    # Create a virtual machine. 
    az vm create --resource-group myResourceGroup --name myVMFrontEnd --image win2016datacenter \
      --admin-username azureuser --admin-password $AdminPassword --vnet-name myVnet --subnet mySubnetFrontEnd \
       --nsg myNetworkSecurityGroupFrontEnd --no-wait --use-unmanaged-disk

    # Create a virtual machine without a public IP address.
    az vm create --resource-group myResourceGroup --name myVMBackEnd --image win2016datacenter \
      --admin-username azureuser --admin-password $AdminPassword --public-ip-address "" --vnet-name myVnet \
      --subnet mySubnetBackEnd --nsg myNetworkSecurityGroupBackEnd --use-unmanaged-disk

    # Get nsg rule name.
    nsgrule=$(az network nsg rule list --resource-group myResourceGroup --nsg-name myNetworkSecurityGroupBackEnd --query [0].name -o tsv)

    # Update backend network security group rule to limit source prefix.
    az network nsg rule update --resource-group myResourceGroup --nsg-name myNetworkSecurityGroupBackEnd \
      --name $nsgrule --protocol tcp --direction inbound --priority 1000 \
      --source-address-prefix 192.168.1.0/24 --source-port-range '*' --destination-address-prefix '*' \
      --destination-port-range 3389 --access allow

## <a name="clean-up-deployment"></a>清理部署 

运行以下命令来删除资源组、VM 和所有相关资源。

    az group delete --name myResourceGroup --yes

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令创建资源组、虚拟机和所有相关资源。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [az group create](https://docs.microsoft.com/zh-cn/cli/azure/group#create) | 创建用于存储所有资源的资源组。 |
| [az network vnet create](https://docs.microsoft.com/zh-cn/cli/azure/network/vnet#create) | 创建 Azure 虚拟网络和子网。 |
| [az network vnet subnet create](https://docs.microsoft.com/zh-cn/cli/azure/network/vnet/subnet#create) | 创建子网。 |
| [az vm create](https://docs.microsoft.com/zh-cn/cli/azure/vm#create) | 创建虚拟机并将其连接到网卡、虚拟网络、子网和 NSG。 此命令还指定要使用的虚拟机映像和管理凭据。  |
| [az network nsg rule update](https://docs.microsoft.com/zh-cn/cli/azure/network/nsg/rule#update) | 更新 NSG 规则。 在本例中，将更新后端规则，仅从前端子网传递流量。 |
| [az network nsg rule list](https://docs.microsoft.com/zh-cn/cli/azure/network/nsg/rule#list) | 返回有关网络安全组规则的信息。 在此示例中，规则名称存储在变量中，以便以后在脚本中使用。 |
| [az group delete](https://docs.microsoft.com/zh-cn/cli/azure/vm/extension#set) | 删除资源组，包括所有嵌套的资源。 |

## <a name="next-steps"></a>后续步骤

有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](https://docs.microsoft.com/zh-cn/cli/azure/overview)。

可以在 [Azure Windows VM 文档](/documentation/articles/virtual-machines-windows-cli-samples/)中找到其他虚拟机 CLI 脚本示例。