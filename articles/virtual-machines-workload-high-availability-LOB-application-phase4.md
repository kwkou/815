<properties 
	pageTitle="业务线应用程序工作负荷阶段 4：配置 web 服务器" 
	description="在 Azure 基础结构服务中部署高可用性业务线应用程序的这个第四阶段，你将创建 web 服务器并在其上加载业务线应用程序。" 
	documentationCenter=""
	services="virtual-machines" 
	authors="JoeDavies-MSFT" 
	manager="timlt" 
	editor=""
	tags="azure-resource-manager"/>

<tags 
	ms.service="virtual-machines" 
	ms.date="08/11/2015" 
	wacn.date="09/15/2015"/>

# 业务线应用程序工作负荷阶段 4：配置 web 服务器

在 Azure 基础结构服务中部署高可用性业务线应用程序的这个阶段，你将构建 web 服务器并在其上加载业务线应用程序。

你必须在转到[阶段 5](/documentation/articles/virtual-machines-workload-high-availability-LOB-application-phase5) 之前完成此阶段。有关所有阶段，请参阅[在 Azure 中部署高可用性业务线应用程序](/documentation/articles/virtual-machines-workload-high-availability-LOB-application-overview)。

## 在 Azure 中创建 web 服务器虚拟机

有两个 web 服务器虚拟机，可以在其上部署 ASP.NET 应用程序或可由 Windows Server 2012 R2 中的 Internet 信息服务 (IIS) 8 托管的旧版应用程序。

首先，将配置内部负载平衡，以便 Azure 将客户端发到业务线应用程序的流量均匀地分布在两个 web 服务器上。这需要你指定内部负载平衡实例，该实例由名称和它自己的 IP 地址（从你分配给 Azure 虚拟网络的子网地址空间中分配）组成。若要确定为内部负载平衡器选择的 IP 地址是否可用，请在 Azure PowerShell 提示符下使用这些命令。为变量指定值，并删除 < and > 字符。

	Switch-AzureMode AzureServiceManagement
	$vnet="<Table V – Item 1 – Value column>"
	$testIP="<a chosen IP address from the subnet address space, Table S - Item 2 – Subnet address space column>"
	Test-AzureStaticVNetIP –VNetName $vnet –IPAddress $testIP

如果在显示的 Test-AzureStaticVNetIP 命令中 **IsAvailable** 字段是 **True**，则可以使用该 IP 地址。

使用此命令切换回 PowerShell 的资源管理器模式。

	Switch-AzureMode AzureResourceManager

接下来，填写变量并运行以下命令集：

	$rgName="<resource group name>"
	$locName="<Azure location of your resource group>"
	$vnetName="<Table V – Item 1 – Value column>"
	$privIP="<available IP address on the subnet>"
	$vnet=Get-AzureVirtualNetwork -Name $vnetName -ResourceGroupName $rgName

	$frontendIP=New-AzureLoadBalancerFrontendIpConfig -Name WebServers-LBFE -PrivateIPAddress $privIP -SubnetId $vnet.Subnets[1].Id
	$beAddressPool=New-AzureLoadBalancerBackendAddressPoolConfig -Name WebServers-LBBE

	# This example assumes unsecured (HTTP-based) web traffic to the web servers.
	$healthProbe=New-AzureLoadBalancerProbeConfig -Name WebServersProbe -Protocol "TCP" -Port 80 -IntervalInSeconds 15 -ProbeCount 2
	$lbrule=New-AzureLoadBalancerRuleConfig -Name "WebTraffic" -FrontendIpConfiguration $frontendIP -BackendAddressPool $beAddressPool -Probe $healthProbe -Protocol "TCP" -FrontendPort 80 -BackendPort 80
	New-AzureLoadBalancer -ResourceGroupName $rgName -Name "WebServersInAzure" -Location $locName -LoadBalancingRule $lbrule -BackendAddressPool $beAddressPool -Probe $healthProbe -FrontendIpConfiguration $frontendIP

接下来，将 DNS 地址记录添加到你组织的内部 DNS 基础结构中，以便将业务线应用程序的完全限定域名（例如 lobapp.corp.contoso.com）解析为分配给内部负载平衡器的 IP 地址（前面的 Azure PowerShell 命令块中 $privIP 的值）。

接下来，使用以下 PowerShell 命令块为两个 web 服务器创建虚拟机。请注意，此 PowerShell 命令集使用下表中的值：

- 表 M，用于虚拟机
- 表 V，用于虚拟网络设置
- 表 S，用于子网
- 表 ST，用于存储帐户
- 表 A，用于可用性集

回想一下，你在[阶段 2](/documentation/articles/virtual-machines-workload-high-availability-LOB-application-phase2) 中定义了表 M，并在[阶段 1](/documentation/articles/virtual-machines-workload-high-availability-LOB-application-phase1) 中定义了表 V、表 S、表 ST 和表 A。

如果已提供所有适当的值，请在 Azure PowerShell 提示符下运行生成的块。

	# Set up subscription and key variables
	$subscr="<name of the Azure subscription>"
	Set-AzureSubscription -Environment AzureChinaCloud -SubscriptionName $subscr
	Switch-AzureMode AzureResourceManager
	$rgName="<resource group name>"
	$locName="<Azure location of your resource group>"
	$webLB=Get-AzureLoadBalancer -ResourceGroupName $rgName -Name "WebServersInAzure"	
	
	# Use the standard storage account
	$saName="<Table ST – Item 2 – Storage account name column>"

	$vnetName="<Table V – Item 1 – Value column>"
	$beSubnetName="<Table S - Item 2 - Name column>"
	$avName="<Table A – Item 3 – Availability set name column>"
	$vnet=Get-AzurevirtualNetwork -Name $vnetName -ResourceGroupName $rgName
	$backendSubnet=Get-AzureVirtualNetworkSubnetConfig -Name $beSubnetName -VirtualNetwork $vnet
	
	# Create the first web server virtual machine
	$vmName="<Table M – Item 6 - Virtual machine name column>"
	$vmSize="<Table M – Item 6 - Minimum size column>"
	$nic=New-AzureNetworkInterface -Name ($vmName + "-NIC") -ResourceGroupName $rgName -Location $locName -Subnet $backendSubnet -LoadBalancerBackendAddressPool $webLB.BackendAddressPools[0]
	$avSet=Get-AzureAvailabilitySet -Name $avName –ResourceGroupName $rgName 
	$vm=New-AzureVMConfig -VMName $vmName -VMSize $vmSize -AvailabilitySetId $avset.Id
	$cred=Get-Credential -Message "Type the name and password of the local administrator account for the first web server." 
	$vm=Set-AzureVMOperatingSystem -VM $vm -Windows -ComputerName $vmName -Credential $cred -ProvisionVMAgent -EnableAutoUpdate
	$vm=Set-AzureVMSourceImage -VM $vm -PublisherName MicrosoftWindowsServer -Offer WindowsServer -Skus 2012-R2-Datacenter -Version "latest"
	$vm=Add-AzureVMNetworkInterface -VM $vm -Id $nic.Id
	$storageAcc=Get-AzureStorageAccount -ResourceGroupName $rgName -Name $saName
	$osDiskUri=$storageAcc.PrimaryEndpoints.Blob.ToString() + "vhds/" + $vmName + "-OSDisk.vhd"
	$vm=Set-AzureVMOSDisk -VM $vm -Name "OSDisk" -VhdUri $osDiskUri -CreateOption fromImage
	New-AzureVM -ResourceGroupName $rgName -Location $locName -VM $vm
	
	# Create the second web server virtual machine
	$vmName="<Table M – Item 7 - Virtual machine name column>"
	$vmSize="<Table M – Item 7 - Minimum size column>"
	$nic=New-AzureNetworkInterface -Name ($vmName + "-NIC") -ResourceGroupName $rgName -Location $locName -Subnet $backendSubnet -LoadBalancerBackendAddressPool $webLB.BackendAddressPools[0]
	$vm=New-AzureVMConfig -VMName $vmName -VMSize $vmSize -AvailabilitySetId $avset.Id
	$cred=Get-Credential -Message "Type the name and password of the local administrator account for the second second SQL Server computer." 
	$vm=Set-AzureVMOperatingSystem -VM $vm -Windows -ComputerName $vmName -Credential $cred -ProvisionVMAgent -EnableAutoUpdate
	$vm=Set-AzureVMSourceImage -VM $vm -PublisherName MicrosoftWindowsServer -Offer WindowsServer -Skus 2012-R2-Datacenter -Version "latest"
	$vm=Add-AzureVMNetworkInterface -VM $vm -Id $nic.Id
	$storageAcc=Get-AzureStorageAccount -ResourceGroupName $rgName -Name $saName
	$osDiskUri=$storageAcc.PrimaryEndpoints.Blob.ToString() + "vhds/" + $vmName + "-OSDisk.vhd"
	$vm=Set-AzureVMOSDisk -VM $vm -Name "OSDisk" -VhdUri $osDiskUri -CreateOption fromImage
	New-AzureVM -ResourceGroupName $rgName -Location $locName -VM $vm

使用所选的远程桌面客户端创建与每个 Web 服务器虚拟机的远程桌面连接。使用其 Intranet DNS 或计算机名称和本地管理员帐户的凭据。

接下来，对于每个 Web 服务器虚拟机，在 Windows PowerShell 提示符下使用这些命令将它们加入到相应的 Active Directory 域。

	$domName="<Active Directory domain name to join, such as corp.contoso.com>"
	Add-Computer -DomainName $domName
	Restart-Computer

请注意，必须在输入 **Add-Computer** 命令后提供域帐户凭据。

在这些虚拟机重新启动后，使用具有本地管理员权限的帐户重新连接到它们。

接下来，对于每个 web 服务器，安装并配置 IIS。

1. 运行服务器管理器，然后单击**“添加角色和功能”**。
2. 在“开始之前”页上，单击**“下一步”**。
3. 在“选择安装类型”页上，单击**“下一步”**。
4. 在“选择目标服务器”页上，单击**“下一步”**。
5. 在“服务器角色”页上，单击**“角色”**列表中的**“Web 服务器(IIS)”**。
6. 出现提示时，单击**“添加功能”**，然后单击**“下一步”**。
7. 在“选择功能”页上，单击**“下一步”**。
8. 在“Web 服务器(IIS)”页上，单击**“下一步”**。
9. 在“选择角色服务”页上，选中或清除 LOB 应用程序所需的服务的复选框，然后单击**“下一步”**。10. 在“确认安装选择”页上，单击**“安装”**。

## 将业务线应用程序部署到 Web 服务器虚拟机上

从本地网络中的计算机：

1.	将业务线应用程序的文件添加到两个 web 服务器上。
2.	在 SQL Server 群集上为业务线应用程序创建数据库。
3.	测试对业务线应用程序的访问及其功能。

下图是成功完成此阶段后生成的配置。

![](./media/virtual-machines-workload-high-availability-LOB-application-phase4/workload-lobapp-phase4.png)

## 后续步骤

若要继续配置此工作负荷，请转到[阶段 5：创建可用性组并添加应用程序数据库](/documentation/articles/virtual-machines-workload-high-availability-LOB-application-phase5)。

## 其他资源

[在 Azure 中部署高可用性业务线应用程序](/documentation/articles/virtual-machines-workload-high-availability-LOB-application-overview)

[业务线应用程序体系结构蓝图](http://msdn.microsoft.com/dn630664)

[在混合云中设置用于测试且基于 Web 的 LOB 应用程序](/documentation/articles/virtual-networks-setup-lobapp-hybrid-cloud-testing)

[Azure 基础结构服务实施准则](/documentation/articles/virtual-machines-infrastructure-services-implementation-guidelines)

[Azure 基础结构服务工作负荷：SharePoint Server 2013 场](/documentation/articles/virtual-machines-workload-intranet-sharepoint-farm)

<!---HONumber=69-->