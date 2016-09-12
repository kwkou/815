<properties 
   pageTitle="使用 PowerShell 在 Resource Manager 中创建面向 Internet 的负载均衡器 | Azure"
   description="了解如何使用 PowerShell 在 Resource Manager 中创建面向 Internet 的负载均衡器"
   services="load-balancer"
   documentationCenter="na"
   authors="joaoma"
   manager="carmonm"
   editor=""
   tags="azure-resource-manager"
/>
<tags  
   ms.service="load-balancer"
   ms.date="04/05/2016"
   wacn.date="08/29/2016" />

# 开始使用 PowerShell 在 Resource Manager 中创建面向 Internet 的负载均衡器

[AZURE.INCLUDE [load-balancer-get-started-internet-arm-selectors-include.md](../../includes/load-balancer-get-started-internet-arm-selectors-include.md)]

[AZURE.INCLUDE [load-balancer-get-started-internet-intro-include.md](../../includes/load-balancer-get-started-internet-intro-include.md)]

>[AZURE.IMPORTANT]在使用 Azure 资源之前，请务必了解 Azure 当前使用两种部署模型：资源管理器部署模型和经典部署模型。在使用任何 Azure 资源之前，请确保你了解[部署模型和工具](/documentation/articles/azure-classic-rm/)。可以通过单击本文顶部的选项卡来查看不同工具的文档。本文介绍资源管理器部署模型。你还可以[了解如何使用经典部署模型创建面向 Internet 的负载均衡器](/documentation/articles/load-balancer-get-started-internet-classic-cli/)。

[AZURE.INCLUDE [load-balancer-get-started-internet-scenario-include.md](../../includes/load-balancer-get-started-internet-scenario-include.md)]

下面的步骤将说明如何通过 PowerShell 使用 Azure Resource Manager 创建面向 Internet 的负载均衡器。使用 Azure Resource Manager，单独配置用于创建面向 Internet 的负载均衡器的项目，然后将这些项目组合在一起来创建资源。

这将涵盖要创建负载均衡器所必须完成的单个任务的序列，并详细说明要怎样做才能实现目标。

## 创建面向 Internet 的负载均衡器需要什么？

需要创建和配置以下对象以部署负载均衡器：

- 前端 IP 配置 - 包含传入网络流量的公共 IP 地址。

- 后端地址池 - 包含要从负载均衡器接收网络流量的虚拟机的网络接口 (NIC)。

- 负载均衡规则 - 包含将负载均衡器上的公共端口映射到后端地址池中的端口的规则。

- 入站 NAT 规则 - 包含将负载均衡器上的公共端口映射到后端地址池中特定虚拟机的端口的规则。

- 探测器 - 包含用于检查后端地址池中虚拟机实例的可用性的运行状况探测器。

你可以在以下网页中获取有关 Azure Resource Manager 的负载均衡器组件的详细信息：[Azure Resource Manager 对负载均衡器的支持](/documentation/articles/load-balancer-arm/)。


## 将 PowerShell 设置为使用 Resource Manager

请确保你有 PowerShell 的 Azure Resource Manager (ARM) 模块的最新生产版本。

### 步骤 1

		Login-AzureRmAccount -EnvironmentName AzureChinaCloud

系统将提示你使用凭据进行身份验证。<BR>

### 步骤 2

检查帐户的订阅

		Get-AzureRmSubscription 

### 步骤 3 

选择要使用的 Azure 订阅。<BR>

		Select-AzureRmSubscription -SubscriptionId 'GUID of subscription'

### 步骤 4

创建新的资源组（如果要使用现有的资源组，请跳过此步骤）


    	New-AzureRmResourceGroup -Name NRP-RG -location "China East"


## 为前端 IP 池创建虚拟网络和公共 IP 地址

### 步骤 1

创建子网和虚拟网络。

    $backendSubnet = New-AzureRmVirtualNetworkSubnetConfig -Name LB-Subnet-BE -AddressPrefix 10.0.2.0/24
    New-AzureRmvirtualNetwork -Name NRPVNet -ResourceGroupName NRP-RG -Location 'China East' -AddressPrefix 10.0.0.0/16 -Subnet $backendSubnet

### 步骤 2

使用 DNS 名称 *loadbalancernrp.chinaeast.chinacloudapp.cn* 创建要由前端 IP 池使用的名为 *PublicIP* 的 Azure 公共 IP 地址 (PIP) 资源。下面的命令使用静态分配类型。

	$publicIP = New-AzureRmPublicIpAddress -Name PublicIp -ResourceGroupName NRP-RG -Location 'China East' –AllocationMethod Static -DomainNameLabel loadbalancernrp 

>[AZURE.IMPORTANT] 负载均衡器将使用公共 IP 的域标签作为其 FQDN 的前缀。这与使用云服务作为负载均衡器 FQDN 的经典部署模型不同。
在此示例中，FQDN 将为 *loadbalancernrp.chinaeast.chinacloudapp.cn*。

## 创建前端 IP 池和后端地址池

### 步骤 1 

创建使用 *PublicIp* PIP、名为 *LB-Frontend* 的前端 IP 池。

	$frontendIP = New-AzureRmLoadBalancerFrontendIpConfig -Name LB-Frontend -PublicIpAddress $publicIP 

### 步骤 2 

创建名为 *LB-backend* 的后端地址池。

	$beaddresspool = New-AzureRmLoadBalancerBackendAddressPoolConfig -Name LB-backend

## 创建 LB 规则、NAT 规则、探测器和负载均衡器

下面的示例将创建以下项：

- 用于将端口 3441 上的所有传入流量转换到端口 3389 的 NAT 规则
- 用于将端口 3442 上的所有传入流量转换到端口 3389 的 NAT 规则。
- 用于将端口 80 上的所有传入流量平衡到后端池中的地址端口 80 的负载均衡器规则。
- 探测规则，它将在名为 *HealthProbe.aspx* 的页上检查运行状况状态。
- 使用上面所有对象的负载均衡器。


### 步骤 1

创建 NAT 规则。

	$inboundNATRule1= New-AzureRmLoadBalancerInboundNatRuleConfig -Name RDP1 -FrontendIpConfiguration $frontendIP -Protocol TCP -FrontendPort 3441 -BackendPort 3389

	$inboundNATRule2= New-AzureRmLoadBalancerInboundNatRuleConfig -Name RDP2 -FrontendIpConfiguration $frontendIP -Protocol TCP -FrontendPort 3442 -BackendPort 3389

### 步骤 2

创建负载均衡器规则。

	$lbrule = New-AzureRmLoadBalancerRuleConfig -Name HTTP -FrontendIpConfiguration $frontendIP -BackendAddressPool  $beAddressPool -Probe $healthProbe -Protocol Tcp -FrontendPort 80 -BackendPort 80

### 步骤 3

创建运行状况探测器。有两种方法可以配置探测器：
 
HTTP 探测器
	
	$healthProbe = New-AzureRmLoadBalancerProbeConfig -Name HealthProbe -RequestPath 'HealthProbe.aspx' -Protocol http -Port 80 -IntervalInSeconds 15 -ProbeCount 2
或

TCP 探测器
	
	$healthProbe = New-AzureRmLoadBalancerProbeConfig -Name HealthProbe -Protocol Tcp -Port 80 -IntervalInSeconds 15 -ProbeCount 2

### 步骤 4

使用上面创建的对象创建负载均衡器。

	$NRPLB = New-AzureRmLoadBalancer -ResourceGroupName NRP-RG -Name NRP-LB -Location 'China East' -FrontendIpConfiguration $frontendIP -InboundNatRule $inboundNATRule1,$inboundNatRule2 -LoadBalancingRule $lbrule -BackendAddressPool $beAddressPool -Probe $healthProbe

## 创建 NIC

你需要创建网络接口（或修改现有网络接口），并将其关联到 NAT 规则、负载均衡器规则和探测器。

### 步骤 1 

获取需要创建 NIC 的虚拟网络和虚拟网络子网。

	$vnet = Get-AzureRmVirtualNetwork -Name NRPVNet -ResourceGroupName NRP-RG
	$backendSubnet = Get-AzureRmVirtualNetworkSubnetConfig -Name LB-Subnet-BE -VirtualNetwork $vnet 

### 步骤 2

创建名为 *lb-nic1-be* 的 NIC，并将其与第一个 NAT 规则和第一个（且仅有的）后端地址池相关联。
	
	$backendnic1= New-AzureRmNetworkInterface -ResourceGroupName NRP-RG -Name lb-nic1-be -Location 'China East' -PrivateIpAddress 10.0.2.6 -Subnet $backendSubnet -LoadBalancerBackendAddressPool $nrplb.BackendAddressPools[0] -LoadBalancerInboundNatRule $nrplb.InboundNatRules[0]

### 步骤 3

创建名为 *lb-nic2-be* 的 NIC，并将其与第二个 NAT 规则和第一个（且仅有的）后端地址池相关联。

	$backendnic2= New-AzureRmNetworkInterface -ResourceGroupName NRP-RG -Name lb-nic2-be -Location 'China East' -PrivateIpAddress 10.0.2.7 -Subnet $backendSubnet -LoadBalancerBackendAddressPool $nrplb.BackendAddressPools[0] -LoadBalancerInboundNatRule $nrplb.InboundNatRules[1]

### 步骤 4

检查 NIC。


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



### 步骤 5

使用 `Add-AzureRmVMNetworkInterface` cmdlet 将 NIC 分配给不同 VM。

你可以在 [Create and preconfigure a Windows Virtual Machine with Resource Manager and Azure PowerShell](/documentation/articles/virtual-machines-windows-ps-create#Example)（使用 Resource Manager 和 Azure PowerShell 创建并预配置 Windows 虚拟机）中使用示例中的选项 5，找到有关如何创建虚拟机和分配 NIC 的指南。


或者，如果你已创建虚拟机，则可以使用以下步骤添加网络接口：

#### 步骤 1 

将负载均衡器资源加载到变量中（如果你还没有这样做）。所用的变量名为 $lb，并使用前面创建的负载均衡器资源的相同名称。

	$lb= get-azurermloadbalancer –name NRP-LB -resourcegroupname NRP-RG

#### 步骤 2 

将后端配置加载到变量。

	$backend=Get-AzureRmLoadBalancerBackendAddressPoolConfig -name backendpool1 -LoadBalancer $lb

#### 步骤 3 

将已创建的网络接口加载到变量中。所用的变量名称为 $nic。所用的网络接口名称与前面的示例相同。

	$nic =get-azurermnetworkinterface –name lb-nic1-be -resourcegroupname NRP-RG

#### 步骤 4

更改网络接口上的后端配置。

	$nic.IpConfigurations[0].LoadBalancerBackendAddressPools=$backend

#### 步骤 5 

保存网络接口对象。

	Set-AzureRmNetworkInterface -NetworkInterface $nic

将网络接口添加到负载均衡器后端池后，它会根据该负载均衡器资源的负载均衡规则开始接收网络流量。

## 更新现有的负载均衡器


### 步骤 1

使用前面示例中的负载均衡器，通过 Get-AzureLoadBalancer 将负载均衡器对象分配给变量 $slb

	$slb = get-AzureRmLoadBalancer -Name NRPLB -ResourceGroupName NRP-RG

### 步骤 2

在以下示例中，你将在前端使用端口 81 添加新的入站 NAT 规则，并将后端池的端口 8181 添加到现有的负载均衡器

	$slb | Add-AzureRmLoadBalancerInboundNatRuleConfig -Name NewRule -FrontendIpConfiguration $slb.FrontendIpConfigurations[0] -FrontendPort 81  -BackendPort 8181 -Protocol TCP


### 步骤 3

使用 Set-AzureLoadBalancer 保存新配置

	$slb | Set-AzureRmLoadBalancer

## 删除负载均衡器

使用命令 `Remove-AzureLoadBalancer` 删除以前在名为“NRP-RG”的资源组中创建的名为“NRP-LB”的负载均衡器

	Remove-AzureRmLoadBalancer -Name NRPLB -ResourceGroupName NRP-RG

>[AZURE.NOTE] 你可以使用可选开关 -Force 来避免显示删除提示。

## 后续步骤

[开始配置内部负载均衡器](/documentation/articles/load-balancer-get-started-ilb-arm-ps/)

[配置负载均衡器分发模式](/documentation/articles/load-balancer-distribution-mode/)

[为负载均衡器配置空闲 TCP 超时设置](/documentation/articles/load-balancer-tcp-idle-timeout/)

<!---HONumber=Mooncake_0822_2016-->
