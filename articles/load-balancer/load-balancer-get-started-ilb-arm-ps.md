<properties 
   pageTitle="在 Resource Manager 中使用 PowerShell 创建内部负载平衡器 | Azure"
   description="了解如何在 Resource Manager 中使用 PowerShell 创建内部负载平衡器"
   services="load-balancer"
   documentationCenter="na"
   authors="joaoma"
   manager="carmonm"
   editor=""
   tags="azure-resource-manager"
/>
<tags  
   ms.service="load-balancer"
   ms.date="02/09/2016"
   wacn.date="08/29/2016" />

# 开始使用 PowerShell 创建内部负载平衡器

[AZURE.INCLUDE [load-balancer-get-started-ilb-arm-selectors-include.md](../../includes/load-balancer-get-started-ilb-arm-selectors-include.md)]
<BR>
[AZURE.INCLUDE [load-balancer-get-started-ilb-intro-include.md](../../includes/load-balancer-get-started-ilb-intro-include.md)]

[AZURE.INCLUDE [azure-arm-classic-important-include](../../includes/learn-about-deployment-models-rm-include.md)] [classic deployment model](/documentation/articles/load-balancer-get-started-ilb-classic-ps/)。

[AZURE.INCLUDE [load-balancer-get-started-ilb-scenario-include.md](../../includes/load-balancer-get-started-ilb-scenario-include.md)]

[AZURE.INCLUDE [azure-ps-prerequisites-include.md](../../includes/azure-ps-prerequisites-include.md)]


下面的步骤将说明如何通过 PowerShell 使用 Azure Resource Manager 创建内部负载平衡器。使用 Azure Resource Manager，单独配置用于创建内部负载平衡器的项目，然后将这些项目组合在一起来创建资源。

本文涵盖要创建内部负载平衡器所必须完成的单个任务的序列，并详细说明要怎样做才能实现创建负载平衡器的目标。


## 创建内部负载平衡器需要什么？


在创建内部负载平衡器之前，需要配置以下项：

- 前端 IP 配置 - 配置传入网络流量的专用 IP 地址

- 后端地址池 - 配置网络接口以接收来自前端 IP 池的负载平衡流量

- 负载平衡规则 - 负载平衡器的源和本地端口配置。

- 探测器 - 为虚拟机实例配置运行状况状态探测器。

- 入站 NAT 规则 - 配置端口规则以直接访问某个虚拟机实例。

你可以在以下网页中获取有关 Azure Resource Manager 的负载平衡器组件的详细信息：[Azure Resource Manager 对负载平衡器的支持](/documentation/articles/load-balancer-arm/)。

以下步骤将演示如何配置 2 个虚拟机之间的负载平衡器。


## 使用 PowerShell 进行分步


### 将 PowerShell 设置为使用 Resource Manager

请确保你有 PowerShell 的 Azure 模块 的最新生产版本，并且正确地让 PowerShell 安装程序访问你的 Azure 订阅。

### 步骤 1

		Login-AzureRmAccount -EnvironmentName AzureChinaCloud



### 步骤 2

检查帐户的订阅

		Get-AzureRmSubscription 

系统将提示你使用凭据进行身份验证。<BR>

### 步骤 3 

选择要使用的 Azure 订阅。<BR>


		Select-AzureRmSubscription -Subscriptionid "GUID of subscription"

### 为负载平衡器创建资源组

### 步骤 4

创建新的资源组（如果要使用现有的资源组，请跳过此步骤）

    	New-AzureRmResourceGroup -Name NRP-RG -location "China East"

Azure 资源管理器要求所有资源组指定一个位置。此位置将用作该资源组中的资源的默认位置。请确保用于创建负载平衡器的所有命令都使用相同的资源组。

在上面的示例中，我们在位置“中国东部”创建了名为“NRP-RG”的资源组。

## 为前端 IP 池创建虚拟网络和专用 IP 地址


### 步骤 1

为虚拟网络创建子网，并将其分配给变量 $backendSubnet

	$backendSubnet = New-AzureRmVirtualNetworkSubnetConfig -Name LB-Subnet-BE -AddressPrefix 10.0.2.0/24

创建虚拟网络：

	$vnet= New-AzureRmVirtualNetwork -Name NRPVNet -ResourceGroupName NRP-RG -Location "China East" -AddressPrefix 10.0.0.0/16 -Subnet $backendSubnet

创建虚拟网络，将子网 lb-subnet-be 添加到虚拟网络 NRPVNet，并将其分配给变量 $vnet



## 创建前端 IP 池和后端地址池

针对传入负载平衡器网络流量设置前端 IP 池和后端地址池以接收经过负载平衡的流量。

### 步骤 1 

使用专用 IP 地址 10.0.2.5 为子网 10.0.2.0/24 创建前端 IP 池，该池将是传入网络流量终结点。

	$frontendIP = New-AzureRmLoadBalancerFrontendIpConfig -Name LB-Frontend -PrivateIpAddress 10.0.2.5 -SubnetId $vnet.subnets[0].Id

### 步骤 2 

设置用于从前端 IP 池接收传入流量的后端地址池：

	$beaddresspool= New-AzureRmLoadBalancerBackendAddressPoolConfig -Name "LB-backend"


## 创建 LB 规则、NAT 规则、探测器和负载平衡器

创建前端 IP 池和后端地址池后，你将需要创建属于负载平衡器资源的规则：

### 步骤 1

	$inboundNATRule1= New-AzureRmLoadBalancerInboundNatRuleConfig -Name "RDP1" -FrontendIpConfiguration $frontendIP -Protocol TCP -FrontendPort 3441 -BackendPort 3389

	$inboundNATRule2= New-AzureRmLoadBalancerInboundNatRuleConfig -Name "RDP2" -FrontendIpConfiguration $frontendIP -Protocol TCP -FrontendPort 3442 -BackendPort 3389

	$healthProbe = New-AzureRmLoadBalancerProbeConfig -Name "HealthProbe" -RequestPath "HealthProbe.aspx" -Protocol http -Port 80 -IntervalInSeconds 15 -ProbeCount 2

 	$lbrule = New-AzureRmLoadBalancerRuleConfig -Name "HTTP" -FrontendIpConfiguration $frontendIP -BackendAddressPool $beAddressPool -Probe $healthProbe -Protocol Tcp -FrontendPort 80 -BackendPort 80


上面的示例将创建以下项：

- NAT 规则，它使端口 3441 的所有传入流量转到端口 3389。
- 第二个 NAT 规则，它使端口 3442 的所有传入流量转到端口 3389。
- 负载平衡器规则，它将公共端口 80 上的所有传入流量负载平衡到后端地址池中的本地端口 80。
- 探测规则，它将检查路径“HealthProbe.aspx”的运行状况状态



### 步骤 2

将所有对象（NAT 规则、负载平衡器规则、探测配置）添加在一起创建负载平衡器：

	$NRPLB = New-AzureRmLoadBalancer -ResourceGroupName "NRP-RG" -Name "NRP-LB" -Location "China East" -FrontendIpConfiguration $frontendIP -InboundNatRule $inboundNATRule1,$inboundNatRule2 -LoadBalancingRule $lbrule -BackendAddressPool $beAddressPool -Probe $healthProbe 


## 创建网络接口

创建内部负载平衡器后，需要定义哪些网络接口将接收传入的负载平衡网络流量、NAT 规则和探测器。在这种情况下，网络接口将单独配置，并可以在以后分配给虚拟机。


### 步骤 1 


获取用于创建网络接口的资源虚拟网络和子网：

	$vnet = Get-AzureRmVirtualNetwork -Name NRPVNet -ResourceGroupName NRP-RG

	$backendSubnet = Get-AzureRmVirtualNetworkSubnetConfig -Name LB-Subnet-BE -VirtualNetwork $vnet 


在此步骤中，我们将创建属于负载平衡器后端池的网络接口，并将为此网络接口关联 RDP 的第一个 NAT 规则：
	
	$backendnic1= New-AzureRmNetworkInterface -ResourceGroupName "NRP-RG" -Name lb-nic1-be -Location "China East" -PrivateIpAddress 10.0.2.6 -Subnet $backendSubnet -LoadBalancerBackendAddressPool $nrplb.BackendAddressPools[0] -LoadBalancerInboundNatRule $nrplb.InboundNatRules[0]

### 步骤 2

创建名为 LB-Nic2-BE 的第二个网络接口：

在此步骤中，我们将创建第二个网络接口，将其分配给同一个负载平衡器后端池，并将关联为 RDP 创建的第二个 NAT 规则：

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

你可以按照文档 [Create and preconfigure a Windows Virtual Machine with Resource Manager and Azure PowerShell](/documentation/articles/virtual-machines-windows-create-powershell#Example)（使用 Resource Manager 和 Azure PowerShell 创建并预配置 Windows 虚拟机）选项 4 或 5，找到相关分步说明，以创建虚拟机并将其分配给 NIC。

或者，如果你已创建虚拟机，则可以使用以下步骤添加网络接口：

#### 步骤 1 

将负载平衡器资源加载到变量中（如果你还没有这样做）。所用的变量名为 $lb，并使用前面创建的负载平衡器资源的相同名称。

	$lb= Get-AzureRmLoadBalancer –name NRP-LB -resourcegroupname NRP-RG

#### 步骤 2 

将后端配置加载到变量。

	$backend= Get-AzureRmLoadBalancerBackendAddressPoolConfig -name backendpool1 -LoadBalancer $lb

#### 步骤 3 

将已创建的网络接口加载到变量中。所用的变量名称为 $nic。所用的网络接口名称与前面的示例相同。

	$nic=Get-AzureRmNetworkInterface –name lb-nic1-be -resourcegroupname NRP-RG

#### 步骤 4

更改网络接口上的后端配置。

	$nic.IpConfigurations[0].LoadBalancerBackendAddressPools=$backend

#### 步骤 5 

保存网络接口对象。

	Set-AzureRmNetworkInterface -NetworkInterface $nic

将网络接口添加到负载平衡器后端池后，它会根据该负载平衡器资源的负载平衡规则开始接收网络流量。

## 更新现有的负载平衡器


### 步骤 1

使用前面示例中的负载平衡器，通过 Get-AzureRmLoadBalancer 将负载平衡器对象分配给变量 $slb

	$slb=get-azureRmLoadBalancer -Name NRPLB -ResourceGroupName NRP-RG

### 步骤 2

在以下示例中，你将在前端使用端口 81 添加新的入站 NAT 规则，并将后端池的端口 8181 添加到现有的负载平衡器

	$slb | Add-AzureRmLoadBalancerInboundNatRuleConfig -Name NewRule -FrontendIpConfiguration $slb.FrontendIpConfigurations[0] -FrontendPort 81  -BackendPort 8181 -Protocol Tcp


### 步骤 3

使用 Set-AzureLoadBalancer 保存新配置

	$slb | Set-AzureRmLoadBalancer

## 删除负载平衡器

使用命令 Remove-AzureRmLoadBalancer 删除以前在名为“NRP-RG”的资源组中创建的名为“NRP-LB”的负载平衡器

	Remove-AzureRmLoadBalancer -Name NRPLB -ResourceGroupName NRP-RG

>[AZURE.NOTE] 你可以使用可选开关 -Force 来避免显示删除提示。



## 后续步骤

[配置负载平衡器分发模式](/documentation/articles/load-balancer-distribution-mode)

[为负载平衡器配置空闲 TCP 超时设置](/documentation/articles/load-balancer-tcp-idle-timeout)
 

<!---HONumber=Mooncake_0822_2016-->
