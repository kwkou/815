<properties
	pageTitle="VM 的常用网络 PowerShell 命令 | Azure"
	description="可用于为 VM 创建虚拟网络及其关联资源的常用 PowerShell 命令。"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="davidmu1"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="06/07/2016"
	wacn.date="07/11/2016"/>

# VM 的常用网络 Azure PowerShell 命令

如果你想要创建虚拟机，则需要创建[虚拟网络](/documentation/articles/virtual-networks-overview/)或了解可在其中添加 VM 的现有虚拟网络。通常情况下，创建 VM 时，还需考虑创建本文所述资源。

## 使用 Azure PowerShell 创建资源

有关如何安装最新版 Azure PowerShell 的信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。选择要使用的订阅，然后登录到你的 Azure 帐户。

资源 | 命令 
-------------- | -------------------------
子网 | $internetSubnet = [New-AzureRmVirtualNetworkSubnetConfig](https://msdn.microsoft.com/zh-cn/library/mt619412.aspx) -Name internetSubnet -AddressPrefix 10.0.1.0/24<BR>$internalSubnet = New-AzureRmVirtualNetworkSubnetConfig -Name internalSubnet -AddressPrefix 10.0.2.0/24<BR><BR>典型网络可能具有一个用于面向 Internet 的负载平衡器的子网和另一个用于内部负载平衡器的子网。 |
虚拟网络 | 创建虚拟网络：<BR><BR>$rgName = "[resource-group-name](/documentation/articles/powershell-azure-resource-manager/)"<BR>$locName = "[location-name](https://msdn.microsoft.com/zh-cn/library/azure/dn495177.aspx)"<BR>$vnetName = "virtual-network-name"<BR>$vnet = [New-AzureRmVirtualNetwork](https://msdn.microsoft.com/zh-cn/library/mt603657.aspx) -Name $vnetName -ResourceGroupName $rgName -Location $locName -AddressPrefix 10.0.0.0/16 -Subnet $internetSubnet,$internalSubnet<BR><BR>获取虚拟网络列表：<BR><BR>[Get-AzureRmVirtualNetwork](https://msdn.microsoft.com/zh-cn/library/mt603515.aspx) -ResourceGroupName $rgName &#124; Sort Name &#124; Select Name<BR><BR>列出虚拟网络中的子网：<BR><BR>Get-AzureRmVirtualNetwork -Name $vnetName -ResourceGroupName $rgName &#124; Select Subnets<BR><BR>应可看到类似如下的结果，显示子网列表：<BR><BR>Subnets<BR>-------<BR>{internetSubnet, internalSubnet}<BR><BR>internetSubnet 的子网索引为 0，internalSubnet 的子网索引为 1。
域名标签 | $domName = "domain-name"<BR>[Test-AzureRmDnsAvailability](https://msdn.microsoft.com/zh-cn/library/mt619419.aspx) -DomainQualifiedName $domName -Location $locName<BR><BR>你可以为[公共 IP 资源](/documentation/articles/virtual-network-ip-addresses-overview-arm/)指定一个 DNS 域名标签，以便在 Azure 托管的 DNS 服务器中为 domainnamelabel.location.chinacloudapp.cn 创建目标为公共 IP 地址的映射。该标签只能包含字母、数字和连字符。第一个和最后一个字符必须是字母或数字，域名标签在其 Azure 位置内必须是唯一的。应始终测试域名标签是否全局唯一。如果返回 **True**，则你建议的名称是全局唯一的。
公共 IP 地址 | $ipName = "public-ip-address-name"<BR>$pip = [New-AzureRmPublicIpAddress](https://msdn.microsoft.com/zh-cn/library/mt603620.aspx) -Name $ipName -ResourceGroupName $rgName -DomainNameLabel $domName -Location $locName -AllocationMethod Dynamic<BR><BR>公共 IP 地址使用你以前创建的域名标签，并为负载平衡器的前端配置所用。
前端 IP 配置 | $feConfigName = "frontend-ip-config-name"<BR>$frontendIP = [New-AzureRmLoadBalancerFrontendIpConfig](https://msdn.microsoft.com/zh-cn/library/mt603510.aspx) -Name $feConfigName -PublicIpAddress $pip<BR><BR>前端配置包括你以前为传入网络流量创建的公共 IP 地址。
后端地址池 | $bePoolName = "back-end-pool-name"<BR>$beAddressPool = [New-AzureRmLoadBalancerBackendAddressPoolConfig](https://msdn.microsoft.com/zh-cn/library/mt603791.aspx) -Name $bePoolName<BR><BR>为通过网络接口访问的负载平衡器后端提供内部地址。
探测 | $probeName = "health-probe-name"<BR>$healthProbe = [New-AzureRmLoadBalancerProbeConfig](https://msdn.microsoft.com/zh-cn/library/mt603847.aspx) -Name $probeName -RequestPath 'HealthProbe.aspx' -Protocol http -Port 80 -IntervalInSeconds 15 -ProbeCount 2<BR><BR>包含用于检查后端地址池中虚拟机实例可用性的运行状况探测。
负载平衡规则 | $lbRule = [New-AzureRmLoadBalancerRuleConfig](https://msdn.microsoft.com/zh-cn/library/mt619391.aspx) -Name HTTP -FrontendIpConfiguration $frontendIP -BackendAddressPool $beAddressPool -Probe $healthProbe -Protocol Tcp -FrontendPort 80 -BackendPort 80<BR><BR>包含将负载平衡器上公用端口映射到后端地址池中端口的规则。
入站 NAT 规则 | $ruleName = "NAT-rule-name"<BR>$inboundNATRule = [New-AzureRmLoadBalancerInboundNatRuleConfig](https://msdn.microsoft.com/zh-cn/library/mt603606.aspx) -Name $ruleName -FrontendIpConfiguration $frontendIP -Protocol TCP -FrontendPort 3441 -BackendPort 3389<BR><BR>包含将负载平衡器上公用端口映射到后端地址池中特定虚拟机端口的规则。
负载平衡器 | $lbName = "load-balancer-name"<BR>$loadBalancer = [New-AzureRmLoadBalancer](https://msdn.microsoft.com/zh-cn/library/mt619450.aspx) -ResourceGroupName $rgName -Name $lbName -Location $locName -FrontendIpConfiguration $frontendIP -InboundNatRule $inboundNATRule -LoadBalancingRule $lbRule -BackendAddressPool $beAddressPool -Probe $healthProbe
网络接口 | $vnet = [Get-AzureRmVirtualNetwork](https://msdn.microsoft.com/zh-cn/library/mt603515.aspx) -Name $vnetName -ResourceGroupName $rgName<BR>$internalSubnet = [Get-AzureRmVirtualNetworkSubnetConfig](https://msdn.microsoft.com/zh-cn/library/mt603817.aspx) -Name "internalSubnet" -VirtualNetwork $vnet<BR>$nicName = "network-interface-name"<BR>$nic1= [New-AzureRmNetworkInterface](https://msdn.microsoft.com/zh-cn/library/mt619370.aspx) -ResourceGroupName $rgName -Name $nicName -Location $locName -PrivateIpAddress 10.0.2.6 -Subnet $internalSubnet -LoadBalancerBackendAddressPool $loadBalancer.BackendAddressPools[0] -LoadBalancerInboundNatRule $loadBalancer.InboundNatRules[0]<BR><BR>使用你以前创建的公用 IP 地址和虚拟网络子网创建网络接口。
	
## 后续步骤

- 使用[创建 VM](/documentation/articles/virtual-machines-windows-ps-create/) 时创建的网络接口。
- 了解如何才能[创建具有多个网络接口的 VM](/documentation/articles/virtual-networks-multiple-nics/)。

<!---HONumber=Mooncake_0704_2016-->