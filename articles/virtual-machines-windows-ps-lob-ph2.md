<!-- not suitable for Mooncake -->

<properties 
	pageTitle="业务线应用程序（阶段 2）| Azure" 
	description="在 Azure 的业务线应用程序的阶段 2 中创建并配置两个 Active Directory 副本域控制器。" 
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

# 业务线应用程序工作负荷阶段 2：配置域控制器

> [AZURE.NOTE]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。这篇文章介绍如何使用资源管理器部署模型，Azure 建议大多数新部署使用资源管理器模型替代经典部署模型。
 

在 Azure 基础结构服务中部署高可用性组业务线应用程序的这个阶段，你将在 Azure 虚拟网络中配置两个副本域控制器，以便可以在 Azure 虚拟网络中对针对 Web 资源的客户端 Web 请求进行身份验证，而不必将身份验证流量通过连接发送到你的本地网络。

你必须在转到[阶段 3](/documentation/articles/virtual-machines-windows-ps-lob-ph3/) 之前完成此阶段。有关所有阶段，请参阅[在 Azure 中部署高可用性业务线应用程序](/documentation/articles/virtual-machines-windows-lob-overview/)。

## 在 Azure 中创建域控制器虚拟机

首先，你需要填写表 M 的**“虚拟机名称”**列，并根据需要在**“最小大小”**列中修改虚拟机大小。

项目 | 虚拟机名称 | 库映像 | 最小大小 
--- | --- | --- | --- 
1\. | \_\_\_\_\_\_\_\_\_\_\_\_\_\_（第一个域控制器，示例 DC1） | Windows Server 2012 R2 Datacenter | Standard\_D2
2\. | \_\_\_\_\_\_\_\_\_\_\_\_\_\_（第二个域控制器，示例 DC2） | Windows Server 2012 R2 Datacenter | Standard\_D2
3\. | \_\_\_\_\_\_\_\_\_\_\_\_\_\_（主数据库服务器，示例 SQL1） | Microsoft SQL Server 2014 Enterprise - Windows Server 2012 R2 | 	Standard\_DS4
4\. | \_\_\_\_\_\_\_\_\_\_\_\_\_\_（辅助数据库服务器，示例 SQL2） | Microsoft SQL Server 2014 Enterprise - Windows Server 2012 R2 | 	Standard\_DS4
5\. | \_\_\_\_\_\_\_\_\_\_\_\_\_\_（群集的多数节点，示例 MN1） | Windows Server 2012 R2 Datacenter | Standard\_D1
6\. | \_\_\_\_\_\_\_\_\_\_\_\_\_\_（第一个 Web 服务器，示例 WEB1） | Windows Server 2012 R2 Datacenter | Standard\_D3
7\. | \_\_\_\_\_\_\_\_\_\_\_\_\_\_（第二个 Web 服务器，示例 WEB2） | Windows Server 2012 R2 Datacenter | Standard\_D3

**表 M - Azure 中高可用性业务线应用程序的虚拟机**

有关虚拟机大小的完整列表，请参阅[虚拟机的大小](/documentation/articles/virtual-machines-windows-sizes/)。

使用以下 Azure PowerShell 命令块为两个域控制器创建虚拟机。为变量指定值，并删除 < and > 字符。请注意，此 PowerShell 命令块使用下表中的值：

- 表 M，用于虚拟机
- 表 V，用于虚拟网络设置
- 表 S，用于子网
- 表 ST，用于存储帐户
- 表 A，用于可用性集

回想一下，你在[阶段 1：配置 Azure](/documentation/articles/virtual-machines-windows-ps-lob-ph1/) 中定义了表 V、表 S、表 ST 和表 A。

> [AZURE.NOTE] 以下命令集使用 Azure PowerShell 1.0 及更高版本。有关详细信息，请参阅 [Azure PowerShell 1.0](https://azure.microsoft.com/blog/azps-1-0/)。

如果已提供所有适当的值，请在 Azure PowerShell 提示符下运行生成的块。

	# Set up key variables
	$rgName="<resource group name>"
	$locName="<Azure location of your resource group>"
	$saName="<Table ST - Item 2 - Storage account name column>"
	$vnetName="<Table V - Item 1 - Value column>"
	$avName="<Table A - Item 1 - Availability set name column>"
	
	# Create the first domain controller
	$vmName="<Table M - Item 1 - Virtual machine name column>"
	$vmSize="<Table M - Item 1 - Minimum size column>"
	$staticIP="<Table V - Item 6 - Value column>"
	$vnet=Get-AzureRMVirtualNetwork -Name $vnetName -ResourceGroupName $rgName
	$nic=New-AzureRMNetworkInterface -Name ($vmName +"-NIC") -ResourceGroupName $rgName -Location $locName -SubnetId $vnet.Subnets[1].Id -PrivateIpAddress $staticIP
	$avSet=Get-AzureRMAvailabilitySet -Name $avName -ResourceGroupName $rgName 
	$vm=New-AzureRMVMConfig -VMName $vmName -VMSize $vmSize -AvailabilitySetId $avset.Id
	
	$diskSize=<size of the extra disk for AD DS data in GB>
	$storageAcc=Get-AzureRMStorageAccount -ResourceGroupName $rgName -Name $saName
	$vhdURI=$storageAcc.PrimaryEndpoints.Blob.ToString() + "vhds/" + $vmName + "-ADDSDisk.vhd"
	Add-AzureRMVMDataDisk -VM $vm -Name "ADDSData" -DiskSizeInGB $diskSize -VhdUri $vhdURI  -CreateOption empty
	
	$cred=Get-Credential -Message "Type the name and password of the local administrator account for the first domain controller." 
	$vm=Set-AzureRMVMOperatingSystem -VM $vm -Windows -ComputerName $vmName -Credential $cred -ProvisionVMAgent -EnableAutoUpdate
	$vm=Set-AzureRMVMSourceImage -VM $vm -PublisherName MicrosoftWindowsServer -Offer WindowsServer -Skus 2012-R2-Datacenter -Version "latest"
	$vm=Add-AzureRMVMNetworkInterface -VM $vm -Id $nic.Id
	$storageAcc=Get-AzureRMStorageAccount -ResourceGroupName $rgName -Name $saName
	$osDiskUri=$storageAcc.PrimaryEndpoints.Blob.ToString() + "vhds/" + $vmName + "-OSDisk.vhd"
	$vm=Set-AzureRMVMOSDisk -VM $vm -Name "OSDisk" -VhdUri $osDiskUri -CreateOption fromImage
	New-AzureRMVM -ResourceGroupName $rgName -Location $locName -VM $vm
	
	# Create the second domain controller
	$vmName="<Table M - Item 2 - Virtual machine name column>"
	$vmSize="<Table M - Item 2 - Minimum size column>"
	$staticIP="<Table V - Item 7 - Value column>"
	$vnet=Get-AzureRMVirtualNetwork -Name $vnetName -ResourceGroupName $rgName
	$nic=New-AzureRMNetworkInterface -Name ($vmName +"-NIC") -ResourceGroupName $rgName -Location $locName -SubnetId $vnet.Subnets[1].Id -PrivateIpAddress $staticIP
	$avSet=Get-AzureRMAvailabilitySet -Name $avName -ResourceGroupName $rgName 
	$vm=New-AzureRMVMConfig -VMName $vmName -VMSize $vmSize -AvailabilitySetId $avset.Id
	
	$diskSize=<size of the extra disk for AD DS data in GB>
	$storageAcc=Get-AzureRMStorageAccount -ResourceGroupName $rgName -Name $saName
	$vhdURI=$storageAcc.PrimaryEndpoints.Blob.ToString() + "vhds/" + $vmName + "-ADDSDisk.vhd"
	Add-AzureRMVMDataDisk -VM $vm -Name "ADDSData" -DiskSizeInGB $diskSize -VhdUri $vhdURI  -CreateOption empty
	
	$cred=Get-Credential -Message "Type the name and password of the local administrator account for the second domain controller." 
	$vm=Set-AzureRMVMOperatingSystem -VM $vm -Windows -ComputerName $vmName -Credential $cred -ProvisionVMAgent -EnableAutoUpdate
	$vm=Set-AzureRMVMSourceImage -VM $vm -PublisherName MicrosoftWindowsServer -Offer WindowsServer -Skus 2012-R2-Datacenter -Version "latest"
	$vm=Add-AzureRMVMNetworkInterface -VM $vm -Id $nic.Id
	$storageAcc=Get-AzureRMStorageAccount -ResourceGroupName $rgName -Name $saName
	$osDiskUri=$storageAcc.PrimaryEndpoints.Blob.ToString() + "vhds/" + $vmName + "-OSDisk.vhd"
	$vm=Set-AzureRMVMOSDisk -VM $vm -Name "OSDisk" -VhdUri $osDiskUri -CreateOption fromImage
	New-AzureRMVM -ResourceGroupName $rgName -Location $locName -VM $vm

> [AZURE.NOTE] 由于这些虚拟机用于 Intranet 应用程序，因此未为它们分配公共 IP 地址或 DNS 域名标签，也未向 Internet 公开。但是，这也意味着你无法从 Azure 门户预览连接到它们。当你查看虚拟机的属性时，“连接”按钮将不可用。使用虚拟机的专用 IP 地址或 Intranet DNS 名称通过“远程桌面连接”附件或其他远程桌面工具可以连接到虚拟机。

## 配置第一个域控制器

使用所选的远程桌面客户端并创建与第一个域控制器虚拟机的远程桌面连接。使用其 Intranet DNS 或计算机名称和本地管理员帐户的凭据。

接下来，你需要将额外的数据磁盘添加到第一个域控制器中。

### <a id="datadisk"></a>初始化空磁盘

1.	在服务器管理器的左窗格中，单击“文件和存储服务”，然后单击“磁盘”。
2.	在内容窗格的“磁盘”组中，单击“磁盘 2”（其“分区”设置为“未知”）。
3.	单击“任务”，然后单击“新建卷”。
4.	在新建卷向导的“开始之前”页上，单击“下一步”。
5.	在“选择服务器和磁盘”页上，单击“磁盘 2”，然后单击“下一步”。出现提示时，单击“确定”。
6.	在“指定卷的大小”页上，单击“下一步”。
7.	在“分配到驱动器号或文件夹”页上，单击“下一步”。
8.	在“选择文件系统设置”页上，单击“下一步”。
9.	在“确认选择”页上，单击“创建”。
10.	完成后，单击“关闭”。

接下来，测试第一个域控制器与你的组织网络上的位置的连接。

### <a id="testconn"></a>测试连接

1.	从桌面上打开 Windows PowerShell 提示符。
2.	使用 **ping** 命令 ping 你的组织网络上的资源的名称和 IP 地址。

此过程可确保 DNS 名称解析正常工作（即，已正确为虚拟机配置了本地 DNS 服务器）并且可以从跨界虚拟网络收发数据包。如果此基本测试失败，请联系你的 IT 部门以便对 DNS 名称解析和数据包传送问题进行故障排除。

接下来，从第一个域控制器上的 Windows PowerShell 命令提示符，运行以下命令：

	$domname="<DNS domain name of the domain for which this computer will be a domain controller>"
	Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
	Install-ADDSDomainController -InstallDns -DomainName $domname  -DatabasePath "F:\NTDS" -SysvolPath "F:\SYSVOL" -LogPath "F:\Logs"

系统将提示你提供域管理员帐户的凭据。计算机将重新启动。

## 配置第二个域控制器

使用所选的远程桌面客户端并创建与第二个域控制器虚拟机的远程桌面连接。使用其 Intranet DNS 或计算机名称和本地管理员帐户的凭据。

接下来，你需要将额外的数据磁盘添加到第二个域控制器中。请参阅[初始化空磁盘过程](#datadisk)。

接下来，从第二个域控制器上的 Windows PowerShell 提示符，运行以下命令：

	$domname="<DNS domain name of the domain for which this computer will be a domain controller>"
	Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
	Install-ADDSDomainController -InstallDns -DomainName $domname  -DatabasePath "F:\NTDS" -SysvolPath "F:\SYSVOL" -LogPath "F:\Logs"

系统将提示你提供域管理员帐户的凭据。计算机将重新启动。

接下来，需要更新你的虚拟网络的 DNS 服务器，使 Azure 可以为虚拟机分配两个新域控制器的 IP 地址用作其 DNS 服务器。请注意，此过程使用表 V（用于虚拟网络设置）和表 M（用于虚拟机）中的值。

1.	在 Azure 门户预览的左窗格中，单击“虚拟网络”，然后单击你的虚拟网络的名称（表 V - 项目 1 -“值”列）。
2.	在“设置”窗格中，单击“DNS 服务器”。
3.	在“DNS 服务器”窗格中，键入以下内容：
	- 对于**主 DNS 服务器**：表 V - 项目 6 -“值”列
	- 对于**辅助 DNS 服务器**：表 V - 项目 7 -“值”列
4.	在 Azure 门户预览的左窗格中，单击“虚拟机”。
5.	在“虚拟机”窗格中，单击第一个域控制器的名称（表 M - 项目 1-“虚拟机名称”列）。
6.	在虚拟机的窗格中，单击“重新启动”。
7.	启动第一个域控制器后，在“虚拟机”窗格中单击第二个域控制器的名称（表 M - 项目 2 -“虚拟机名称”列）。
8.	在虚拟机的窗格中，单击“重新启动”。等到第二个域控制器已启动。

请注意，我们重新启动两个域控制器，这样将不会为它们配置本地 DNS 服务器作为 DNS 服务器。因为它们本身就是 DNS 服务器，将它们提升为域控制器时，已自动为其配置本地 DNS 服务器作为 DNS 转发器。

接下来，我们需要创建一个 Active Directory 复制站点，以确保 Azure 虚拟网络中的服务器使用本地域控制器。使用域管理员帐户登录到主域控制器，并在管理员级别的 Windows PowerShell 提示符下运行以下命令：

	$vnet="<Table V - Item 1 - Value column>"
	$vnetSpace="<Table V - Item 5 - Value column>"
	New-ADReplicationSite -Name $vnet 
	New-ADReplicationSubnet -Name $vnetSpace -Site $vnet

接下来，添加要管理 SQL Server 虚拟机的用户帐户。

	New-ADUser -SamAccountName sqlservice -AccountPassword (read-host "Set user password" -assecurestring) -name "sqlservice" -enabled $true -PasswordNeverExpires $true -ChangePasswordAtLogon $false

下图使用占位符计算机名称显示成功完成此阶段后生成的配置。

![](./media/virtual-machines-windows-ps-lob-ph2/workload-lobapp-phase2.png)

## 后续步骤

- 若要继续配置此工作负荷，请使用[阶段 3](/documentation/articles/virtual-machines-windows-ps-lob-ph3/)。


<!---HONumber=Mooncake_0411_2016-->