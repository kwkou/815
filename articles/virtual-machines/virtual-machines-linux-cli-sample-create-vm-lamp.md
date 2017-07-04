<properties
    pageTitle="Azure CLI 脚本示例 - 在负载均衡虚拟机规模集中部署 LAMP 堆栈 | Azure"
    description="使用自定义脚本扩展在 Azure 上的负载均衡虚拟机规模集中部署 LAMP 堆栈。"
    services="virtual-machines-linux"
    documentationcenter="virtual-machines"
    author="allclark"
    manager="douge"
    editor="tysonn"
    tags="azure-service-management" />
<tags
    ms.assetid=""
    ms.service="virtual-machines-linux"
    ms.devlang="azurecli"
    ms.topic="article"
    ms.tgt_pltfrm="vm-linux"
    ms.workload="infrastructure"
    ms.date="04/05/2017"
    wacn.date=""
    ms.author="allclark"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="66b5de20c1a965a5ca3bae7a4b78e43929f886bd"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="deploy-the-lamp-stack-in-a-load-balanced-virtual-machine-scale-set"></a>在负载均衡虚拟机规模集中部署 LAMP 堆栈

此示例创建一个虚拟机规模集，并应用运行自定义脚本的扩展在规模集中每个虚拟机上部署 LAMP 堆栈。

[AZURE.INCLUDE [sample-cli-install](../../includes/sample-cli-install.md)]

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

## <a name="sample-script"></a>示例脚本

    #!/bin/bash

    # Create the resource group if it doesn't exist
    az group create -n myResourceGroup -l chinanorth

    # Build the Scale Set
    az vmss create -n myScaleSet -g myResourceGroup --public-ip-address-dns-name my-lamp-sample \
        --image CentOS --storage-sku Premium_LRS --admin-username deploy --vm-sku Standard_DS3_v2 --use-unmanaged-disk

    # Add a load balanced endpoint on port 80 routed to the backend servers on port 80
    az network lb rule create -g myResourceGroup -n http-rule --backend-pool-name myScaleSetLBBEPool \
        --backend-port 80 --frontend-ip-name LoadBalancerFrontEnd --frontend-port 80 --lb-name myScaleSetLB \
        --protocol Tcp

    # Create a virtual machine scale set custom script extension. This extension will provide configuration
    # to each of the virtual machines within the scale set on how to provision their software stack.
    # The configuration (./projected_config.json) contains commands to be executed upon provisioning
    # of instances. This is helpful for hooking into configuration management software or simply
    # provisioning your software stack directly.
    az vmss extension set -n CustomScript --publisher Microsoft.Azure.Extensions --version 2.0 \
       -g myResourceGroup --vmss-name myScaleSet --protected-settings ./protected_config.json --no-auto-upgrade

    # The instances that we have were created before the extension was added.
    # Update these instances to run the new configuration on each of them.
    az vmss update-instances --instance-ids "*" -n myScaleSet -g myResourceGroup

    # Scaling adds new instances. These instances will run the configuration when they're provisioned.
    az vmss scale --new-capacity 2 -n myScaleSet -g myResourceGroup


## <a name="connect"></a>连接

使用此代码查看如何连接到 VM 和规模集。

    #!/bin/bash

    FQDN=$(az network public-ip list -g myResourceGroup --query \
                    "[?starts_with(dnsSettings.fqdn, 'my-lamp-')].dnsSettings.fqdn | [0]" -o tsv)

    PORTS=$(az network lb show -g myResourceGroup -n myScaleSetLB \
        --query "inboundNatRules[].{backend: backendPort, frontendPort: frontendPort}" -o tsv)
    while read CMD; do
        read _ frontend <<< "${CMD}"
        echo "'ssh deploy@${FQDN} -p ${frontend}'"
    done <<< "${PORTS}"

    echo ""
    echo "You can now reach the scale set by opening your browser to: 'http://${FQDN}'."

## <a name="clean-up-deployment"></a>清理部署 

运行如下命令来删除资源组、规模集和 VM 以及所有相关资源。

    az group delete -n myResourceGroup

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令创建资源组、虚拟机、可用性集、负载均衡器和所有相关资源。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [az group create](https://docs.microsoft.com/zh-cn/cli/azure/group#create) | 创建用于存储所有资源的资源组。 |
| [az vmss create](https://docs.microsoft.com/zh-cn/cli/azure/vmss#create) | 创建虚拟机规模集 |
| [az network lb rule create](https://docs.microsoft.com/zh-cn/cli/azure/network/lb/rule#create) | 添加负载均衡终结点 |
| [az vmss extension set](https://docs.microsoft.com/zh-cn/cli/azure/vmss/extension#set) | 创建对 VM 部署运行自定义脚本的扩展 |
| [az vmss update-instances](https://docs.microsoft.com/zh-cn/cli/azure/vmss#update-instances) | 在将扩展应用到规模集之前部署的 VM 实例上运行此自定义脚本。 |
| [az vmss scale](https://docs.microsoft.com/zh-cn/cli/azure/vmss#scale) | 通过添加更多 VM 实例来扩大规模集。 部署实例时，在实例上运行此自定义脚本。 |
| [az network public-ip list](https://docs.microsoft.com/zh-cn/cli/azure/network/public-ip#list) | 获取示例创建的 VM 的 IP 地址。 |
| [az network lb show](https://docs.microsoft.com/zh-cn/cli/azure/network/lb#show) | 获取负载均衡器使用的前端和后端端口。 |

## <a name="next-steps"></a>后续步骤

有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](https://docs.microsoft.com/zh-cn/cli/azure/overview)。

可以在 [Azure Linux VM 文档](/documentation/articles/virtual-machines-linux-cli-samples/)中找到其他虚拟机 CLI 脚本示例。