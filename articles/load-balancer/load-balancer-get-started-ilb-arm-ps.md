<properties
   pageTitle="在 Resource Manager 中使用 PowerShell 创建内部负载均衡器 | Azure"
   description="了解如何在 Resource Manager 中使用 PowerShell 创建内部负载均衡器"
   services="load-balancer"
   documentationCenter="na"
   authors="sdwheeler"
   manager="carmonm"
   editor=""
   tags="azure-resource-manager"
/>  

<tags
   ms.service="load-balancer"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="02/09/2016"
   wacn.date="12/05/2016"
   ms.author="sewhee" />  


# 使用 PowerShell 创建内部负载均衡器

> [AZURE.SELECTOR]
[Azure Portal](/documentation/articles/load-balancer-get-started-ilb-arm-portal/)
[PowerShell](/documentation/articles/load-balancer-get-started-ilb-arm-ps/)
[Azure CLI](/documentation/articles/load-balancer-get-started-ilb-arm-cli/)
[Template](/documentation/articles/load-balancer-get-started-ilb-arm-template/)

>[AZURE.INCLUDE [load-balancer-get-started-ilb-intro-include.md](../../includes/load-balancer-get-started-ilb-intro-include.md)]

>[AZURE.NOTE] Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍如何使用 Resource Manager 部署模型。Microsoft 建议对大多数新部署使用该模型，而不要使用[经典部署模型](/documentation/articles/load-balancer-get-started-ilb-classic-ps/)。

[AZURE.INCLUDE [load-balancer-get-started-ilb-scenario-include.md](../../includes/load-balancer-get-started-ilb-scenario-include.md)]

[AZURE.INCLUDE [azure-ps-prerequisites-include.md](../../includes/azure-ps-prerequisites-include.md)]

以下步骤介绍如何使用 Azure Resource Manager 和 PowerShell 创建内部负载均衡器。使用 Azure Resource Manager 时，会单独配置用于创建内部负载均衡器的项目，然后组合这些项目用于创建负载均衡器。

需要创建和配置以下对象以部署负载均衡器：

* 前端 IP 配置 - 配置传入网络流量的专用 IP 地址
* 后端地址池 - 配置网络接口以接收来自前端 IP 池的负载平衡流量
* 负载平衡规则 - 负载均衡器的源和本地端口配置。
* 探测器 - 为虚拟机实例配置运行状况状态探测器。
* 入站 NAT 规则 - 配置端口规则以直接访问某个虚拟机实例。

你可以在以下网页中获取有关 Azure Resource Manager 的负载均衡器组件的详细信息：[Azure Resource Manager 对负载均衡器的支持](/documentation/articles/load-balancer-arm/)。

以下步骤介绍如何配置两个虚拟机之间的负载均衡器。

## 将 PowerShell 设置为使用 Resource Manager

请确保你有 PowerShell 的 Azure 模块 的最新生产版本，并且正确地让 PowerShell 安装程序访问你的 Azure 订阅。

### 步骤 1

	Login-AzureRmAccount -EnvironmentName AzureChinaCloud


### 步骤 2

检查帐户的订阅

	Get-AzureRmSubscription

系统将提示你使用凭据进行身份验证。<BR>

### 步骤 3

选择要使用的 Azure 订阅。


	Select-AzureRmSubscription -Subscriptionid "GUID of subscription"

### 为负载均衡器创建资源组

创建新的资源组（如果要使用现有的资源组，请跳过此步骤）

	New-AzureRmResourceGroup -Name NRP-RG -location "China East"

Azure 资源管理器要求所有资源组指定一个位置。此位置将用作该资源组中的资源的默认位置。请确保用于创建负载均衡器的所有命令都使用相同的资源组。

在上面的示例中，我们在位置“中国东部”创建了名为“NRP-RG”的资源组。

## 为前端 IP 池创建虚拟网络和专用 IP 地址

为虚拟网络创建子网，并将其分配给变量 $backendSubnet

	$backendSubnet = New-AzureRmVirtualNetworkSubnetConfig -Name LB-Subnet-BE -AddressPrefix 10.0.2.0/24

创建虚拟网络：

	$vnet= New-AzureRmVirtualNetwork -Name NRPVNet -ResourceGroupName NRP-RG -Location "China East" -AddressPrefix 10.0.0.0/16 -Subnet $backendSubnet

创建虚拟网络，将子网 lb-subnet-be 添加到虚拟网络 NRPVNet，并将其分配给变量 $vnet

## 创建前端 IP 池和后端地址池

为传入负载均衡器网络流量设置前端 IP 池，并创建后端地址池用于接收负载平衡的流量。

### 步骤 1

使用专用 IP 地址 10.0.2.5 为子网 10.0.2.0/24 创建前端 IP 池，该池将是传入网络流量终结点。

	$frontendIP = New-AzureRmLoadBalancerFrontendIpConfig -Name LB-Frontend -PrivateIpAddress 10.0.2.5 -SubnetId $vnet.subnets[0].Id

### 步骤 2

设置用于从前端 IP 池接收传入流量的后端地址池：

	$beaddresspool= New-AzureRmLoadBalancerBackendAddressPoolConfig -Name "LB-backend"


## 创建 LB 规则、NAT 规则、探测器和负载均衡器

创建前端 IP 池和后端地址池后，你将需要创建属于负载均衡器资源的规则：

### 步骤 1

	$inboundNATRule1= New-AzureRmLoadBalancerInboundNatRuleConfig -Name "RDP1" -FrontendIpConfiguration $frontendIP -Protocol TCP -FrontendPort 3441 -BackendPort 3389

	$inboundNATRule2= New-AzureRmLoadBalancerInboundNatRuleConfig -Name "RDP2" -FrontendIpConfiguration $frontendIP -Protocol TCP -FrontendPort 3442 -BackendPort 3389

	$healthProbe = New-AzureRmLoadBalancerProbeConfig -Name "HealthProbe" -RequestPath "HealthProbe.aspx" -Protocol http -Port 80 -IntervalInSeconds 15 -ProbeCount 2

	$lbrule = New-AzureRmLoadBalancerRuleConfig -Name "HTTP" -FrontendIpConfiguration $frontendIP -BackendAddressPool $beAddressPool -Probe $healthProbe -Protocol Tcp -FrontendPort 80 -BackendPort 80


上面的示例将创建以下项：

* NAT 规则，它使端口 3441 的所有传入流量转到端口 3389。
* 第二个 NAT 规则，它使端口 3442 的所有传入流量转到端口 3389。
* 负载均衡器规则，它将公共端口 80 上的所有传入流量负载平衡到后端地址池中的本地端口 80。
* 探测规则，它将检查路径“HealthProbe.aspx”的运行状况状态

### 步骤 2

将所有对象（NAT 规则、负载均衡器规则、探测配置）添加在一起创建负载均衡器：

	$NRPLB = New-AzureRmLoadBalancer -ResourceGroupName "NRP-RG" -Name "NRP-LB" -Location "China East" -FrontendIpConfiguration $frontendIP -InboundNatRule $inboundNATRule1,$inboundNatRule2 -LoadBalancingRule $lbrule -BackendAddressPool $beAddressPool -Probe $healthProbe 


## 创建网络接口

创建内部负载均衡器后，需要定义哪些网络接口将接收传入的负载平衡网络流量、NAT 规则和探测器。在这种情况下，网络接口将单独配置，并可以在以后分配给虚拟机。

### 步骤 1

获取用于创建网络接口的资源虚拟网络和子网：

	$vnet = Get-AzureRmVirtualNetwork -Name NRPVNet -ResourceGroupName NRP-RG

	$backendSubnet = Get-AzureRmVirtualNetworkSubnetConfig -Name LB-Subnet-BE -VirtualNetwork $vnet


此步骤创建属于负载均衡器后端池的网络接口，并为此网络接口关联 RDP 的第一个 NAT 规则：

	$backendnic1= New-AzureRmNetworkInterface -ResourceGroupName "NRP-RG" -Name lb-nic1-be -Location "China East" -PrivateIpAddress 10.0.2.6 -Subnet $backendSubnet -LoadBalancerBackendAddressPool $nrplb.BackendAddressPools[0] -LoadBalancerInboundNatRule $nrplb.InboundNatRules[0]

### 步骤 2

创建名为 LB-Nic2-BE 的第二个网络接口：

此步骤创建第二个网络接口，将其分配给同一负载均衡器后端池，并关联为 RDP 创建的第二个 NAT 规则：

 	$backendnic2= New-AzureRmNetworkInterface -ResourceGroupName "NRP-RG" -Name lb-nic2-be -Location "China East" -PrivateIpAddress 10.0.2.7 -Subnet $backendSubnet -LoadBalancerBackendAddressPool $nrplb.BackendAddressPools[0] -LoadBalancerInboundNatRule $nrplb.InboundNatRules[1]


最终结果将显示以下信息：

    $backendnic1

预期输出：

    Name                 : lb-nic1-be
    ResourceGroupName    : NRP-RG
    Location             : chinaeast
    Id                   : /subscriptions/f50504a2-1865-4541-823a-b32842e3e0ee/resourceGroups/NRP-RG/providers/Microsoft.Network/networkInterfaces/lb-nic1-be
    Etag                 : W/"d448256a-e1df-413a-9103-a137e07276d1"
    ProvisioningState    : Succeeded
    Tags                 :
    VirtualMachine       : null
    IpConfigurations     : [
                         {
                           "PrivateIpAddress": "10.0.2.6",
                           "PrivateIpAllocationMethod": "Static",
                           "Subnet": {
                             "Id": "/subscriptions/f50504a2-1865-4541-823a-b32842e3e0ee/resourceGroups/NRP-RG/providers/Microsoft.Network/virtualNetworks/NRPVNet/subnets/LB-Subnet-BE"
                           },
                           "PublicIpAddress": {
                             "Id": null
                           },
                           "LoadBalancerBackendAddressPools": [
                             {
                               "Id": "/subscriptions/f50504a2-1865-4541-823a-b32842e3e0ee/resourceGroups/NRP-RG/providers/Microsoft.Network/loadBalancers/NRPlb/backendAddressPools/LB-backend"
                             }
                           ],
                           "LoadBalancerInboundNatRules": [
                             {
                               "Id": "/subscriptions/f50504a2-1865-4541-823a-b32842e3e0ee/resourceGroups/NRP-RG/providers/Microsoft.Network/loadBalancers/NRPlb/inboundNatRules/RDP1"
                             }
                           ],
                           "ProvisioningState": "Succeeded",
                           "Name": "ipconfig1",
                           "Etag": "W/"d448256a-e1df-413a-9103-a137e07276d1"",
                           "Id": "/subscriptions/f50504a2-1865-4541-823a-b32842e3e0ee/resourceGroups/NRP-RG/providers/Microsoft.Network/networkInterfaces/lb-nic1-be/ipConfigurations/ipconfig1"
                         }
                       ]
    DnsSettings          : {
                         "DnsServers": [],
                         "AppliedDnsServers": []
                       }
    AppliedDnsSettings   :
    NetworkSecurityGroup : null
    Primary              : False



### 步骤 3

使用命令 Add-AzureRmVMNetworkInterface 将 NIC 分配给虚拟机。

可以通过以下文档找到相关分步说明，以创建虚拟机并将其分配给 NIC：[使用 PowerShell 创建 Azure VM(/documentation/articles/virtual-machines-windows-create-powershell)。

## 添加网络接口

如果已创建虚拟机，则可以使用以下步骤添加网络接口：

### 步骤 1

将负载均衡器资源加载到变量中（如果你还没有这样做）。所用的变量名为 $lb，并使用前面创建的负载均衡器资源的相同名称。

	$lb= Get-AzureRmLoadBalancer –name NRP-LB -resourcegroupname NRP-RG

### 步骤 2

将后端配置加载到变量。

	$backend= Get-AzureRmLoadBalancerBackendAddressPoolConfig -name backendpool1 -LoadBalancer $lb

### 步骤 3

将已创建的网络接口加载到变量中。所用的变量名称为 $nic。所用的网络接口名称与前面的示例相同。

	$nic=Get-AzureRmNetworkInterface –name lb-nic1-be -resourcegroupname NRP-RG

### 步骤 4

更改网络接口上的后端配置。

	$nic.IpConfigurations[0].LoadBalancerBackendAddressPools=$backend

### 步骤 5

保存网络接口对象。

	Set-AzureRmNetworkInterface -NetworkInterface $nic

将网络接口添加到负载均衡器后端池后，它会根据该负载均衡器资源的负载平衡规则开始接收网络流量。

## 更新现有的负载均衡器

### 步骤 1
使用前面示例中的负载均衡器，通过 Get-AzureRmLoadBalancer 将负载均衡器对象分配给变量 $slb

	$slb=Get-AzureRmLoadBalancer -Name NRPLB -ResourceGroupName NRP-RG

### 步骤 2

在以下示例中，你将在前端使用端口 81 添加新的入站 NAT 规则，并将后端池的端口 8181 添加到现有的负载均衡器

	$slb | Add-AzureRmLoadBalancerInboundNatRuleConfig -Name NewRule -FrontendIpConfiguration $slb.FrontendIpConfigurations[0] -FrontendPort 81  -BackendPort 8181 -Protocol Tcp


### 步骤 3

使用 Set-AzureLoadBalancer 保存新配置

	$slb | Set-AzureRmLoadBalancer

## 删除负载均衡器

使用命令 Remove-AzureRmLoadBalancer 删除以前在名为“NRP-RG”的资源组中创建的名为“NRP-LB”的负载均衡器

	Remove-AzureRmLoadBalancer -Name NRPLB -ResourceGroupName NRP-RG

>[AZURE.NOTE] 你可以使用可选开关 -Force 来避免显示删除提示。



## 后续步骤

[配置负载均衡器分发模式](/documentation/articles/load-balancer-distribution-mode/)

[为负载均衡器配置空闲 TCP 超时设置](/documentation/articles/load-balancer-tcp-idle-timeout/)

<!---HONumber=Mooncake_1128_2016-->