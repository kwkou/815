<properties
    pageTitle="使用 PowerShell cmdlet 创建虚拟机规模集 | Azure"
    description="开始使用 Azure PowerShell cmdlet 来创建和管理你的第一个 Azure 虚拟机规模集"
    services="virtual-machines-windows"
    documentationcenter=""
    author="danielsollondon"
    manager="timlt"
    editor=""
    tags="azure-resource-manager"
    translationtype="Human Translation" />
<tags
    ms.assetid="430d9d64-1f35-48f0-a4fd-9b69910ffa59"
    ms.service="virtual-machines-windows"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/21/2017"
    wacn.date="04/17/2017"
    ms.author="danielsollondon"
    ms.sourcegitcommit="e0e6e13098e42358a7eaf3a810930af750e724dd"
    ms.openlocfilehash="70f3e0a9e54d46691a986773be4e7f9a23d23e6d"
    ms.lasthandoff="04/06/2017" />

# <a name="creating-virtual-machine-scale-sets-using-powershell-cmdlets"></a>使用 PowerShell cmdlet 创建虚拟机规模集
本文演示如何创建虚拟机规模集 (VMSS) 的示例。 该示例创建三个节点的规模集，包括相关联的网络和存储。

## <a name="first-steps"></a>前几个步骤
务必安装最新的 Azure PowerShell 模块，以确保具有维护和创建规模集所需的 PowerShell cmdlet。
转到[此处](http://aka.ms/webpi-azps)的命令行工具，获取最新的可用 Azure 模块。

若要查找 VMSS 相关的 cmdlet，请使用搜索字符串 \*VMSS\*。 例如，_gcm \*vmss\*_

## <a name="creating-a-vmss"></a>创建 VMSS
#### <a name="create-resource-group"></a>创建资源组

    $loc = 'chinanorth';
    $rgname = 'mynewrgwu';
      New-AzureRmResourceGroup -Name $rgname -Location $loc -Force;

### <a name="create-networking-vnet--subnet"></a>创建网络（VNET/子网）
#### <a name="subnet-specification"></a>子网规范

    $subnetName = 'websubnet'
      $subnet = New-AzureRmVirtualNetworkSubnetConfig -Name $subnetName -AddressPrefix "10.0.0.0/24";

#### <a name="vnet-specification"></a>VNET 规范

    $vnet = New-AzureRmVirtualNetwork -Force -Name ('vnet' + $rgname) -ResourceGroupName $rgname -Location $loc -AddressPrefix "10.0.0.0/16" -Subnet $subnet;
    $vnet = Get-AzureRmVirtualNetwork -Name ('vnet' + $rgname) -ResourceGroupName $rgname;

    # In this case assume the new subnet is the only one
    $subnetId = $vnet.Subnets[0].Id;

#### <a name="create-public-ip-resource-to-allow-external-access"></a>创建允许外部访问的公共 IP 资源
该资源将绑定到负载均衡器。

    $pubip = New-AzureRmPublicIpAddress -Force -Name ('pubip' + $rgname) -ResourceGroupName $rgname -Location $loc -AllocationMethod Dynamic -DomainNameLabel ('pubip' + $rgname);
    $pubip = Get-AzureRmPublicIpAddress -Name ('pubip' + $rgname) -ResourceGroupName $rgname;

#### <a name="create-load-balancer"></a>创建负载均衡器

    $frontendName = 'fe' + $rgname
    $backendAddressPoolName = 'bepool' + $rgname
    $probeName = 'vmssprobe' + $rgname
    $inboundNatPoolName = 'innatpool' + $rgname
    $lbruleName = 'lbrule' + $rgname
    $lbName = 'vmsslb' + $rgname

    # Bind Public IP to Load Balancer
    $frontend = New-AzureRmLoadBalancerFrontendIpConfig -Name $frontendName -PublicIpAddress $pubip

#### <a name="configure-load-balancer"></a>配置负载均衡器
创建后端地址池配置，规模集中 VM 的 NIC 共享该配置。

    $backendAddressPool = New-AzureRmLoadBalancerBackendAddressPoolConfig -Name $backendAddressPoolName

设置负载均衡的探测端口，并根据应用程序适当地更改设置。

    $probe = New-AzureRmLoadBalancerProbeConfig -Name $probeName -RequestPath healthcheck.aspx -Protocol http -Port 80 -IntervalInSeconds 15 -ProbeCount 2

创建入站 NAT 池，以便通过负载均衡器将连接直接路由到（非负载均衡）规模集中的 VM。 此操作使用 RDP 进行演示，你的应用程序可能不需要进行此操作。

    $frontendpoolrangestart = 3360
    $frontendpoolrangeend = 3370
    $backendvmport = 3389
    $inboundNatPool = New-AzureRmLoadBalancerInboundNatPoolConfig -Name $inboundNatPoolName -FrontendIPConfigurationId `
    $frontend.Id -Protocol Tcp -FrontendPortRangeStart $frontendpoolrangestart -FrontendPortRangeEnd $frontendpoolrangeend -BackendPort $backendvmport;

创建负载均衡规则。本示例演示如何使用上述步骤中的设置来平衡端口 80 请求的负载。

    $protocol = 'Tcp'
    $feLBPort = 80
    $beLBPort = 80

    $lbrule = New-AzureRmLoadBalancerRuleConfig -Name $lbruleName `
    -FrontendIPConfiguration $frontend -BackendAddressPool $backendAddressPool `
    -Probe $probe -Protocol $protocol -FrontendPort $feLBPort -BackendPort $beLBPort `
    -IdleTimeoutInMinutes 15 -EnableFloatingIP -LoadDistribution SourceIP -Verbose;

使用配置创建负载均衡器。

    $actualLb = New-AzureRmLoadBalancer -Name $lbName -ResourceGroupName $rgname -Location $loc `
    -FrontendIpConfiguration $frontend -BackendAddressPool $backendAddressPool `
    -Probe $probe -LoadBalancingRule $lbrule -InboundNatPool $inboundNatPool -Verbose;

检查 LB 设置，检查负载均衡端口配置。请注意，创建规模集中的 VM 之前，看不到入站 NAT 规则。

    $expectedLb = Get-AzureRmLoadBalancer -Name $lbName -ResourceGroupName $rgname

##### <a name="configure-and-create-the-scale-set"></a>配置和创建规模集
请注意，此基础结构示例演示如何在规模集中设置分发和调整网络流量，但此处指定的 VM 映像并未安装任何 Web 服务。

    # specify scale set Name
    $vmssName = 'vmss' + $rgname;

    ## specify VMSS specific details
    $adminUsername = 'azadmin';
    $adminPassword = "Password1234!";

    $PublisherName = 'MicrosoftWindowsServer'
    $Offer         = 'WindowsServer'
    $Sku          = '2012-R2-Datacenter'
    $Version       = 'latest'
    $vmNamePrefix = 'winvmss'

    ###add an extension
    $extname = 'BGInfo';
    $publisher = 'Microsoft.Compute';
    $exttype = 'BGInfo';
    $extver = '2.1';

将 NIC 绑定到负载均衡器和子网

    $ipCfg = New-AzureRmVmssIPConfig -Name 'nic' `
    -LoadBalancerInboundNatPoolsId $actualLb.InboundNatPools[0].Id `
    -LoadBalancerBackendAddressPoolsId $actualLb.BackendAddressPools[0].Id `
    -SubnetId $subnetId;

创建规模集配置

    # Specify number of nodes
    $numberofnodes = 3

    $vmss = New-AzureRmVmssConfig -Location $loc -SkuCapacity $numberofnodes -SkuName 'Standard_A2' -UpgradePolicyMode 'automatic' `
        | Add-AzureRmVmssNetworkInterfaceConfiguration -Name $subnetName -Primary $true -IPConfiguration $ipCfg `
        | Set-AzureRmVmssOSProfile -ComputerNamePrefix $vmNamePrefix -AdminUsername $adminUsername -AdminPassword $adminPassword `
        | Set-AzureRmVmssStorageProfile -OsDiskCreateOption 'FromImage' -OsDiskCaching 'None' `
        -ImageReferenceOffer $Offer -ImageReferenceSku $Sku -ImageReferenceVersion $Version `
        -ImageReferencePublisher $PublisherName `
        | Add-AzureRmVmssExtension -Name $extname -Publisher $publisher -Type $exttype -TypeHandlerVersion $extver -AutoUpgradeMinorVersion $true

生成规模集配置

    New-AzureRmVmss -ResourceGroupName $rgname -Name $vmssName -VirtualMachineScaleSet $vmss -Verbose;

现在你已创建规模集。 在本示例中，可以测试使用 RDP 连接到单个 VM：

    VM0 : pubipmynewrgwu.chinanorth.chinacloudapp.cn:3360
    VM1 : pubipmynewrgwu.chinanorth.chinacloudapp.cn:3361
    VM2 : pubipmynewrgwu.chinanorth.chinacloudapp.cn:3362
<!--Update_Description: wording update-->