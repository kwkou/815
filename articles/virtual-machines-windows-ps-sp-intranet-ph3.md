<properties
	pageTitle="SharePoint Server 2013 场（阶段 3）| Microsoft Azure"
	description="在 Azure 的 SharePoint Server 2013 场阶段 3 中创建计算机和 SQL Server 群集并启用可用性组。"
	documentationCenter=""
	services="virtual-machines-windows"
	authors="JoeDavies-MSFT"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="12/11/2015"
	wacn.date="06/07/2016"/>

# SharePoint Intranet 场工作负荷阶段 3：配置 SQL Server 基础结构

> [AZURE.NOTE]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model)。这篇文章介绍如何使用资源管理器部署模型，Microsoft 建议大多数新部署使用资源管理器模型替代经典部署模型。

在部署仅限 Intranet 的 SharePoint 2013 场（在 Azure 基础结构服务中通过 SQL Server AlwaysOn 可用性组进行）的这个阶段，你将创建和配置两台 SQL Server 计算机和一台群集多数节点计算机，然后将它们组合成 Windows Server 群集。

你必须在转到[阶段 4](/documentation/articles/virtual-machines-workload-intranet-sharepoint-phase4) 之前完成此阶段。请参阅[在 Azure 中通过 SQL Server AlwaysOn 可用性组部署 SharePoint](/documentation/articles/virtual-machines-workload-intranet-sharepoint-overview) 以了解所有阶段的相关信息。

> [AZURE.NOTE]这些指令使用 Azure 映像库中的 SQL Server 映像并将收取让你使用 SQL Server 许可证的持续费用。还可以在 Azure 中创建虚拟机并安装你自己的 SQL Server 许可证，但你必须具有软件保障和许可移动性才能在虚拟机（包括 Azure 虚拟机）上使用 SQL Server 许可证。有关在虚拟机上安装 SQL Server 的详细信息，请参阅[安装 SQL Server](https://msdn.microsoft.com/zh-cn/library/bb500469.aspx)。

## 在 Azure 中创建 SQL Server 群集虚拟机

有两个 SQL Server 虚拟机。一个 SQL Server 包含某一可用性组的主数据库副本。第二个 SQL Server 包含辅助备份副本。提供该备份是为了确保高可用性。其他虚拟机用于群集多数节点。

使用以下 PowerShell 命令块为三个服务器创建虚拟机。为变量指定值，并删除 < and > 字符。请注意，此 PowerShell 命令集使用下表中的值：

- 表 M，用于虚拟机
- 表 V，用于虚拟网络设置
- 表 S，用于子网
- 表 ST，用于存储帐户
- 表 A，用于可用性集

回想一下，你在[阶段 2：配置域控制器](/documentation/articles/virtual-machines-workload-intranet-sharepoint-phase2)中定义了表 M，在[阶段 1：配置 Azure](/documentation/articles/virtual-machines-workload-intranet-sharepoint-phase1) 中定义了表 V、表 S、表 ST 和表 A。

> [AZURE.NOTE]以下命令集使用 Azure PowerShell 1.0 及更高版本。有关详细信息，请参阅 [Azure PowerShell 1.0](https://azure.microsoft.com/blog/azps-1-0/)。

如果已提供所有适当的值，请在 Azure PowerShell 命令提示符下运行生成的块。


	# Set up key variables
	$rgName="<your resource group name>"
	$locName="<Azure location of your resource group>"
	$saName="<Table ST – Item 1 – Storage account name column>"
	$vnetName="<Table V – Item 1 – Value column>"
	$avName="<Table A – Item 2 – Availability set name column>"
	
	# Create the first SQL Server virtual machine
	$vmName="<Table M – Item 3 - Virtual machine name column>"
	$vmSize="<Table M – Item 3 - Minimum size column>"
	$vnet=Get-AzureRMVirtualNetwork -Name $vnetName -ResourceGroupName $rgName
	$nic=New-AzureRMNetworkInterface -Name ($vmName +"-NIC") -ResourceGroupName $rgName -Location $locName -SubnetId $vnet.Subnets[1].Id
	$avSet=Get-AzureRMAvailabilitySet –Name $avName –ResourceGroupName $rgName 
	$vm=New-AzureRMVMConfig -VMName $vmName -VMSize $vmSize -AvailabilitySetId $avset.Id
	
	$diskSize=<size of the extra disk for SQL data in GB>
	$diskLabel="<the label on the disk>"
	$storageAcc=Get-AzureRMStorageAccount -ResourceGroupName $rgName -Name $saName
	$vhdURI=$storageAcc.PrimaryEndpoints.Blob.ToString() + "vhds/" + $vmName + "-SQLDataDisk.vhd"
	Add-AzureRMVMDataDisk -VM $vm -Name $diskLabel -DiskSizeInGB $diskSize -VhdUri $vhdURI  -CreateOption empty
	
	$cred=Get-Credential -Message "Type the name and password of the local administrator account for the first SQL Server computer." 
	$vm=Set-AzureRMVMOperatingSystem -VM $vm -Windows -ComputerName $vmName -Credential $cred -ProvisionVMAgent -EnableAutoUpdate
	$vm=Set-AzureRMVMSourceImage -VM $vm -PublisherName MicrosoftSQLServer -Offer SQL2014-WS2012R2 -Skus Enterprise -Version "latest"
	$vm=Add-AzureRMVMNetworkInterface -VM $vm -Id $nic.Id
	$storageAcc=Get-AzureRMStorageAccount -ResourceGroupName $rgName -Name $saName
	$osDiskUri=$storageAcc.PrimaryEndpoints.Blob.ToString() + "vhds/" + $vmName + "-OSDisk.vhd"
	$vm=Set-AzureRMVMOSDisk -VM $vm -Name "OSDisk" -VhdUri $osDiskUri -CreateOption fromImage
	New-AzureRMVM -ResourceGroupName $rgName -Location $locName -VM $vm
	
	# Create the second SQL Server virtual machine
	$vmName="<Table M – Item 4 - Virtual machine name column>"
	$vmSize="<Table M – Item 4 - Minimum size column>"
	$nic=New-AzureRMNetworkInterface -Name ($vmName +"-NIC") -ResourceGroupName $rgName -Location $locName -SubnetId $vnet.Subnets[1].Id
	$vm=New-AzureRMVMConfig -VMName $vmName -VMSize $vmSize -AvailabilitySetId $avset.Id
	
	$diskSize=<size of the extra disk for SQL data in GB>
	$diskLabel="<the label on the disk>"
	$vhdURI=$storageAcc.PrimaryEndpoints.Blob.ToString() + "vhds/" + $vmName + "-ADDSDisk.vhd"
	Add-AzureRMVMDataDisk -VM $vm -Name $diskLabel -DiskSizeInGB $diskSize -VhdUri $vhdURI  -CreateOption empty
	
	$cred=Get-Credential -Message "Type the name and password of the local administrator account for the second SQL Server computer." 
	$vm=Set-AzureRMVMOperatingSystem -VM $vm -Windows -ComputerName $vmName -Credential $cred -ProvisionVMAgent -EnableAutoUpdate
	$vm=Set-AzureRMVMSourceImage -VM $vm -PublisherName MicrosoftSQLServer -Offer SQL2014-WS2012R2 -Skus Enterprise -Version "latest"
	$vm=Add-AzureRMVMNetworkInterface -VM $vm -Id $nic.Id
	$storageAcc=Get-AzureRMStorageAccount -ResourceGroupName $rgName -Name $saName
	$osDiskUri=$storageAcc.PrimaryEndpoints.Blob.ToString() + "vhds/" + $vmName + "-OSDisk.vhd"
	$vm=Set-AzureRMVMOSDisk -VM $vm -Name "OSDisk" -VhdUri $osDiskUri -CreateOption fromImage
	New-AzureRMVM -ResourceGroupName $rgName -Location $locName -VM $vm
	
	# Create the cluster majority node server
	$vmName="<Table M – Item 5 - Virtual machine name column>"
	$vmSize="<Table M – Item 5 - Minimum size column>"
	$nic=New-AzureRMNetworkInterface -Name ($vmName +"-NIC") -ResourceGroupName $rgName -Location $locName -SubnetId $vnet.Subnets[1].Id
	$vm=New-AzureRMVMConfig -VMName $vmName -VMSize $vmSize -AvailabilitySetId $avset.Id
	$cred=Get-Credential -Message "Type the name and password of the local administrator account for the cluster majority node server." 
	$vm=Set-AzureRMVMOperatingSystem -VM $vm -Windows -ComputerName $vmName -Credential $cred -ProvisionVMAgent -EnableAutoUpdate
	$vm=Set-AzureRMVMSourceImage -VM $vm -PublisherName MicrosoftWindowsServer -Offer WindowsServer -Skus 2012-R2-Datacenter -Version "latest"
	$vm=Add-AzureRMVMNetworkInterface -VM $vm -Id $nic.Id
	$storageAcc=Get-AzureRMStorageAccount -ResourceGroupName $rgName -Name $saName
	$osDiskUri=$storageAcc.PrimaryEndpoints.Blob.ToString() + "vhds/" + $vmName + "-OSDisk.vhd"
	$vm=Set-AzureRMVMOSDisk -VM $vm -Name "OSDisk" -VhdUri $osDiskUri -CreateOption fromImage
	New-AzureRMVM -ResourceGroupName $rgName -Location $locName -VM $vm

> [AZURE.NOTE]由于这些虚拟机用于 Intranet 应用程序，因此未为它们分配公共 IP 地址或 DNS 域名标签，也未向 Internet 公开。但是，这也意味着你无法从 Azure 门户连接到它们。当你查看虚拟机的属性时，“连接”按钮将不可用。使用虚拟机的专用 IP 地址或 Intranet DNS 名称通过“远程桌面连接”附件或其他远程桌面工具可以连接到虚拟机。

## 配置 SQL Server 计算机

对于运行 SQL Server 的每个虚拟机，使用所选的远程桌面客户端创建远程桌面连接。使用其 Intranet DNS 或计算机名称和本地管理员帐户的凭据。

对于每个运行 SQL Server 的虚拟机，在 Windows PowerShell 提示符下使用这些命令将它们加入到相应的 AD DS 域。

	$domName="<AD DS domain name to join, such as corp.contoso.com>"
	Add-Computer -DomainName $domName
	Restart-Computer

请注意，必须在输入 Add-Computer 命令后提供域帐户凭据。

在这些虚拟机重新启动后，使用具有本地管理员权限的帐户重新连接到它们。

对于每个 SQL Server，执行以下操作：

1. 按照[使用远程桌面连接过程登录到虚拟机](/documentation/articles/virtual-machines-workload-intranet-sharepoint-phase2#logon)使用本地管理员帐户的凭据登录。

2. 使用[初始化空磁盘过程](/documentation/articles/virtual-machines-workload-intranet-sharepoint-phase2#datadisk)两次（对每个 SQL Server 使用一次），以添加额外的数据磁盘。

3. 在 Windows PowerShell 命令提示符下运行以下命令。

		md f:\Data
		md f:\Log
		md f:\Backup

4. 使用[测试连接过程](/documentation/articles/virtual-machines-workload-intranet-sharepoint-phase2#testconn)测试与你的组织网络上的位置的连接。此过程可确保 DNS 名称解析正常工作（即，已正确为虚拟机配置了虚拟网络中的 DNS 服务器）并且可以从跨界虚拟网络收发数据包。

使用以下过程两次（对每个 SQL Server 使用一次），以将 SQL Server 配置为将 F: 驱动器用于新数据库和用于帐户和权限。

1.	在“开始”屏幕上，键入 **SQL Studio**，然后单击 **SQL Server 2014 Management Studio**。
2.	在**“连接到服务器”**中，单击**“连接”**。
3.	在左窗格中，右键单击顶级节点（按计算机命名的默认实例），然后单击**“属性”**。
4.	在**“服务器属性”**中，单击**“数据库设置”**。
5.	在**“数据库默认位置”**中，设置以下值：
- 对于**“数据”**，将路径设置为 **f:\\Data**。
- 对于**“日志”**，将路径设置为 **f:\\Log**。
- 对于**“备份”**，将路径设置为 **f:\\Backup**。
- 只有新数据库使用这些位置。
6.	单击**“确定”**以关闭该窗口。
7.	在左窗格中，展开**“安全性文件夹”**。
8.	右键单击**“登录名”**，然后单击**“新建登录名”**。
9.	在**“登录名”**中，键入 *domain*\\sp\_farm\_db（其中 *domain* 是创建 sp\_farm\_db 帐户的域的名称）。
10.	在**“选择页”**下，单击**“服务器角色”**，单击 **sysadmin**，然后单击**“确定”**。
11.	关闭 SQL Server 2014 Management Studio。

使用以下过程两次（对每个 SQL Server 使用一次），以允许使用 sp\_farm\_db 帐户进行远程桌面连接。

1.	在“开始”屏幕上，右键单击**“这台电脑”**，然后单击**“属性”**。
2.	在**“系统”**窗口中，单击**“远程设置”**。
3.	在**“远程桌面”**部分中，单击**“选择用户”**，然后单击**“添加”**。
4.	在“输入要选择的对象名称”中，键入 [domain]**\\sp\_farm\_db**，然后单击“确定”三次。

SQL Server 需要客户端用于访问数据库服务器的端口。它还需要用于与 SQL Server Management Studio 连接的端口和用于管理高可用性组的端口。接下来，在管理员级别的 Windows PowerShell 命令提示符下运行以下命令两次（对每个 SQL Server 运行一次），以添加允许到 SQL Server 的入站流量的防火墙规则。

	New-NetFirewallRule -DisplayName "SQL Server ports 1433, 1434, and 5022" -Direction Inbound –Protocol TCP –LocalPort 1433,1434,5022 -Action Allow

对于每个 SQL Server 虚拟机，以本地管理员身份注销。

有关优化 Azure 中的 SQL Server 性能的信息，请参阅 [Azure 虚拟机中 SQL Server 的性能最佳实践](/documentation/articles/virtual-machines-sql-server-performance-best-practices)。你还可以为 SharePoint 场存储帐户禁用地域冗余存储 (GRS)，并使用存储空间来优化 IOP。

## 配置群集多数节点服务器

针对群集多数节点按照[使用远程桌面连接过程登录到虚拟机](virtual-machines-workload-intranet-sharepoint-phase2#logon)使用域帐户的凭据登录。

在群集多数节点上，使用[测试连接过程](/documentation/articles/virtual-machines-workload-intranet-sharepoint-phase2#testconn)测试与你的组织网络上的位置的连接。

## 创建 Windows Server 群集

SQL Server AlwaysOn 可用性组依赖于 Windows Server 的 Windows Server 故障转移群集 (WSFC) 功能。该功能允许多台计算机作为群集中的一个组参与。当一台计算机失败时，第二台计算机已准备好接替它。因此，第一个任务是在所有参与的计算机上启用故障转移群集功能，这些计算机包括：

- 主 SQL Server
- 辅助 SQL Server
- 群集多数节点

故障转移群集要求至少三个 VM。两台计算机托管 SQL Server。第二个 SQL Server VM 是同步的辅助副本，并确保在主计算机出现故障时无数据丢失。第三台计算机不需要托管 SQL Server。群集多数节点在 WSFC 中提供了仲裁。由于 WSFC 群集依赖于仲裁来监视运行状况，因此必须始终有一个主体来确保 WSFC 群集处于联机状态。如果群集中只有两台计算机，并且其中一台出现故障，则在两台中计算机只有一台出现故障时可能没有主体。有关详细信息，请参阅 [WSFC 仲裁模式和投票配置 (SQL Server)](http://msdn.microsoft.com/zh-cn/library/hh270280.aspx)。

针对这两台 SQL Server 计算机和群集多数节点，在管理员级别的 Windows PowerShell 命令提示符下运行以下命令。

	Install-WindowsFeature Failover-Clustering -IncludeManagementTools

由于 Azure 中的 DHCP 出现当前不符合 RFC 的行为，创建 Windows Server 故障转移群集 (WSFC) 群集可能会失败。有关详细信息，请在“Azure 虚拟机中的 SQL Server 的高可用性和灾难恢复”中搜索“Azure 网络中的 WSFC 群集行为”。但是，有一种变通解决方法。使用以下步骤创建群集。

1.	使用 **sp\_install** 帐户登录到主 SQL Server 虚拟机。
2.	在“开始”屏幕中，键入**“故障转移”**，然后单击**“故障转移群集管理器”**。
3.	在左窗格中，右键单击**“故障转移群集管理器”**，然后单击**“创建群集”**。
4.	在“开始之前”页上，单击“下一步”。
5.	在“选择服务器”页上，键入主 SQL Server 计算机的名称，单击**“添加”**，然后单击**“下一步”**。
6.	在“验证警告”页上，单击**“否，我不需要 Microsoft 对该群集的支持，因此不希望运行验证测试。当我单击“下一步”时，继续创建群集**，然后单击“下一步”。
7.	在“用于管理群集的访问点”页的“群集名称”文本框中，键入你的群集的名称，然后单击“下一步”。
8.	在“确认”页中，单击**“下一步”**以开始创建群集。
9.	在“摘要”页上，单击**“完成”**。
10.	在左窗格中，单击新群集。在内容窗格的**“群集核心资源”**部分中，打开你的服务器群集名称。**“IP 地址”**资源将显示为**“失败”**状态。由于为该群集分配了与计算机本身相同的 IP 地址，IP 地址资源将无法联机。结果是重复的地址。
11.	右键单击失败的**“IP 地址”**资源，然后单击**“属性”**。
12.	在**“IP 地址属性”**对话框中，单击**“静态 IP 地址”**。
13.	键入与 SQL Server 所在的子网对应的地址范围中未用的 IP，然后单击**“确定”**。
14.	右键单击失败的“IP 地址”资源，然后单击**“联机”**。等到这两个资源均已联机。当群集名称资源处于联机状态时，它会使用新的 Active Directory (AD) 计算机帐户更新域控制器。稍后将使用此 AD 帐户运行可用性组群集服务。
15.	现在，已创建 AD 帐户，请将群集名称脱机。右键单击**“群集核心资源”**中的群集名称，然后单击**“脱机”**。
16.	若要删除群集 IP 地址，请右键单击**“IP 地址”**，单击**“删除”**，然后在出现提示时单击**“是”**。该群集资源将不能再进入联机状态，因为它依赖于 IP 地址资源。但是，可用性组不依赖群集名称或 IP 地址就能正常工作。因此，可以使群集名称保持处于脱机状态。
17.	要向群集添加剩余的节点，请右键单击左窗格中的群集名称，然后单击“添加节点”。
18.	在“开始之前”页上，单击“下一步”。
19.	在“选择服务器”页上，键入名称，然后单击**“添加”**以将辅助 SQL Server 和群集多数节点添加到群集。添加这两台计算机后，请单击**“下一步”**。如果无法添加计算机，并且错误消息为“远程注册表服务未运行”，请执行以下操作。登录到计算机，打开“服务”管理单元 (services.msc) 并启用远程注册表。有关详细信息，请参阅[无法连接到远程注册表服务](http://technet.microsoft.com/library/bb266998.aspx)。
20.	在“验证警告”页上，单击**“否，我不需要 Microsoft 对该群集的支持，因此不希望运行验证测试。当我单击“下一步”时，继续创建群集**，然后单击“下一步”。
21.	在“确认”页上，单击**“下一步”**。
22.	在“摘要”页上，单击**“完成”**。
23.	在左窗格中，单击**“节点”**。你应看到所有三台计算机均列出。

## 启用 AlwaysOn 可用性组

下一步是使用 SQL Server 配置管理器启用 AlwaysOn 可用性组。请注意，SQL Server 中的可用性组与 Azure 可用性集不同。可用性组包含高度可用且可恢复的数据库。Azure 可用性集将虚拟机分配给不同容错域。有关容错域的详细信息，请参阅[管理虚拟机的可用性](/documentation/articles/virtual-machines-manage-availability)。

使用以下步骤在 SQL Server 上启用 AlwaysOn 可用性组。

1.	使用 **sp\_farm\_db** 帐户或在 SQL Server 具有 sysadmin 服务器角色的某个其他帐户登录到主 SQL Server。
2.	在“开始”屏幕上，键入**“SQL Server 配置”**，然后单击**“SQL Server 配置管理器”**。
3.	在左窗格中，单击**“SQL Server 服务”**。
4.	在内容窗格中，双击 **SQL Server (MSSQLSERVER)**。
5.	在“SQL Server (MSSQLSERVER)属性”中，单击“AlwaysOn 高可用性”选项卡，选择“启用 AlwaysOn 可用性组”，单击“应用”，然后在出现提示时单击“确定”。尚不要关闭属性窗口。
6.	单击“虚拟机管理可用性”选项卡，然后在**“帐户名称”**中键入 [Domain]**\\sqlservice**。在**“密码”**和**“确认密码”**中键入 sqlservice 帐户密码，然后单击**“确定”**。
7.	在消息窗口中，单击**“是”**以重新启动 SQL Server 服务。
8.	登录到辅助 SQL Server 并重复此过程。

下图使用占位符计算机名称显示成功完成此阶段后生成的配置。

![](./media/virtual-machines-workload-intranet-sharepoint-phase3/workload-spsqlao_03.png)

## 后续步骤

- 若要继续配置此工作负荷，请使用[阶段 4](/documentation/articles/virtual-machines-workload-intranet-sharepoint-phase4)。

<!---HONumber=Mooncake_0104_2016-->