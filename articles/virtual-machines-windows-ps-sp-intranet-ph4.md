<properties
	pageTitle="SharePoint Server 2013 场（阶段 4）| Microsoft Azure"
	description="在 Azure 的 SharePoint Server 2013 场阶段 4 中创建 SharePoint Server 虚拟机和新的 SharePoint 场。"
	documentationCenter=""
	services="virtual-machines-windows"
	authors="JoeDavies-MSFT"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="12/11/2015"
	wacn.date=""/>

# SharePoint Intranet 场工作负荷阶段 4：配置 SharePoint Server

> [AZURE.NOTE]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model)。这篇文章介绍如何使用资源管理器部署模型，Microsoft 建议大多数新部署使用资源管理器模型替代经典部署模型。

在部署仅限 Intranet 的 SharePoint 2013 场（在 Azure 基础结构服务中通过 SQL Server AlwaysOn 可用性组进行）的这个阶段，你将构建出 SharePoint 场的应用程序层和 Web 层，并使用 SharePoint 配置向导创建场。

你必须在转到[阶段 5](/documentation/articles/virtual-machines-workload-intranet-sharepoint-phase5) 之前完成此阶段。请参阅[在 Azure 中通过 SQL Server AlwaysOn 可用性组部署 SharePoint](/documentation/articles/virtual-machines-workload-intranet-sharepoint-overview) 以了解所有阶段的相关信息。

## 在 Azure 中创建 SharePoint Server 虚拟机

有四个 SharePoint Server 虚拟机。两个 SharePoint Server 虚拟机用于前端 Web 服务器，另两个用于管理和托管 SharePoint 应用程序。每个层中的两个 SharePoint Server 可提供高可用性。

首先，将配置内部负载平衡，以便 Azure 将客户端流量均匀地分布在两个前端 web 服务器上。这需要你指定内部负载平衡实例，该实例由名称和它自己的 IP 地址（从你分配给 Azure 虚拟网络的子网地址空间中获取）组成。

> [AZURE.NOTE]以下命令集使用 Azure PowerShell 1.0 及更高版本。有关详细信息，请参阅 [Azure PowerShell 1.0](https://azure.microsoft.com/blog/azps-1-0/)。

为变量指定值，并删除 < and > 字符。请注意，以下 Azure PowerShell 命令集使用下表中的值：

- 表 M，用于虚拟机
- 表 V，用于虚拟网络设置
- 表 S，用于子网
- 表 ST，用于存储帐户
- 表 A，用于可用性集

回想一下，你在[阶段 2：配置域控制器](/documentation/articles/virtual-machines-workload-intranet-sharepoint-phase2)中定义了表 M，在[阶段 1：配置 Azure](/documentation/articles/virtual-machines-workload-intranet-sharepoint-phase1) 中定义了表 V、表 S、表 A 和表 C。

如果已提供所有适当的值，请在 Azure PowerShell 命令提示符下运行生成的块。

	# Set up key variables
	$rgName="<resource group name>"
	$locName="<Azure location of your resource group>"
	$vnetName="<Table V – Item 1 – Value column>"
	$privIP="<available IP address on the subnet>"
	$vnet=Get-AzureRMVirtualNetwork -Name $vnetName -ResourceGroupName $rgName

	$frontendIP=New-AzureRMLoadBalancerFrontendIpConfig -Name SharePointWebServers-LBFE -PrivateIPAddress $privIP -SubnetId $vnet.Subnets[1].Id
	$beAddressPool=New-AzureRMLoadBalancerBackendAddressPoolConfig -Name SharePointWebServers-LBBE

	# This example assumes unsecured (HTTP-based) web traffic to the web servers.
	$healthProbe=New-AzureRMLoadBalancerProbeConfig -Name WebServersProbe -Protocol "TCP" -Port 80 -IntervalInSeconds 15 -ProbeCount 2
	$lbrule=New-AzureRMLoadBalancerRuleConfig -Name "WebTraffic" -FrontendIpConfiguration $frontendIP -BackendAddressPool $beAddressPool -Probe $healthProbe -Protocol "TCP" -FrontendPort 80 -BackendPort 80
	New-AzureRMLoadBalancer -ResourceGroupName $rgName -Name "SharePointWebServersInAzure" -Location $locName -LoadBalancingRule $lbrule -BackendAddressPool $beAddressPool -Probe $healthProbe -FrontendIpConfiguration $frontendIP

接下来，将 DNS 地址记录添加到你所在组织的内部 DNS 基础结构中，以便将 SharePoint 场的完全限定域名（例如 spfarm.corp.contoso.com）解析为分配给内部负载平衡器的 IP 地址（前面的 Azure PowerShell 命令块中 $privIP 的值）。

使用以下 Azure PowerShell 命令块为四个 SharePoint Server 创建虚拟机。如果已提供所有适当的值，请在 Azure PowerShell 命令提示符下运行生成的块。

	# Set up key variables
	$rgName="<resource group name>"
	$locName="<Azure location of your resource group>"
	$saName="<Table ST – Item 2 – Storage account name column>"
	$vnetName="<Table V – Item 1 – Value column>"
	$avName="<Table A – Item 3 – Availability set name column>"
	
	# Create the first application server
	$vmName="<Table M – Item 6 - Virtual machine name column>"
	$vmSize="<Table M – Item 6 - Minimum size column>"
	$vnet=Get-AzureRMVirtualNetwork -Name $vnetName -ResourceGroupName $rgName
	$nic=New-AzureRMNetworkInterface -Name ($vmName +"-NIC") -ResourceGroupName $rgName -Location $locName -SubnetId $vnet.Subnets[1].Id
	$avSet=Get-AzureRMAvailabilitySet –Name $avName –ResourceGroupName $rgName 
	$vm=New-AzureRMVMConfig -VMName $vmName -VMSize $vmSize -AvailabilitySetId $avset.Id
	$cred=Get-Credential -Message "Type the name and password of the local administrator account for the first application server." 
	$vm=Set-AzureRMVMOperatingSystem -VM $vm -Windows -ComputerName $vmName -Credential $cred -ProvisionVMAgent -EnableAutoUpdate
	$vm=Set-AzureRMVMSourceImage -VM $vm -PublisherName MicrosoftSharePoint -Offer MicrosoftSharePointServer -Skus 2013 -Version "latest"
	$vm=Add-AzureRMVMNetworkInterface -VM $vm -Id $nic.Id
	$storageAcc=Get-AzureRMStorageAccount -ResourceGroupName $rgName -Name $saName
	$osDiskUri=$storageAcc.PrimaryEndpoints.Blob.ToString() + "vhds/" + $vmName + "-OSDisk.vhd"
	$vm=Set-AzureRMVMOSDisk -VM $vm -Name "OSDisk" -VhdUri $osDiskUri -CreateOption fromImage
	New-AzureRMVM -ResourceGroupName $rgName -Location $locName -VM $vm

	# Create the second application server
	$vmName="<Table M – Item 7 - Virtual machine name column>"
	$vmSize="<Table M – Item 7 - Minimum size column>"
	$vnet=Get-AzureRMVirtualNetwork -Name $vnetName -ResourceGroupName $rgName
	$nic=New-AzureRMNetworkInterface -Name ($vmName +"-NIC") -ResourceGroupName $rgName -Location $locName -SubnetId $vnet.Subnets[1].Id
	$avSet=Get-AzureRMAvailabilitySet –Name $avName –ResourceGroupName $rgName 
	$vm=New-AzureRMVMConfig -VMName $vmName -VMSize $vmSize -AvailabilitySetId $avset.Id
	$cred=Get-Credential -Message "Type the name and password of the local administrator account for the second application server." 
	$vm=Set-AzureRMVMOperatingSystem -VM $vm -Windows -ComputerName $vmName -Credential $cred -ProvisionVMAgent -EnableAutoUpdate
	$vm=Set-AzureRMVMSourceImage -VM $vm -PublisherName MicrosoftSharePoint -Offer MicrosoftSharePointServer -Skus 2013 -Version "latest"
	$vm=Add-AzureRMVMNetworkInterface -VM $vm -Id $nic.Id
	$storageAcc=Get-AzureRMStorageAccount -ResourceGroupName $rgName -Name $saName
	$osDiskUri=$storageAcc.PrimaryEndpoints.Blob.ToString() + "vhds/" + $vmName + "-OSDisk.vhd"
	$vm=Set-AzureRMVMOSDisk -VM $vm -Name "OSDisk" -VhdUri $osDiskUri -CreateOption fromImage
	New-AzureRMVM -ResourceGroupName $rgName -Location $locName -VM $vm

	# Change the availability set
	$avName="<Table A – Item 4 – Availability set name column>"

	# Set up key variables
	$beSubnetName="<Table S - Item 2 - Name column>"
	$webLB=Get-AzureRMLoadBalancer -ResourceGroupName $rgName -Name "SharePointWebServersInAzure"	
	$vnet=Get-AzureRMVirtualNetwork -Name $vnetName -ResourceGroupName $rgName
	$backendSubnet=Get-AzureRMVirtualNetworkSubnetConfig -Name $beSubnetName -VirtualNetwork $vnet
	
	# Create the first front end web server virtual machine
	$vmName="<Table M – Item 8 - Virtual machine name column>"
	$vmSize="<Table M – Item 8 - Minimum size column>"
	$nic=New-AzureRMNetworkInterface -Name ($vmName + "-NIC") -ResourceGroupName $rgName -Location $locName -Subnet $backendSubnet -LoadBalancerBackendAddressPool $webLB.BackendAddressPools[0]
	$avSet=Get-AzureRMAvailabilitySet -Name $avName –ResourceGroupName $rgName 
	$vm=New-AzureRMVMConfig -VMName $vmName -VMSize $vmSize -AvailabilitySetId $avset.Id
	$cred=Get-Credential -Message "Type the name and password of the local administrator account for the first front end web server." 
	$vm=Set-AzureRMVMOperatingSystem -VM $vm -Windows -ComputerName $vmName -Credential $cred -ProvisionVMAgent -EnableAutoUpdate
	$vm=Set-AzureRMVMSourceImage -VM $vm -PublisherName MicrosoftSharePoint -Offer MicrosoftSharePointServer -Skus 2013 -Version "latest"
	$vm=Add-AzureRMVMNetworkInterface -VM $vm -Id $nic.Id
	$storageAcc=Get-AzureRMStorageAccount -ResourceGroupName $rgName -Name $saName
	$osDiskUri=$storageAcc.PrimaryEndpoints.Blob.ToString() + "vhds/" + $vmName + "-OSDisk.vhd"
	$vm=Set-AzureRMVMOSDisk -VM $vm -Name "OSDisk" -VhdUri $osDiskUri -CreateOption fromImage
	New-AzureRMVM -ResourceGroupName $rgName -Location $locName -VM $vm
	
	# Create the second front end web server virtual machine
	$vmName="<Table M – Item 9 - Virtual machine name column>"
	$vmSize="<Table M – Item 9 - Minimum size column>"
	$nic=New-AzureRMNetworkInterface -Name ($vmName + "-NIC") -ResourceGroupName $rgName -Location $locName -Subnet $backendSubnet -LoadBalancerBackendAddressPool $webLB.BackendAddressPools[0]
	$vm=New-AzureRMVMConfig -VMName $vmName -VMSize $vmSize -AvailabilitySetId $avset.Id
	$cred=Get-Credential -Message "Type the name and password of the local administrator account for the second front end web server." 
	$vm=Set-AzureRMVMOperatingSystem -VM $vm -Windows -ComputerName $vmName -Credential $cred -ProvisionVMAgent -EnableAutoUpdate
	$vm=Set-AzureRMVMSourceImage -VM $vm -PublisherName MicrosoftSharePoint -Offer MicrosoftSharePointServer -Skus 2013 -Version "latest"
	$vm=Add-AzureRMVMNetworkInterface -VM $vm -Id $nic.Id
	$storageAcc=Get-AzureRMStorageAccount -ResourceGroupName $rgName -Name $saName
	$osDiskUri=$storageAcc.PrimaryEndpoints.Blob.ToString() + "vhds/" + $vmName + "-OSDisk.vhd"
	$vm=Set-AzureRMVMOSDisk -VM $vm -Name "OSDisk" -VhdUri $osDiskUri -CreateOption fromImage
	New-AzureRMVM -ResourceGroupName $rgName -Location $locName -VM $vm

> [AZURE.NOTE]由于这些虚拟机用于 Intranet 应用程序，因此未为它们分配公共 IP 地址或 DNS 域名标签，也未向 Internet 公开。但是，这也意味着你无法从 Azure 门户连接到它们。当你查看虚拟机的属性时，“连接”按钮将不可用。

使用所选的远程桌面客户端创建与每个虚拟机的远程桌面连接。使用其 Intranet DNS 或计算机名称和本地管理员帐户的凭据。

接下来，对于每个虚拟机，在 Windows PowerShell 提示符下使用这些命令将它们加入到相应的 Active Directory 域。

	$domName="<Active Directory domain name to join, such as corp.contoso.com>"
	Add-Computer -DomainName $domName
	Restart-Computer

请注意，必须在输入 **Add-Computer** 命令后提供域帐户凭据。

在它们重新启动后，利用[“使用远程桌面连接登录到虚拟机”过程](/documentation/articles/virtual-machines-workload-intranet-sharepoint-phase2#logon)四次（对每个 SharePoint Server 利用一次），使用 [Domain]\\sp\_farm\_db 帐户凭据登录。这些凭据是在[阶段 2：配置域控制器](virtual-machines-workload-intranet-sharepoint-phase2)中创建的。

使用[测试连接过程](/documentation/articles/virtual-machines-workload-intranet-sharepoint-phase2#testconn)四次（对每个 SharePoint Server 使用一次），测试与你的组织网络上的位置的连接。

> [AZURE.NOTE]SharePoint Server 是从 SharePoint Server 2013 试用版映像中创建而来。对于 SharePoint Server 2013 Standard 版或 Enterprise 版，你需要将安装转换为使用“零售”或“批量许可证”密钥。

## 配置 SharePoint 场

使用以下步骤来配置场中的第一个 SharePoint Server：

1.	从第一个 SharePoint 应用程序服务器的桌面上，双击**“SharePoint 2013 产品配置向导”**。当系统询问你是否允许程序对计算机进行更改时，请单击“是”。
2.	在“欢迎使用 SharePoint 产品”页上，单击“下一步”。
3.	此时将显示**“SharePoint 产品配置向导”**对话框，警告将重新启动或重置服务（如 IIS）。单击**“是”**。
4.	在“连接到服务器场”页上，选择“创建新的服务器场”，然后单击“下一步”。
5.	在“指定配置数据库设置”页上，执行以下操作：
 - 在“数据库服务器”中，键入主数据库服务器的名称。
 - 在“用户名”中，键入 [Domain]**\\sp\_farm\_db**（在[阶段 2：配置域控制器](/documentation/articles/virtual-machines-workload-intranet-sharepoint-phase2)中创建的）。前面曾提到 sp\_farm\_db 帐户对数据库服务器具有 sysadmin 权限。
 - 在**“密码”**中，键入 sp\_farm\_db 帐户密码。
6.	单击**“下一步”**。
7.	在“指定场安全设置”页上，键入通行短语两次。记录通行短语，并将其存储在安全位置以供将来参考。单击**“下一步”**。
8.	在“配置 SharePoint 管理中心 Web 应用程序”页上，单击“下一步”。
9.	此时将显示“完成 SharePoint 产品配置向导”页。单击**“下一步”**。
10.	此时将显示“配置 SharePoint 产品”页。等到配置过程完成，大约 8 分钟。
11.	成功配置场后，请单击**“完成”**。此时新的管理网站将启动。
12.	若要开始配置 SharePoint 场，请单击**“启动向导”**。

在第二个 SharePoint 应用程序服务器和两个前端 Web 服务器上执行以下过程：

1.	在桌面上，双击**“SharePoint 2013 产品配置向导”**。当系统询问你是否允许程序对计算机进行更改时，请单击“是”。
2.	在“欢迎使用 SharePoint 产品”页上，单击“下一步”。
3.	此时将显示**“SharePoint 产品配置向导”**对话框，警告将重新启动或重置服务（如 IIS）。单击**“是”**。
4.	在“连接到服务器场”页上，单击“连接到现有服务器场”，然后单击“下一步”。
5.	在“指定配置数据库设置”页上，在“数据库服务器”中键入主数据库服务器的名称，然后单击“检索数据库名称”。
6.	在数据库名称列表中单击“SharePoint\_Config”，然后单击“下一步”。
7.	在“指定场安全设置”页上，键入上一个过程中提供的通行短语。单击**“下一步”**。
8.	此时将显示“完成 SharePoint 产品配置向导”页。单击**“下一步”**。
9.	在“配置成功”页上，单击“完成”。

当 SharePoint 创建场时，它将在主 SQL Server 虚拟机上配置一组服务器登录名。SQL Server 2012 引入了具有包含数据库的密码的用户概念。数据库本身存储所有数据库元数据和用户信息，而在此数据库中定义的用户不需要具有相应的登录名。此数据库中的信息由可用性组复制，并在故障转移之后可用。有关详细信息，请参阅[包含数据库](http://go.microsoft.com/fwlink/p/?LinkId=262794)。

但是，默认情况下 SharePoint 数据库不是包含数据库。因此，你将需要手动配置辅助数据库服务器，使其与主数据库服务器具有一组相同的 SharePoint 场帐户登录名。可以通过从 SQL Server Management Studio 中同时连接到这两个服务器来执行此同步。

在完成此初始设置后，针对 SharePoint 场功能的更多配置选项将可用。有关详细信息，请参阅[在 Azure 基础结构服务上规划 SharePoint 2013](http://msdn.microsoft.com/zh-cn/library/dn275958.aspx)。

这是成功完成此阶段后生成的配置。

![](./media/virtual-machines-workload-intranet-sharepoint-phase4/workload-spsqlao_04.png)

## 后续步骤

- 若要继续配置此工作负荷，请使用[阶段 5](/documentation/articles/virtual-machines-workload-intranet-sharepoint-phase5)。

<!---HONumber=Mooncake_0104_2016-->