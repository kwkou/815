<!-- need to be verified -->

<properties
    pageTitle="使用 Azure CLI 在多个 IP 配置上进行负载均衡 | Azure"
    description="了解如何使用 Azure CLI 将多个 IP 地址分配给虚拟机 | Resource Manager。"
    services="virtual-network"
    documentationcenter="na"
    author="anavinahar"
    manager="narayan"
    editor=""
    tags="azure-resource-manager" />
<tags 
    ms.assetid="" 
    ms.service="load-balancer"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="03/10/2017"
    wacn.date="04/17/2017"
    ms.author="annahar" />

# <a name="load-balancing-on-multiple-ip-configurations"></a>在多个 IP 配置上进行负载均衡
> [AZURE.SELECTOR]
- [门户](/documentation/articles/load-balancer-multiple-ip/)
- [CLI](/documentation/articles/load-balancer-multiple-ip-cli/)
- [PowerShell](/documentation/articles/load-balancer-multiple-ip-powershell/)

本文介绍如何将 Azure 负载均衡器用于辅助网络接口 (NIC) 的多个 IP 地址。 目前，对一个 NIC 的多个 IP 地址的支持是预览版的功能。 有关详细信息，请参阅本文的 [限制](#limitations) 部分。 以下场景说明了如何通过负载均衡器使用此功能。

在此方案中，有两个运行 Windows 的 VM，每个 VM 有一个主 NIC 和一个辅助 NIC。 每个辅助 NIC 具有两个 IP 配置。 每个 VM 托管网站 contoso.com 和 fabrikam.com。 每个网站都绑定到辅助 NIC 的一个 IP 配置。 我们使用 Azure 负载均衡器公开两个前端 IP 地址，每个地址分别对应于一个网站，从而将流量分发到网站的各个 IP 配置。 此场景中两个前端以及两个后端池 IP 地址都使用相同的端口号。

![负载均衡应用场景图像](./media/load-balancer-multiple-ip/lb-multi-ip.PNG)

## <a name="steps-to-load-balance-on-multiple-ip-configurations"></a>在多个 IP 配置上进行负载均衡的步骤

按照以下步骤来实现本文所概述的场景：

1. 按照所链接的文章中的步骤[安装和配置 Azure CLI](/documentation/articles/xplat-cli-install/)，然后登录到你的 Azure 帐户。
2. 如下所述[创建一个资源组](/documentation/articles/virtual-machines-linux-create-cli-complete/#create-resource-groups-and-choose-deployment-locations)并将其命名为 *contosofabrikam*。

        azure group create contosofabrikam chinaeast

3. 为两个 VM [创建可用性集](/documentation/articles/virtual-machines-linux-create-cli-complete/#create-an-availability-set)。 对于此场景，请使用以下命令：

        azure availset create --resource-group contosofabrikam --location chinaeast --name myAvailabilitySet

4. [创建一个虚拟网络](/documentation/articles/virtual-machines-linux-create-cli-complete/#create-a-virtual-network-and-subnet)并将其命名为 *myVNet*，并创建一个名为 *mySubnet* 的子网：

        azure network vnet create --resource-group contosofabrikam --name myVnet --address-prefixes 10.0.0.0/16  --location chinaeast

        azure network vnet subnet create --resource-group contosofabrikam --vnet-name myVnet --name mySubnet --address-prefix 10.0.0.0/24

5. [创建负载均衡器](/documentation/articles/virtual-machines-linux-create-cli-complete/#create-a-load-balancer-and-ip-pools)并将其命名为 *mylb*：

        azure network lb create --resource-group contosofabrikam --location chinaeast --name mylb

6. 为负载均衡器的前端 IP 配置创建两个动态公共 IP 地址：

        azure network public-ip create --resource-group contosofabrikam --location chinaeast --name PublicIp1 --domain-name-label contoso --allocation-method Dynamic

        azure network public-ip create --resource-group contosofabrikam --location chinaeast --name PublicIp2 --domain-name-label fabrikam --allocation-method Dynamic

7. 创建两个分别名为 *contosofe* 和 *fabrikamfe* 的前端 IP 配置：

        azure network lb frontend-ip create --resource-group contosofabrikam --lb-name mylb --public-ip-name PublicIp1 --name contosofe
        azure network lb frontend-ip create --resource-group contosofabrikam --lb-name mylb --public-ip-name PublicIp2 --name fabrkamfe

8. 创建后端地址池 - *contosopool* 和 *fabrikampool*、[探测器](/documentation/articles/virtual-machines-linux-create-cli-complete/#create-a-load-balancer-health-probe) - *HTTP* 以及负载均衡规则 - *HTTPc* 和 *HTTPf*：

        azure network lb address-pool create --resource-group contosofabrikam --lb-name mylb --name contosopool
        azure network lb address-pool create --resource-group contosofabrikam --lb-name mylb --name fabrikampool

        azure network lb probe create --resource-group contosofabrikam --lb-name mylb --name HTTP --protocol "http" --interval 15 --count 2 --path index.html

        azure network lb rule create --resource-group contosofabrikam --lb-name mylb --name HTTPc --protocol tcp --probe-name http--frontend-port 5000 --backend-port 5000 --frontend-ip-name contosofe --backend-address-pool-name contosopool
        azure network lb rule create --resource-group contosofabrikam --lb-name mylb --name HTTPf --protocol tcp --probe-name http --frontend-port 5000 --backend-port 5000 --frontend-ip-name fabrkamfe --backend-address-pool-name fabrikampool

9. 运行以下命令，然后检查输入以[验证是否正确创建了负载均衡器](/documentation/articles/virtual-machines-linux-create-cli-complete/#verify-the-load-balancer)：

        azure network lb show --resource-group contosofabrikam --name mylb

10. 为第一个虚拟机 VM1 [创建公共 IP](/documentation/articles/virtual-machines-linux-create-cli-complete/#create-a-public-ip-address) *myPublicIp* 和[存储帐户](/documentation/articles/virtual-machines-linux-create-cli-complete/#create-a-storage-account) *mystorageaccont1*，如下所示：

        azure network public-ip create --resource-group contosofabrikam --location chinaeast --name myPublicIP --domain-name-label mypublicdns345 --allocation-method Dynamic

        azure storage account create --location chinaeast --resource-group contosofabrikam --kind Storage --sku-name GRS mystorageaccount1

11. 为 VM1 [创建网络接口](/documentation/articles/virtual-machines-linux-create-cli-complete/#create-an-nic-to-use-with-the-linux-vm)，添加另一个 IP 配置 *VM1-ipconfig2*，然后[创建 VM](/documentation/articles/virtual-machines-linux-create-cli-complete/#create-the-linux-vms)，如下所示：

        azure network nic create --resource-group contosofabrikam --location chinaeast --subnet-vnet-name myVnet --subnet-name mySubnet --name VM1Nic1 --ip-config-name NIC1-ipconfig1
        azure network nic create --resource-group contosofabrikam --location chinaeast --subnet-vnet-name myVnet --subnet-name mySubnet --name VM1Nic2 --ip-config-name VM1-ipconfig1 --public-ip-name myPublicIP --lb-address-pool-ids "/subscriptions/<your subscription ID>/resourceGroups/contosofabrikam/providers/Microsoft.Network/loadBalancers/mylb/backendAddressPools/contosopool"
        azure network nic ip-config create --resource-group contosofabrikam --nic-name VM1Nic2 --name VM1-ipconfig2 --lb-address-pool-ids "/subscriptions/<your subscription ID>/resourceGroups/contosofabrikam/providers/Microsoft.Network/loadBalancers/mylb/backendAddressPools/fabrikampool"
        azure vm create --resource-group contosofabrikam --name VM1 --location chinaeast --os-type linux --nic-names VM1Nic1,VM1Nic2  --vnet-name VNet1 --vnet-subnet-name Subnet1 --availset-name myAvailabilitySet --vm-size Standard_DS3_v2 --storage-account-name mystorageaccount1 --image-urn canonical:UbuntuServer:16.04.0-LTS:latest --admin-username <your username>  --admin-password <your password>

12. 为第二个 VM 重复步骤 10-11：

        azure network public-ip create --resource-group contosofabrikam --location chinaeast --name myPublicIP2 --domain-name-label mypublicdns785 --allocation-method Dynamic
        azure storage account create --location chinaeast --resource-group contosofabrikam --kind Storage --sku-name GRS mystorageaccount2
        azure network nic create --resource-group contosofabrikam --location chinaeast --subnet-vnet-name myVnet --subnet-name mySubnet --name VM2Nic1
        azure network nic create --resource-group contosofabrikam --location chinaeast --subnet-vnet-name myVnet --subnet-name mySubnet --name VM2Nic2 --ip-config-name VM2-ipconfig1 --public-ip-name myPublicIP2 --lb-address-pool-ids "/subscriptions/<your subscription ID>/resourceGroups/contosofabrikam/providers/Microsoft.Network/loadBalancers/mylb/backendAddressPools/contosopool"
        azure network nic ip-config create --resource-group contosofabrikam --nic-name VM2Nic2 --name VM2-ipconfig2 --lb-address-pool-ids "/subscriptions/<your subscription ID>/resourceGroups/contosofabrikam/providers/Microsoft.Network/loadBalancers/mylb/backendAddressPools/fabrikampool"
        azure vm create --resource-group contosofabrikam --name VM2 --location chinaeast --os-type linux --nic-names VM2Nic1,VM2Nic2 --vnet-name VNet1 --vnet-subnet-name Subnet1 --availset-name myAvailabilitySet --vm-size Standard_DS3_v2 --storage-account-name mystorageaccount2 --image-urn canonical:UbuntuServer:16.04.0-LTS:latest --admin-username <your username>  --admin-password <your password>

13. 最后，必须将 DNS 资源记录配置为指向各自的负载均衡器的前端 IP 地址。 可以在 Azure DNS 中托管域。 有关将 Azure DNS 与负载均衡器配合使用的详细信息，请参阅[将 Azure DNS 与其他 Azure 服务配合使用](/documentation/articles/dns-for-azure-services/)。

## <a name="next-steps"></a>后续步骤
- 若要深入了解如何在 Azure 中结合使用负载均衡服务，请参阅[在 Azure 中使用负载均衡服务](/documentation/articles/traffic-manager-load-balancing-azure/)。
- 若要了解如何在 Azure 中使用不同类型的日志对负载均衡器进行管理和故障排除，请参阅 [Azure 负载均衡器的 Log Analytics](/documentation/articles/load-balancer-monitor-log/)。

<!-- Update_Description:udpate meta properties; wording update; add new programmer section code  -->