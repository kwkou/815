<properties
    pageTitle="Azure CLI 脚本示例 - 使用 NLB 创建 Linux VM | Azure"
    description="Azure CLI 脚本示例 - 使用 NLB 创建 Linux VM"
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
    ms.openlocfilehash="923ad59539b0e9664c1eb5f1e4c070486d38a45e"
    ms.lasthandoff="04/06/2017" />

# <a name="create-a-highly-available-vm"></a>创建高度可用 VM

此脚本示例创建运行多个 Ubuntu 虚拟机（使用高度可用且负载均衡的配置进行配置）所需的所有项。 运行脚本后，即可拥有已加入到 Azure 可用性集并可通过 Azure 负载均衡器访问的 3 个虚拟机。 

必要时，请使用 [Azure CLI 安装指南](https://docs.microsoft.com/zh-cn/cli/azure/install-azure-cli)中的说明安装 Azure CLI，然后运行 `az login` 创建与 Azure 的连接。

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

此示例在 Bash shell 中正常工作。 有关在 Windows 客户端上运行 Azure CLI 脚本的选项，请参阅[在 Windows 中运行 Azure CLI](/documentation/articles/virtual-machines-windows-cli-options/)。

## <a name="sample-script"></a>示例脚本

    #!/bin/bash

    # Create a resource group.
    az group create --name myResourceGroup --location chinanorth

    # Create a virtual network.
    az network vnet create --resource-group myResourceGroup --location chinanorth --name myVnet --subnet-name mySubnet

    # Create a public IP address.
    az network public-ip create --resource-group myResourceGroup --name myPublicIP

    # Create an Azure Network Load Balancer.
    az network lb create --resource-group myResourceGroup --name myLoadBalancer --public-ip-address myPublicIP \
      --frontend-ip-name myFrontEndPool --backend-pool-name myBackEndPool

    # Creates an NLB probe on port 80.
    az network lb probe create --resource-group myResourceGroup --lb-name myLoadBalancer \
      --name myHealthProbe --protocol tcp --port 80

    # Creates an NLB rule for port 80.
    az network lb rule create --resource-group myResourceGroup --lb-name myLoadBalancer --name myLoadBalancerRuleWeb \
      --protocol tcp --frontend-port 80 --backend-port 80 --frontend-ip-name myFrontEndPool \
      --backend-pool-name myBackEndPool --probe-name myHealthProbe

    # Create three NAT rules for port 22.
    for i in `seq 1 3`; do
      az network lb inbound-nat-rule create \
        --resource-group myResourceGroup --lb-name myLoadBalancer \
        --name myLoadBalancerRuleSSH$i --protocol tcp \
        --frontend-port 422$i --backend-port 22 \
        --frontend-ip-name myFrontEndPool
    done

    # Create a network security group
    az network nsg create --resource-group myResourceGroup --name myNetworkSecurityGroup

    # Create a network security group rule for port 22.
    az network nsg rule create --resource-group myResourceGroup --nsg-name myNetworkSecurityGroup --name myNetworkSecurityGroupRuleSSH \
      --protocol tcp --direction inbound --source-address-prefix '*' --source-port-range '*'  \
      --destination-address-prefix '*' --destination-port-range 22 --access allow --priority 1000

    # Create a network security group rule for port 80.
    az network nsg rule create --resource-group myResourceGroup --nsg-name myNetworkSecurityGroup --name myNetworkSecurityGroupRuleHTTP \
    --protocol tcp --direction inbound --priority 1001 --source-address-prefix '*' --source-port-range '*' \
    --destination-address-prefix '*' --destination-port-range 80 --access allow --priority 2000

    # Create three virtual network cards and associate with public IP address and NSG.
    for i in `seq 1 3`; do
      az network nic create \
        --resource-group myResourceGroup --name myNic$i \
        --vnet-name myVnet --subnet mySubnet \
        --network-security-group myNetworkSecurityGroup --lb-name myLoadBalancer \
        --lb-address-pools myBackEndPool --lb-inbound-nat-rules myLoadBalancerRuleSSH$i
    done

    # Create an availability set.
    az vm availability-set create --resource-group myResourceGroup --name myAvailabilitySet --platform-fault-domain-count 3 --platform-update-domain-count 3

    # Create three virtual machines, this creates SSH keys if not present.
    for i in `seq 1 3`; do
      az vm create \
        --resource-group myResourceGroup \
        --name myVM$i \
        --availability-set myAvailabilitySet \
        --nics myNic$i \
        --image UbuntuLTS \
        --generate-ssh-keys \
        --no-wait \
        --use-unmanaged-disk
    done

## <a name="clean-up-deployment"></a>清理部署 

运行以下命令来删除资源组、VM 和所有相关资源。

    az group delete --name myResourceGroup

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令创建资源组、虚拟机、可用性集、负载均衡器和所有相关资源。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [az group create](https://docs.microsoft.com/zh-cn/cli/azure/group#create) | 创建用于存储所有资源的资源组。 |
| [az network vnet create](https://docs.microsoft.com/zh-cn/cli/azure/network/vnet#create) | 创建 Azure 虚拟网络和子网。 |
| [az network public-ip create](https://docs.microsoft.com/zh-cn/cli/azure/network/public-ip#create) | 使用静态 IP 地址和关联的 DNS 名称创建公共 IP 地址。 |
| [az network lb create](https://docs.microsoft.com/zh-cn/cli/azure/network/lb#create) | 创建 Azure 网络负载均衡器 (NLB)。 |
| [az network lb probe create](https://docs.microsoft.com/zh-cn/cli/azure/network/lb/probe#create) | 创建 NLB 探测。 NLB 探测用于监视 NLB 集中的每个 VM。 如果任何 VM 无法访问，流量将不会路由到该 VM。 |
| [az network lb rule create](https://docs.microsoft.com/zh-cn/cli/azure/network/lb/rule#create) | 创建 NLB 规则。 在此示例中，将为端口 80 创建一个规则。 当 HTTP 流量到达 NLB 时，它将路由到 NLB 集中的一个 VM 的端口 80。 |
| [az network lb inbound-nat-rule create](https://docs.microsoft.com/zh-cn/cli/azure/network/lb/inbound-nat-rule#create) | 创建 NLB 网络地址转换 (NAT) 规则。  NAT 规则将 NLB 的端口映射到 VM 上的端口。 在此示例中，将为发往 NLB 集中的每个 VM 的 SSH 流量创建 NAT 规则。  |
| [az network nsg create](https://docs.microsoft.com/zh-cn/cli/azure/network/nsg#create) | 创建网络安全组 (NSG)，这是 Internet 和虚拟机之间的安全边界。 |
| [az network nsg rule create](https://docs.microsoft.com/zh-cn/cli/azure/network/nsg/rule#create) | 创建 NSG 规则以允许入站流量。 在此示例中，将为 SSH 流量打开端口 22。 |
| [az network nic create](https://docs.microsoft.com/zh-cn/cli/azure/network/nic#create) | 创建虚拟网卡并将其连接到虚拟网络、子网和 NSG。 |
| [az vm availability-set create](https://docs.microsoft.com/zh-cn/cli/azure/network/lb/rule#create) | 创建可用性集。 可用性集通过将虚拟机分布到各个物理资源上（以便发生故障时，不会影响整个集）来确保应用程序运行时间。 |
| [az vm create](https://docs.microsoft.com/zh-cn/cli/azure/vm/availability-set#create) | 创建虚拟机并将其连接到网卡、虚拟网络、子网和 NSG。 此命令还指定要使用的虚拟机映像和管理凭据。  |
| [az group delete](https://docs.microsoft.com/zh-cn/cli/azure/vm/extension#set) | 删除资源组，包括所有嵌套的资源。 |

## <a name="next-steps"></a>后续步骤

有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](https://docs.microsoft.com/zh-cn/cli/azure/overview)。

可以在 [Azure Linux VM 文档](/documentation/articles/virtual-machines-linux-cli-samples/)中找到其他虚拟机 CLI 脚本示例。