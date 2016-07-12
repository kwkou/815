<!-- not suitable for Mooncake -->

<properties 
	pageTitle="业务线应用程序（阶段 4）| Azure" 
	description="在 Azure 的业务线应用程序阶段 4 中创建 Web 服务器并在其上加载业务线应用程序。" 
	documentationCenter=""
	services="virtual-machines-windows" 
	authors="JoeDavies-MSFT" 
	manager="timlt" 
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="04/01/2016"
	wacn.date="06/29/2016"/>

# 业务线应用程序工作负荷阶段 4：配置 web 服务器

> [AZURE.NOTE]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。这篇文章介绍如何使用资源管理器部署模型，Azure 建议大多数新部署使用资源管理器模型替代经典部署模型。
 

在 Azure 基础结构服务中部署高可用性业务线应用程序的这个阶段，你将构建 web 服务器并在其上加载业务线应用程序。

你必须在转到[阶段 5](/documentation/articles/virtual-machines-windows-ps-lob-ph5/) 之前完成此阶段。有关所有阶段，请参阅[在 Azure 中部署高可用性业务线应用程序](/documentation/articles/virtual-machines-windows-lob-overview/)。

## 在 Azure 中创建 web 服务器虚拟机

有两个 web 服务器虚拟机，可以在其上部署 ASP.NET 应用程序或可由 Windows Server 2012 R2 中的 Internet 信息服务 (IIS) 8 托管的旧版应用程序。

首先，将配置内部负载平衡，以便 Azure 将客户端发到业务线应用程序的流量均匀地分布在两个 web 服务器上。这需要你指定内部负载平衡实例，该实例由名称和它自己的 IP 地址（从你分配给 Azure 虚拟网络的子网地址空间中分配）组成。

> [AZURE.NOTE] 以下命令集使用 Azure PowerShell 1.0 及更高版本。有关详细信息，请参阅 [Azure PowerShell 1.0](https://azure.microsoft.com/blog/azps-1-0/)。

为变量指定值，并删除 < and > 字符。请注意，以下 Azure PowerShell 命令集使用下表中的值：

- 表 M，用于虚拟机
- 表 V，用于虚拟网络设置
- 表 S，用于子网
- 表 ST，用于存储帐户
- 表 A，用于可用性集

回想一下，你在[阶段 2](/documentation/articles/virtual-machines-windows-ps-lob-ph2/) 中定义了表 M，并在[阶段 1](/documentation/articles/virtual-machines-windows-ps-lob-ph1/) 中定义了表 V、表 S、表 ST 和表 A。

如果已提供所有适当的值，请在 Azure PowerShell 命令提示符下运行生成的块。

	# Set up key variables
	$rgName="<resource group name>"
	$locName="<Azure location of your resource group>"
	$vnetName="<Table V - Item 1 - Value column>"
	$privIP="<available IP address on the subnet>"
	$vnet=Get-AzureRMVirtualNetwork -Name $vnetName -ResourceGroupName $rgName

	$frontendIP=New-AzureRMLoadBalancerFrontendIpConfig -Name WebServers-LBFE -PrivateIPAddress $privIP -SubnetId $vnet.Subnets[1].Id
	$beAddressPool=New-AzureRMLoadBalancerBackendAddressPoolConfig -Name WebServers-LBBE

	# This example assumes unsecured (HTTP-based) web traffic to the web servers.
	$healthProbe=New-AzureRMLoadBalancerProbeConfig -Name WebServersProbe -Protocol "TCP" -Port 80 -IntervalInSeconds 15 -ProbeCount 2
	$lbrule=New-AzureRMLoadBalancerRuleConfig -Name "WebTraffic" -FrontendIpConfiguration $frontendIP -BackendAddressPool $beAddressPool -Probe $healthProbe -Protocol "TCP" -FrontendPort 80 -BackendPort 80
	New-AzureRMLoadBalancer -ResourceGroupName $rgName -Name "WebServersInAzure" -Location $locName -LoadBalancingRule $lbrule -BackendAddressPool $beAddressPool -Probe $healthProbe -FrontendIpConfiguration $frontendIP

接下来，将 DNS 地址记录添加到你组织的内部 DNS 基础结构中，以便将业务线应用程序的完全限定域名（例如 lobapp.corp.contoso.com）解析为分配给内部负载平衡器的 IP 地址（前面的 Azure PowerShell 命令块中 $privIP 的值）。

接下来，使用以下 PowerShell 命令块为两个 web 服务器创建虚拟机。

如果已提供所有适当的值，请在 Azure PowerShell 提示符下运行生成的块。

	# Set up key variables
	$rgName="<resource group name>"
	$locName="<Azure location of your resource group>"
	$webLB=Get-AzureRMLoadBalancer -ResourceGroupName $rgName -Name "WebServersInAzure"	
	
	# Use the standard storage account
	$saName="<Table ST - Item 2 - Storage account name column>"

	$vnetName="<Table V - Item 1 - Value column>"
	$beSubnetName="<Table S - Item 2 - Name column>"
	$avName="<Table A - Item 3 - Availability set name column>"
	$vnet=Get-AzureRMVirtualNetwork -Name $vnetName -ResourceGroupName $rgName
	$backendSubnet=Get-AzureRMVirtualNetworkSubnetConfig -Name $beSubnetName -VirtualNetwork $vnet
	
	# Create the first web server virtual machine
	$vmName="<Table M - Item 6 - Virtual machine name column>"
	$vmSize="<Table M - Item 6 - Minimum size column>"
	$nic=New-AzureRMNetworkInterface -Name ($vmName + "-NIC") -ResourceGroupName $rgName -Location $locName -Subnet $backendSubnet -LoadBalancerBackendAddressPool $webLB.BackendAddressPools[0]
	$avSet=Get-AzureRMAvailabilitySet -Name $avName -ResourceGroupName $rgName 
	$vm=New-AzureRMVMConfig -VMName $vmName -VMSize $vmSize -AvailabilitySetId $avset.Id
	$cred=Get-Credential -Message "Type the name and password of the local administrator account for the first web server." 
	$vm=Set-AzureRMVMOperatingSystem -VM $vm -Windows -ComputerName $vmName -Credential $cred -ProvisionVMAgent -EnableAutoUpdate
	$vm=Set-AzureRMVMSourceImage -VM $vm -PublisherName MicrosoftWindowsServer -Offer WindowsServer -Skus 2012-R2-Datacenter -Version "latest"
	$vm=Add-AzureRMVMNetworkInterface -VM $vm -Id $nic.Id
	$storageAcc=Get-AzureRMStorageAccount -ResourceGroupName $rgName -Name $saName
	$osDiskUri=$storageAcc.PrimaryEndpoints.Blob.ToString() + "vhds/" + $vmName + "-OSDisk.vhd"
	$vm=Set-AzureRMVMOSDisk -VM $vm -Name "OSDisk" -VhdUri $osDiskUri -CreateOption fromImage
	New-AzureRMVM -ResourceGroupName $rgName -Location $locName -VM $vm
	
	# Create the second web server virtual machine
	$vmName="<Table M - Item 7 - Virtual machine name column>"
	$vmSize="<Table M - Item 7 - Minimum size column>"
	$nic=New-AzureRMNetworkInterface -Name ($vmName + "-NIC") -ResourceGroupName $rgName -Location $locName -Subnet $backendSubnet -LoadBalancerBackendAddressPool $webLB.BackendAddressPools[0]
	$vm=New-AzureRMVMConfig -VMName $vmName -VMSize $vmSize -AvailabilitySetId $avset.Id
	$cred=Get-Credential -Message "Type the name and password of the local administrator account for the second SQL Server computer." 
	$vm=Set-AzureRMVMOperatingSystem -VM $vm -Windows -ComputerName $vmName -Credential $cred -ProvisionVMAgent -EnableAutoUpdate
	$vm=Set-AzureRMVMSourceImage -VM $vm -PublisherName MicrosoftWindowsServer -Offer WindowsServer -Skus 2012-R2-Datacenter -Version "latest"
	$vm=Add-AzureRMVMNetworkInterface -VM $vm -Id $nic.Id
	$storageAcc=Get-AzureRMStorageAccount -ResourceGroupName $rgName -Name $saName
	$osDiskUri=$storageAcc.PrimaryEndpoints.Blob.ToString() + "vhds/" + $vmName + "-OSDisk.vhd"
	$vm=Set-AzureRMVMOSDisk -VM $vm -Name "OSDisk" -VhdUri $osDiskUri -CreateOption fromImage
	New-AzureRMVM -ResourceGroupName $rgName -Location $locName -VM $vm

> [AZURE.NOTE] 由于这些虚拟机用于 Intranet 应用程序，因此未为它们分配公共 IP 地址或 DNS 域名标签，也未向 Internet 公开。但是，这也意味着你无法从 Azure 门户预览连接到它们。当你查看虚拟机的属性时，“连接”按钮将不可用。

使用所选的远程桌面客户端创建与每个 Web 服务器虚拟机的远程桌面连接。使用其 Intranet DNS 或计算机名称和本地管理员帐户的凭据。

接下来，对于每个 Web 服务器虚拟机，在 Windows PowerShell 提示符下使用这些命令将它们加入到相应的 Active Directory 域。

	$domName="<Active Directory domain name to join, such as corp.contoso.com>"
	Add-Computer -DomainName $domName
	Restart-Computer

请注意，必须在输入 “Add-Computer” 命令后提供域帐户凭据。

在这些虚拟机重新启动后，使用具有本地管理员权限的帐户重新连接到它们。

接下来，对于每个 web 服务器，安装并配置 IIS。

1. 运行服务器管理器，然后单击“添加角色和功能”。
2. 在“开始之前”页上，单击“下一步”。
3. 在“选择安装类型”页上，单击“下一步”。
4. 在“选择目标服务器”页上，单击“下一步”。
5. 在“服务器角色”页上，单击“角色”列表中的“Web 服务器(IIS)”。
6. 出现提示时，单击“添加功能”，然后单击“下一步”。
7. 在“选择功能”页上，单击“下一步”。
8. 在“Web 服务器(IIS)”页上，单击“下一步”。
9. 在“选择角色服务”页上，选中或清除 LOB 应用程序所需的服务的复选框，然后单击“下一步”。10. 在“确认安装选择”页上，单击“安装”。

## 将业务线应用程序部署到 Web 服务器虚拟机上

从本地网络中的计算机：

1.	将业务线应用程序的文件添加到两个 web 服务器上。
2.	在 SQL Server 群集上为业务线应用程序创建数据库。
3.	测试对业务线应用程序的访问及其功能。

下图是成功完成此阶段后生成的配置。

![](./media/virtual-machines-windows-ps-lob-ph4/workload-lobapp-phase4.png)

## 后续步骤

- 使用[阶段 5](/documentation/articles/virtual-machines-windows-ps-lob-ph5/) 完成此工作负荷的配置。

<!---HONumber=Mooncake_0411_2016-->