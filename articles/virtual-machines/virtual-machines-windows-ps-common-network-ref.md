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
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/27/2016"
	wacn.date="01/05/2017"
	ms.author="davidmu"/>  


# VM 的常用网络 Azure PowerShell 命令

如果希望创建虚拟机，需要创建[虚拟网络](/documentation/articles/virtual-networks-overview/)或对能够添加 VM 的现有虚拟网络比较了解。通常情况下，创建 VM 时，还需考虑创建本文所述资源。

有关安装最新版本的 Azure PowerShell、选择订阅和登录帐户的信息，请参阅[如何安装和配置 Azure PowerShell](https://docs.microsoft.com/powershell/azureps-cmdlets-docs)。

## 创建网络资源

任务 | 命令 
-------------- | -------------------------
创建子网配置 | $subnet1 = [New-AzureRmVirtualNetworkSubnetConfig](https://msdn.microsoft.com/zh-cn/library/mt619412.aspx) -Name "subnet\_name" -AddressPrefix XX.X.X.X/XX<BR>$subnet2 = New-AzureRmVirtualNetworkSubnetConfig -Name "subnet\_name" -AddressPrefix XX.X.X.X/XX<BR><BR>典型的网络可能具有一个用于[面向 Internet 的负载均衡器](/documentation/articles/load-balancer-internet-overview/)的子网和另一个用于[内部负载均衡器](/documentation/articles/load-balancer-internal-overview/)的子网。 |
创建虚拟网络 | $vnet = [New-AzureRmVirtualNetwork](https://msdn.microsoft.com/zh-cn/library/mt603657.aspx) -Name "virtual\_network\_name" -ResourceGroupName "resource\_group\_name" -Location "location\_name" -AddressPrefix XX.X.X.X/XX -Subnet $subnet1, $subnet2
唯一域名的测试 | [Test-AzureRmDnsAvailability](https://msdn.microsoft.com/zh-cn/library/mt619419.aspx) -DomainQualifiedName "domain\_name" -Location "location\_name"<BR><BR>可以为[公共 IP 资源](/documentation/articles/virtual-network-ip-addresses-overview-arm/)指定一个 DNS 域名，以便在 Azure 托管的 DNS 服务器中为 domainname.location.chinacloudapp.cn 创建目标为公共 IP 地址的映射。字段只能包含字母、数字和连字符。第一个和最后一个字符必须是字母或数字，域名在其 Azure 位置内必须是唯一的。如果返回 **True**，则建议的名称是全局唯一的。
创建公共 IP 地址 | $pip = [New-AzureRmPublicIpAddress](https://msdn.microsoft.com/zh-cn/library/mt603620.aspx) -Name "ip\_address\_name" -ResourceGroupName "resource\_group\_name" -DomainNameLabel "domain\_name" -Location "location\_name" -AllocationMethod Dynamic<BR><BR>公共 IP 地址使用以前测试的域名，并为负载均衡器的前端配置所用。
创建前端 IP 配置 | $frontendIP = [New-AzureRmLoadBalancerFrontendIpConfig](https://msdn.microsoft.com/zh-cn/library/mt603510.aspx) -Name "frontend\_ip\_name" -PublicIpAddress $pip<BR><BR>前端配置包括以前为传入网络流量创建的公共 IP 地址。
创建后端地址池 | $beAddressPool = [New-AzureRmLoadBalancerBackendAddressPoolConfig](https://msdn.microsoft.com/zh-cn/library/mt603791.aspx) -Name "backend\_pool\_name"<BR><BR>为通过网络接口访问的负载均衡器后端提供内部地址。
创建探测 | $healthProbe = [New-AzureRmLoadBalancerProbeConfig](https://msdn.microsoft.com/zh-cn/library/mt603847.aspx) -Name "probe\_name" -RequestPath 'HealthProbe.aspx' -Protocol http -Port 80 -IntervalInSeconds 15 -ProbeCount 2<BR><BR>包含用于检查后端地址池中虚拟机实例可用性的运行状况探测。
创建负载均衡规则 | $lbRule = [New-AzureRmLoadBalancerRuleConfig](https://msdn.microsoft.com/zh-cn/library/mt619391.aspx) -Name HTTP -FrontendIpConfiguration $frontendIP -BackendAddressPool $beAddressPool -Probe $healthProbe -Protocol Tcp -FrontendPort 80 -BackendPort 80<BR><BR>包含将负载均衡器上公用端口分配给后端地址池中端口的规则。
创建入站 NAT 规则 | $inboundNATRule = [New-AzureRmLoadBalancerInboundNatRuleConfig](https://msdn.microsoft.com/zh-cn/library/mt603606.aspx) -Name "rule\_name" -FrontendIpConfiguration $frontendIP -Protocol TCP -FrontendPort 3441 -BackendPort 3389<BR><BR>包含将负载均衡器上公用端口映射到后端地址池中特定虚拟机端口的规则。
创建负载均衡器 | $loadBalancer = [New-AzureRmLoadBalancer](https://msdn.microsoft.com/zh-cn/library/mt619450.aspx) -ResourceGroupName "resource\_group\_name" -Name "load\_balancer\_name" -Location "location\_name" -FrontendIpConfiguration $frontendIP -InboundNatRule $inboundNATRule -LoadBalancingRule $lbRule -BackendAddressPool $beAddressPool -Probe $healthProbe
创建网络接口 | $nic1= [New-AzureRmNetworkInterface](https://msdn.microsoft.com/zh-cn/library/mt619370.aspx) -ResourceGroupName "resource\_group\_name" -Name "network\_interface\_name" -Location "location\_name" -PrivateIpAddress XX.X.X.X -Subnet subnet2 -LoadBalancerBackendAddressPool $loadBalancer.BackendAddressPools[0] -LoadBalancerInboundNatRule $loadBalancer.InboundNatRules[0]<BR><BR>使用以前创建的公用 IP 地址和虚拟网络子网创建网络接口。
	
## 获取有关网络资源的信息

任务 | 命令 
-------------- | -------------------------
列出虚拟网络 | [Get-AzureRmVirtualNetwork](https://msdn.microsoft.com/zh-cn/library/mt603515.aspx) -ResourceGroupName "resource\_group\_name"<BR><BR>列出资源组中的所有虚拟网络。
获取有关虚拟网络的信息 | Get-AzureRmVirtualNetwork -Name "virtual\_network\_name" -ResourceGroupName "resource\_group\_name"
列出虚拟网络中的子网 | Get-AzureRmVirtualNetwork -Name "virtual\_network\_name" -ResourceGroupName "resource\_group\_name" | Select Subnets
获取有关子网的信息 | [Get-AzureRmVirtualNetworkSubnetConfig](https://msdn.microsoft.com/zh-cn/library/mt603817.aspx) -Name "subnet\_name" -VirtualNetwork $vnet<BR><BR>获取指定虚拟网络中子网的信息。$vnet 值表示 Get-AzureRmVirtualNetwork 返回的对象。
列出 IP 地址 | [Get-AzureRmPublicIpAddress](https://msdn.microsoft.com/zh-cn/library/mt619342.aspx) -ResourceGroupName "resource\_group\_name"<BR><BR>列出资源组中的公共 IP 地址。
列出负载均衡器 | [Get-AzureRmLoadBalancer](https://msdn.microsoft.com/zh-cn/library/mt603668.aspx) -ResourceGroupName "resource\_group\_name"<BR><BR>列出资源组中的所有负载均衡器。
列出网络接口 | [Get-AzureRmNetworkInterface](https://msdn.microsoft.com/zh-cn/library/mt619434.aspx) -ResourceGroupName "resource\_group\_name"<BR><BR>列出资源组中的所有网络接口。
获取有关网络接口的信息 | Get-AzureRmNetworkInterface -Name "network\_interface\_name" -ResourceGroupName "resource\_group\_name"<BR><BR>获取有关特定网络接口的信息。
获取网络接口的 IP 配置 | [Get-AzureRmNetworkInterfaceIPConfig](https://msdn.microsoft.com/zh-cn/library/mt732618.aspx) -Name "ipconfiguration\_name" -NetworkInterface $nic<BR><BR>获取有关指定网络接口的 IP 配置的信息。$nic 值表示 Get-AzureRmNetworkInterface 返回的对象。

## 管理网络资源

任务 | 命令 
-------------- | -------------------------
将子网添加到虚拟网络 | [Add-AzureRmVirtualNetworkSubnetConfig](https://msdn.microsoft.com/zh-cn/library/mt603722.aspx) -AddressPrefix XX.X.X.X/XX -Name "subnet\_name" -VirtualNetwork $vnet<BR><BR>将子网添加到现有虚拟网络。$vnet 值表示 Get-AzureRmVirtualNetwork 返回的对象。
删除虚拟网络 | [Remove-AzureRmVirtualNetwork](https://msdn.microsoft.com/zh-cn/library/mt619338.aspx) -Name "virtual\_network\_name" -ResourceGroupName "resource\_group\_name"<BR><BR>从资源组中删除指定的虚拟网络。
删除网络接口 | [Remove-AzureRmNetworkInterface](https://msdn.microsoft.com/zh-cn/library/mt603836.aspx) -Name "network\_interface\_name" -ResourceGroupName "resource\_group\_name"<BR><BR>从资源组中删除指定的网络接口。
删除负载均衡器 | [Remove-AzureRmLoadBalancer](https://msdn.microsoft.com/zh-cn/library/mt603862.aspx) -Name "load\_balancer\_name" -ResourceGroupName "resource\_group\_name"<BR><BR>从资源组中删除指定的负载均衡器。
删除公共 IP 地址 | [Remove-AzureRmPublicIpAddress](https://msdn.microsoft.com/zh-cn/library/mt619352.aspx)-Name "ip\_address\_name" -ResourceGroupName "resource\_group\_name"<BR><BR>从资源组中删除指定的公共 IP 地址。

## 后续步骤

- 使用在[创建 VM](/documentation/articles/virtual-machines-windows-ps-create/) 时创建的网络接口。
- 了解如何才能[创建具有多个网络接口的 VM](/documentation/articles/virtual-networks-multiple-nics/)。

<!---HONumber=Mooncake_1121_2016-->