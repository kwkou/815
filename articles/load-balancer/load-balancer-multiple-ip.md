<properties
   pageTitle="在多个 IP 配置上进行负载均衡 | Azure"
   description="主要和辅助 IP 配置服务之间的负载均衡。"
   services="load-balancer"
   documentationCenter="na"
   authors="anavinahar"
   manager="narayan"
   editor="na" />
<tags
   ms.assetid="244907cd-b275-4494-aaf7-dcfc4d93edfe"
   ms.service="load-balancer"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="11/01/2016"
   wacn.date="01/05/2017"
   ms.author="annahar" />

# 在多个 IP 配置上进行负载均衡

本文介绍如何将 Azure Load Balancer 用于每个虚拟网络接口 (NIC) 的多个 IP 地址。目前，对一个 NIC 的多个 IP 地址的支持是预览版的功能。有关详细信息，请参阅本文的[限制](#limitations)部分。以下场景说明了如何通过负载均衡器使用此功能。

在本场景中有两个运行 Windows 的 VM，每个 VM 有一个 NIC。每个 NIC 具有多个 IP 配置。每个 VM 都托管了 contoso.com 和 fabrikam.com 这两个网站。每个网站都绑定到 NIC 的一个 IP 配置。我们使用负载均衡器来公开两个前端 IP 地址，每个地址分别对应于一个网站，从而将流量分发到各个网站的 IP 配置。此场景中两个前端以及两个后端池 IP 地址都使用相同的端口号。

![负载均衡应用场景图像](./media/load-balancer-multiple-ip/lb-multi-ip.PNG)  


## <a name="limitations"></a> 限制

现在，只有使用 Azure PowerShell 才能对辅助 IP 配置进行负载均衡配置。该限制是暂时性的，以后随时可能更改。重新访问此页以检查更新。

[AZURE.INCLUDE [virtual-network-preview](../../includes/virtual-network-preview.md)]

## 在多个 IP 配置上进行负载均衡的步骤

按照以下步骤来实现本文所概述的场景：

1. 安装 Azure PowerShell 中的说明进行操作。有关安装最新版本的 Azure PowerShell、选择订阅和登录帐户的信息，请参阅[如何安装和配置 Azure PowerShell](https://docs.microsoft.com/powershell/azureps-cmdlets-docs)。
2. 使用以下设置创建资源组：

        $location = "chinaeast".
        $myResourceGroup = "contosofabrikam"

有关详细信息，请参阅[创建资源组](/documentation/articles/virtual-machines-windows-ps-create/)的第 2 步。

3. [创建用于包含 VM 的可用性集](/documentation/articles/virtual-machines-windows-create-availability-set/)。对于此场景，请使用以下命令：

        New-AzureRmAvailabilitySet -ResourceGroupName "contosofabrikam" -Name "myAvailset" -Location "China East"

4. 按照[创建 Windows VM](/documentation/articles/virtual-machines-windows-ps-create/) 中步骤 3 至 5 的说明准备创建具有单个 NIC 的 VM。执行步骤 6.1，使用以下命令而不是步骤 6.2：

        $availset = Get-AzureRmAvailabilitySet -ResourceGroupName "contosofabrikam" -Name "myAvailset"
        New-AzureRmVMConfig -VMName "VM1" -VMSize "Standard_DS1_v2" -AvailabilitySetId $availset.Id

然后完成[创建 Windows VM](/documentation/articles/virtual-machines-windows-ps-create/) 的步骤 6.3 至 6.8。

5. 向每个 VM 中添加另一个 IP 配置。按照[将多个 IP 地址分配给虚拟机](/documentation/articles/virtual-network-multiple-ip-addresses-powershell#add)文章中的说明执行操作。请使用以下配置设置：

        $NicName = "VM1-NIC"
        $RgName = "contosofabrikam"
        $NicLocation = "China East"
        $IPConfigName4 = "VM1-ipconfig2"
        $Subnet1 = Get-AzureRmVirtualNetworkSubnetConfig -Name "mySubnet" -VirtualNetwork $myVnet

在本文中无需将辅助 IP 配置与公共 IP 关联。请编辑此命令以删除公共 IP 关联部分。

6. 针对 VM2 再次完成本文的步骤 4 到 6。执行此操作时务必将 VM 名称替换为 VM2。请注意，无需为第二个 VM 创建虚拟网络。用户可以根据自己的用例创建或不创建新的子网。

7. 创建两个公共 IP 地址并将它们存储在相应的变量中，如下所示：

        $publicIP1 = New-AzureRmPublicIpAddress -Name PublicIp1 -ResourceGroupName contosofabrikam -Location 'China East' -AllocationMethod Dynamic -DomainNameLabel contoso
        $publicIP2 = New-AzureRmPublicIpAddress -Name PublicIp2 -ResourceGroupName contosofabrikam -Location 'China East' -AllocationMethod Dynamic -DomainNameLabel fabrikam

        $publicIP1 = Get-AzureRmPublicIpAddress -Name PublicIp1 -ResourceGroupName contosofabrikam
        $publicIP2 = Get-AzureRmPublicIpAddress -Name PublicIp2 -ResourceGroupName contosofabrikam

8. 创建两个前端 IP 配置：

        $frontendIP1 = New-AzureRmLoadBalancerFrontendIpConfig -Name contosofe -PublicIpAddress $publicIP1
        $frontendIP2 = New-AzureRmLoadBalancerFrontendIpConfig -Name fabrikamfe -PublicIpAddress $publicIP2

9. 创建后端地址池、探测程序和负载均衡规则：

        $beaddresspool1 = New-AzureRmLoadBalancerBackendAddressPoolConfig -Name contosopool
        $beaddresspool2 = New-AzureRmLoadBalancerBackendAddressPoolConfig -Name fabrikampool

        $healthProbe = New-AzureRmLoadBalancerProbeConfig -Name HTTP -RequestPath 'index.html' -Protocol http -Port 80 -IntervalInSeconds 15 -ProbeCount 2

        $lbrule1 = New-AzureRmLoadBalancerRuleConfig -Name HTTPc -FrontendIpConfiguration $frontendIP1 -BackendAddressPool $beaddresspool1 -Probe $healthprobe -Protocol Tcp -FrontendPort 80 -BackendPort 80
        $lbrule2 = New-AzureRmLoadBalancerRuleConfig -Name HTTPf -FrontendIpConfiguration $frontendIP2 -BackendAddressPool $beaddresspool2 -Probe $healthprobe -Protocol Tcp -FrontendPort 80 -BackendPort 80

10. 一旦创建这些资源后，即可创建负载均衡器：

        $mylb = New-AzureRmLoadBalancer -ResourceGroupName contosofabrikam -Name mylb -Location 'China East' -FrontendIpConfiguration $frontendIP1 -LoadBalancingRule $lbrule -BackendAddressPool $beAddressPool -Probe $healthProbe

11. 将第二个后端地址池和前端 IP 配置添加到新创建的负载均衡器：

        $mylb = Get-AzureRmLoadBalancer -Name "mylb" -ResourceGroupName $myResourceGroup | Add-AzureRmLoadBalancerBackendAddressPoolConfig -Name fabrikampool | Set-AzureRmLoadBalancer

        $mylb | Add-AzureRmLoadBalancerFrontendIpConfig -Name fabrikamfe -PublicIpAddress $publicIP2 | Set-AzureRmLoadBalancer
    
        Add-AzureRmLoadBalancerRuleConfig -Name HTTP -LoadBalancer $mylb -FrontendIpConfiguration $frontendIP2 -BackendAddressPool $beaddresspool2 -Probe $healthProbe -Protocol Tcp -FrontendPort 80 -BackendPort 80 | Set-AzureRmLoadBalancer

12. 下面的命令获取 NIC，然后将每个 NIC 的两个 IP 配置添加到负载均衡器的后端地址池：

        $nic1 = Get-AzureRmNetworkInterface -Name "VM1-NIC" -ResourceGroupName "MyResourcegroup";
        $nic2 = Get-AzureRmNetworkInterface -Name "VM2-NIC" -ResourceGroupName "MyResourcegroup";

        $nic1.IpConfigurations[0].LoadBalancerBackendAddressPools.Add($mylb.BackendAddressPools[0]);
        $nic1.IpConfigurations[1].LoadBalancerBackendAddressPools.Add($mylb.BackendAddressPools[1]);
        $nic2.IpConfigurations[0].LoadBalancerBackendAddressPools.Add($mylb.BackendAddressPools[0]);
        $nic2.IpConfigurations[1].LoadBalancerBackendAddressPools.Add($mylb.BackendAddressPools[1]);

        $mylb = $mylb | Set-AzureRmLoadBalancer

        $nic1 | Set-AzureRmNetworkInterface
        $nic2 | Set-AzureRmNetworkInterface

13. 最后，必须将 DNS 资源记录配置为指向各自的负载均衡器的前端 IP 地址。

<!---HONumber=Mooncake_1128_2016-->