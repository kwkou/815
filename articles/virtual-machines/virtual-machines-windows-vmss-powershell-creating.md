<properties
	pageTitle="使用 PowerShell cmdlet 创建虚拟机规模集 | Azure"
	description="开始使用 Azure PowerShell cmdlet 来创建和管理你的第一个 Azure 虚拟机规模集"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="danielsollondon"
	manager="guybo"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-windows"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/29/2016"
	wacn.date="11/21/2016"
	ms.author="danielsollondon"/>

# 使用 PowerShell cmdlet 创建虚拟机规模集

本示例演示如何创建虚拟机规模集 (VMSS)，其中将会创建具有 3 个节点的 VMSS，并包含所有关联的网络和存储。

## 前几个步骤
确保已安装最新的 Azure PowerShell 模块，其中包含维护和创建 VMSS 所需的 PowerShell cmdlet。
转到[此处](http://aka.ms/webpi-azps)的命令行工具，以获取最新可用的 Azure 模块。

若要查找 VMSS 相关的 cmdlet，请使用搜索字符串 VMSS。

## 创建 VMSS

##### 创建资源组

	$loc = 'chinanorth';
	$rgname = 'mynewrgwu';
	New-AzureRmResourceGroup -Name $rgname -Location $loc -Force;

##### 创建存储帐户

设置存储帐户类型/名称。

	$stoname = 'sto' + $rgname;
	$stotype = 'Standard_LRS';
	New-AzureRmStorageAccount -ResourceGroupName $rgname -Name $stoname -Location $loc -Type $stotype;

	$stoaccount = Get-AzureRmStorageAccount -ResourceGroupName $rgname -Name $stoname;

#### 创建网络（VNET/子网）

##### 子网规范

	$subnetName = 'websubnet'
	$subnet = New-AzureRmVirtualNetworkSubnetConfig -Name $subnetName -AddressPrefix "10.0.0.0/24";

##### VNET 规范

	$vnet = New-AzureRmVirtualNetwork -Force -Name ('vnet' + $rgname) -ResourceGroupName $rgname -Location $loc -AddressPrefix "10.0.0.0/16" -DnsServer "10.1.1.1" -Subnet $subnet;
	$vnet = Get-AzureRmVirtualNetwork -Name ('vnet' + $rgname) -ResourceGroupName $rgname;

	#In this case below we assume the new subnet is the only one, note difference if you have one already or have adjusted this code to more than one subnet.
	$subnetId = $vnet.Subnets[0].Id;

##### 创建允许外部访问的公共 IP 资源

这将绑定到负载均衡器。

	$pubip = New-AzureRmPublicIpAddress -Force -Name ('pubip' + $rgname) -ResourceGroupName $rgname -Location $loc -AllocationMethod Dynamic -DomainNameLabel ('pubip' + $rgname);
	$pubip = Get-AzureRmPublicIpAddress -Name ('pubip' + $rgname) -ResourceGroupName $rgname;

##### 创建并配置负载均衡器

	$frontendName = 'fe' + $rgname
	$backendAddressPoolName = 'bepool' + $rgname
	$probeName = 'vmssprobe' + $rgname
	$inboundNatPoolName = 'innatpool' + $rgname
	$lbruleName = 'lbrule' + $rgname
	$lbName = 'vmsslb' + $rgname

	#Bind Public IP to Load Balancer
	$frontend = New-AzureRmLoadBalancerFrontendIpConfig -Name $frontendName -PublicIpAddress $pubip

##### 配置负载均衡器
创建后端地址池配置，这将由 VMSS 中 VM 的 NIC 共享。

	$backendAddressPool = New-AzureRmLoadBalancerBackendAddressPoolConfig -Name $backendAddressPoolName

设置负载均衡的探测端口，并根据应用程序适当地更改设置。

	$probe = New-AzureRmLoadBalancerProbeConfig -Name $probeName -RequestPath healthcheck.aspx -Protocol http -Port 80 -IntervalInSeconds 15 -ProbeCount 2

创建 NAT 规则以通过负载均衡器直接路由（而不平衡负载）连接到 VMSS 中的 VM，请注意这只是演示 RDP 的用法，实际上应使用内部 VNET 方法通过 RDP 访问这些服务器。

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

检查 LB 设置，检查负载均衡端口配置，请注意，在 VMSS 中的 VM 创建之前，你将不看到入站 NAT 规则。

	$expectedLb = Get-AzureRmLoadBalancer -Name $lbName -ResourceGroupName $rgname

##### 配置和创建 VMSS

请注意，此基础结构示例演示如何在 VMSS 中设置分发和缩放网络流量，但此处指定的 VM 映像并未安装任何 Web 服务。

	#specify VMSS Name
	$vmssName = 'vmss' + $rgname;

	##specify VMSS specific details
	$adminUsername = 'azadmin';
	$adminPassword = "Password1234!";

	$PublisherName = 'MicrosoftWindowsServer'
	$Offer         = 'WindowsServer'
	$Sku          = '2012-R2-Datacenter'
	$Version       = 'latest'
	$vmNamePrefix = 'winvmss'

	$vhdContainer = "https://" + $stoname + ".blob.core.chinacloudapi.cn/" + $vmssName;

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

	$ipCfg.LoadBalancerBackendAddressPools.Add($actualLb.BackendAddressPools[0].Id);
	$ipCfg.LoadBalancerInboundNatPools.Add($actualLb.InboundNatPools[0].Id);

创建 VMSS 配置

	#Specify number of nodes
	$numberofnodes = 3

	$vmss = New-AzureRmVmssConfig -Location $loc -SkuCapacity $numberofnodes -SkuName 'Standard_A2' -UpgradePolicyMode 'automatic' `
		| Add-AzureRmVmssNetworkInterfaceConfiguration -Name $subnetName -Primary $true -IPConfiguration $ipCfg `
		| Set-AzureRmVmssOSProfile -ComputerNamePrefix $vmNamePrefix -AdminUsername $adminUsername -AdminPassword $adminPassword `
		| Set-AzureRmVmssStorageProfile -Name 'test' -OsDiskCreateOption 'FromImage' -OsDiskCaching 'None' `
		-ImageReferenceOffer $Offer -ImageReferenceSku $Sku -ImageReferenceVersion $Version `
		-ImageReferencePublisher $PublisherName -VhdContainer $vhdContainer `
		| Add-AzureRmVmssExtension -Name $extname -Publisher $publisher -Type $exttype -TypeHandlerVersion $extver -AutoUpgradeMinorVersion $true

生成 VMSS 配置

	New-AzureRmVmss -ResourceGroupName $rgname -Name $vmssName -VirtualMachineScaleSet $vmss -Verbose;

现在你已创建 VMSS。在本示例中，可以测试使用 RDP 连接到单个 VM：

	VM0 : pubipmynewrgwu.chinanorth.chinacloudapp.cn:3360
	VM1 : pubipmynewrgwu.chinanorth.chinacloudapp.cn:3361
	VM2 : pubipmynewrgwu.chinanorth.chinacloudapp.cn:3362

<!---HONumber=Mooncake_0425_2016-->