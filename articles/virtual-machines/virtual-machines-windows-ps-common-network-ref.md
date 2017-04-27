<properties
    pageTitle="适用于 Azure 虚拟网络的常见 PowerShell 命令 | Azure"
    description="可用于为 VM 创建虚拟网络及其关联资源的常用 PowerShell 命令。"
    services="virtual-machines-windows"
    documentationcenter=""
    author="davidmu1"
    manager="timlt"
    editor=""
    tags="azure-resource-manager"
    translationtype="Human Translation" />
<tags
    ms.assetid="56e1a73c-8299-4996-bd03-f74585caa1dc"
    ms.service="virtual-machines-windows"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/02/2017"
    wacn.date="04/17/2017"
    ms.author="davidmu"
    ms.sourcegitcommit="e0e6e13098e42358a7eaf3a810930af750e724dd"
    ms.openlocfilehash="c661878795039483fbec373ed5bb7b646d9d7ac2"
    ms.lasthandoff="04/06/2017" />

# <a name="common-powershell-commands-for-azure-virtual-networks"></a>适用于 Azure 虚拟网络的常见 PowerShell 命令

如果想要创建虚拟机，需要创建[虚拟网络](/documentation/articles/virtual-networks-overview/)或了解可在其中添加 VM 的现有虚拟网络。 通常情况下，创建 VM 时，还需考虑创建本文所述资源。

有关安装最新版 Azure PowerShell、选择订阅和登录到帐户的信息，请参阅[如何安装和配置 Azure PowerShell](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs)。

如果运行本文中的多个命令，以下一些变量可能将对你有用：

- $location - 网络资源的位置。 可以使用 [Get-AzureRmLocation](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.resources/v3.5.0/get-azurermlocation) 查找适合你的地理区域。
- $myResourceGroup - 网络资源所在的资源组名称。

## <a name="create-network-resources"></a>创建网络资源

| 任务 | 命令 |
| ---- | ------- |
| 创建子网配置 |$subnet1 = [New-AzureRmVirtualNetworkSubnetConfig](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/AzureRM.Network/v3.4.0/New-AzureRmVirtualNetworkSubnetConfig) -Name "mySubnet1" -AddressPrefix XX.X.X.X/XX<BR>$subnet2 = New-AzureRmVirtualNetworkSubnetConfig -Name "mySubnet2" -AddressPrefix XX.X.X.X/XX<BR><BR>典型的网络可能包含用于[面向 Internet 的负载均衡器](/documentation/articles/load-balancer-internet-overview/)的子网，以及用于[内部负载均衡器](/documentation/articles/load-balancer-internal-overview/)的独立子网。 |
| 创建虚拟网络 |$vnet = [New-AzureRmVirtualNetwork](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/AzureRM.Network/v3.4.0/New-AzureRmVirtualNetwork) -Name "myVNet" -ResourceGroupName $myResourceGroup -Location $location -AddressPrefix XX.X.X.X/XX -Subnet $subnet1, $subnet2 |
| 测试唯一域名 |[Test-AzureRmDnsAvailability](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/AzureRM.Network/v3.4.0/Test-AzureRmDnsAvailability) -DomainNameLabel "myDNS" -Location $location<BR><BR>可以为[公共 IP 资源](/documentation/articles/virtual-network-ip-addresses-overview-arm/)指定一个 DNS 域名，以便在 Azure 托管的 DNS 服务器中创建 domainname.location.chinacloudapp.cn 到公共 IP 地址的映射。 字段只能包含字母、数字和连字符。 第一个和最后一个字符必须是字母或数字，域名在其 Azure 位置内必须是唯一的。 如果返回 **True** ，则建议的名称是全局唯一的。 |
| 创建公共 IP 地址 |$pip = [New-AzureRmPublicIpAddress](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/AzureRM.Network/v3.4.0/New-AzureRmPublicIpAddress) -Name "myPublicIp" -ResourceGroupName $myResourceGroup -DomainNameLabel "myDNS" -Location $location -AllocationMethod Dynamic<BR><BR>公共 IP 地址使用前面测试过的并由负载均衡器前端配置使用的域名。 |
| 创建前端 IP 配置 |$frontendIP = [New-AzureRmLoadBalancerFrontendIpConfig](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/AzureRM.Network/v3.4.0/New-AzureRmLoadBalancerFrontendIpConfig) -Name "myFrontendIP" -PublicIpAddress $pip<BR><BR>前端配置包括前面针对传入网络流量创建的公共 IP 地址。 |
| 创建后端地址池 |$beAddressPool = [New-AzureRmLoadBalancerBackendAddressPoolConfig](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/AzureRM.Network/v3.4.0/New-AzureRmLoadBalancerBackendAddressPoolConfig) -Name "myBackendAddressPool"<BR><BR>提供可通过网络接口访问的负载均衡器后端内部地址。 |
| 创建探测 |$healthProbe = [New-AzureRmLoadBalancerProbeConfig](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/AzureRM.Network/v3.4.0/New-AzureRmLoadBalancerProbeConfig) -Name "myProbe" -RequestPath 'HealthProbe.aspx' -Protocol http -Port 80 -IntervalInSeconds 15 -ProbeCount 2<BR><BR>包含用于检查后端地址池中虚拟机实例的可用性的运行状况探测。 |
| 创建负载均衡规则 |$lbRule = [New-AzureRmLoadBalancerRuleConfig](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/AzureRM.Network/v3.4.0/New-AzureRmLoadBalancerRuleConfig) -Name HTTP -FrontendIpConfiguration $frontendIP -BackendAddressPool $beAddressPool -Probe $healthProbe -Protocol Tcp -FrontendPort 80 -BackendPort 80<BR><BR>包含可将负载均衡器上的公共端口分配到后端地址池中的端口的规则。 |
| 创建入站 NAT 规则 |$inboundNATRule = [New-AzureRmLoadBalancerInboundNatRuleConfig](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/AzureRM.Network/v3.4.0/New-AzureRmLoadBalancerInboundNatRuleConfig) -Name "myInboundRule1" -FrontendIpConfiguration $frontendIP -Protocol TCP -FrontendPort 3441 -BackendPort 3389<BR><BR>包含将负载均衡器上的公共端口映射到后端地址池中特定虚拟机的端口的规则。 |
| 创建负载均衡器 |$loadBalancer = [New-AzureRmLoadBalancer](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/AzureRM.Network/v3.4.0/New-AzureRmLoadBalancer) -ResourceGroupName $myResourceGroup -Name "myLoadBalancer" -Location $location -FrontendIpConfiguration $frontendIP -InboundNatRule $inboundNATRule -LoadBalancingRule $lbRule -BackendAddressPool $beAddressPool -Probe $healthProbe |
| 创建网络接口 |$nic1= [New-AzureRmNetworkInterface](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/AzureRM.Network/v3.4.0/New-AzureRmNetworkInterface) -ResourceGroupName $myResourceGroup -Name "myNIC" -Location $location -PrivateIpAddress XX.X.X.X -Subnet $subnet2 -LoadBalancerBackendAddressPool $loadBalancer.BackendAddressPools[0] -LoadBalancerInboundNatRule $loadBalancer.InboundNatRules[0]<BR><BR>创建一个使用前面所创建公共 IP 地址和虚拟网络子网的网络接口。 |

## <a name="get-information-about-network-resources"></a>获取有关网络资源的信息

| 任务 | 命令 |
| ---- | ------- |
| 列出虚拟网络 |[Get-AzureRmVirtualNetwork](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/AzureRM.Network/v3.4.0/Get-AzureRmVirtualNetwork) -ResourceGroupName $myResourceGroup<BR><BR>列出资源组中的所有虚拟网络。 |
| 获取有关虚拟网络的信息 |Get-AzureRmVirtualNetwork -Name "myVNet" -ResourceGroupName $myResourceGroup |
| 列出虚拟网络中的子网 |Get-AzureRmVirtualNetwork -Name "myVNet" -ResourceGroupName $myResourceGroup &#124; Select Subnets |
| 获取有关子网的信息 |[Get-AzureRmVirtualNetworkSubnetConfig](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/AzureRM.Network/v3.4.0/Get-AzureRmVirtualNetworkSubnetConfig) -Name "mySubnet1" -VirtualNetwork $vnet<BR><BR>获取指定虚拟网络中子网的相关信息。 $vnet 值表示 Get-AzureRmVirtualNetwork 返回的对象。 |
| 列出 IP 地址 |[Get-AzureRmPublicIpAddress](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/AzureRM.Network/v3.4.0/Get-AzureRmPublicIpAddress) -ResourceGroupName $myResourceGroup<BR><BR>列出资源组中的公共 IP 地址。 |
| 列出负载均衡器 |[Get-AzureRmLoadBalancer](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/AzureRM.Network/v3.4.0/Get-AzureRmLoadBalancer) -ResourceGroupName $myResourceGroup<BR><BR>列出资源组中的所有负载均衡器。 |
| 列出网络接口 |[Get-AzureRmNetworkInterface](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/AzureRM.Network/v3.4.0/Get-AzureRmNetworkInterface) -ResourceGroupName $myResourceGroup<BR><BR>列出资源组中的所有网络接口。 |
| 获取有关网络接口的信息 |Get-AzureRmNetworkInterface -Name "myNIC" -ResourceGroupName $myResourceGroup<BR><BR>获取有关特定网络接口的信息。 |
| 获取网络接口的 IP 配置 |[Get-AzureRmNetworkInterfaceIPConfig](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/AzureRM.Network/v3.4.0/Get-AzureRmNetworkInterfaceIpConfig) -Name "myNICIP" -NetworkInterface $nic<BR><BR>获取指定网络接口的 IP 配置的相关信息。 $nic 值表示 Get-AzureRmNetworkInterface 返回的对象。 |

## <a name="manage-network-resources"></a>管理网络资源

| 任务 | 命令 |
| ---- | ------- |
| 将子网添加到虚拟网络 |[Add-AzureRmVirtualNetworkSubnetConfig](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/AzureRM.Network/v3.4.0/Add-AzureRmVirtualNetworkSubnetConfig) -AddressPrefix XX.X.X.X/XX -Name "mySubnet1" -VirtualNetwork $vnet<BR><BR>将子网添加到现有虚拟网络。 $vnet 值表示 Get-AzureRmVirtualNetwork 返回的对象。 |
| 删除虚拟网络 |[Remove-AzureRmVirtualNetwork](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/AzureRM.Network/v3.4.0/Remove-AzureRmVirtualNetwork) -Name "myVNet" -ResourceGroupName $myResourceGroup<BR><BR>从资源组中删除指定的虚拟网络。 |
| 删除网络接口 |[Remove-AzureRmNetworkInterface](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/AzureRM.Network/v3.4.0/Remove-AzureRmNetworkInterface) -Name "myNIC" -ResourceGroupName $myResourceGroup<BR><BR>从资源组中删除指定的网络接口。 |
| 删除负载均衡器 |[Remove-AzureRmLoadBalancer](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/AzureRM.Network/v3.4.0/Remove-AzureRmLoadBalancer) -Name "myLoadBalancer" -ResourceGroupName $myResourceGroup<BR><BR>从资源组中删除指定的负载均衡器。 |
| 删除公共 IP 地址 |[Remove-AzureRmPublicIpAddress](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/AzureRM.Network/v3.4.0/Remove-AzureRmPublicIpAddress)-Name "myIPAddress" -ResourceGroupName $myResourceGroup<BR><BR>从资源组中删除指定的公共 IP 地址。 |

## <a name="next-steps"></a>后续步骤
* 使用[创建 VM](/documentation/articles/virtual-machines-windows-ps-create/) 时创建的网络接口。
* 了解如何[创建具有多个网络接口的 VM](/documentation/articles/virtual-networks-multiple-nics/)。
<!--Update_Description: wording update-->