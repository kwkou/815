<properties
	pageTitle="SharePoint Intranet 场工作负荷阶段 3：配置 SQL Server 基础结构"
	description="在部署仅限 Intranet 的 SharePoint 2013 场（在 Azure 基础结构服务中通过 SQL Server AlwaysOn 可用性组进行）的这个第三阶段，你将创建 SQL Server 群集计算机和群集本身。"
	documentationCenter=""
	services="virtual-machines"
	authors="JoeDavies-MSFT"
	manager="timlt"
	editor=""
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines"
	ms.date="07/21/2015"
	wacn.date="09/18/2015"/>

# SharePoint Intranet 场工作负荷阶段 3：配置 SQL Server 基础结构

在部署仅限 Intranet 的 SharePoint 2013 场（在 Azure 基础结构服务中通过 SQL Server AlwaysOn 可用性组进行）的这个阶段，你将在服务管理中创建并配置两台 SQL Server 计算机和一台群集多数节点计算机，然后将它们组合成 Windows Server 群集。

你必须在转到[阶段 4](/documentation/articles/virtual-machines-workload-intranet-sharepoint-phase4) 之前完成此阶段。请参阅[在 Azure 中通过 SQL Server AlwaysOn 可用性组部署 SharePoint](/documentation/articles/virtual-machines-workload-intranet-sharepoint-overview) 以了解所有阶段的相关信息。

## 在 Azure 中创建 SQL Server 群集虚拟机

有两个 SQL Server 虚拟机。一个 SQL Server 包含某一可用性组的主数据库副本。第二个 SQL Server 包含辅助备份副本。提供该备份是为了确保高可用性。其他虚拟机用于群集多数节点。

使用以下 PowerShell 命令块为三个服务器创建虚拟机。为变量指定值，并删除 < and > 字符。请注意，此 PowerShell 命令集使用下表中的值：

- 表 M，用于虚拟机。
- 表 V，用于虚拟网络设置。
- 表 S，用于子网。
- 表 A，用于可用性集。
- 表 C，用于云服务。

回想一下，你在[阶段 2：配置域控制器](/documentation/articles/virtual-machines-workload-intranet-sharepoint-phase2)中定义了表 M，在[阶段 1：配置 Azure](/documentation/articles/virtual-machines-workload-intranet-sharepoint-phase1) 中定义了表 V、表 S、表 A 和表 C。

如果已提供所有适当的值，请在 Azure PowerShell 命令提示符下运行生成的块。

	# Create the first SQL server
	$vmName="<Table M – Item 3 - Virtual machine name column>"
	$vmSize="<Table M – Item 3 - Minimum size column, specify one: Small, Medium, Large, ExtraLarge, A5, A6, A7, A8, A9>"
	$availSet="<Table A – Item 2 – Availability set name column>"

	$image= Get-AzureVMImage | where { $_.ImageFamily -eq "SQL Server 2014 RTM Enterprise on Windows Server 2012 R2" } | sort PublishedDate -Descending | select -ExpandProperty ImageName -First 1
	$vm1=New-AzureVMConfig -Name $vmName -InstanceSize $vmSize -ImageName $image -AvailabilitySetName $availSet

	$cred1=Get-Credential –Message "Type the name and password of the local administrator account for the first SQL Server computer."
	$cred2=Get-Credential –Message "Now type the name and password of an account that has permissions to add this virtual machine to the domain."
	$ADDomainName="<name of the AD domain that the server is joining (example CORP)>"
	$domainDNS="<FQDN of the AD domain that the server is joining (example corp.contoso.com)>"
	$vm1 | Add-AzureProvisioningConfig -AdminUsername $cred1.GetNetworkCredential().Username -Password $cred1.GetNetworkCredential().Password -WindowsDomain -Domain $ADDomainName -DomainUserName $cred2.GetNetworkCredential().Username -DomainPassword $cred2.GetNetworkCredential().Password -JoinDomain $domainDNS

	$diskSize=<size of the additional data disk in GB>
	$diskLabel="<the label on the disk>"
	$lun=<Logical Unit Number (LUN) of the disk>
	$vm1 | Add-AzureDataDisk -CreateNew -DiskSizeInGB $diskSize -DiskLabel $diskLabel -LUN $lun -HostCaching None

	$subnetName="<Table 6 – Item 1 – Subnet name column>"
	$vm1 | Set-AzureSubnet -SubnetNames $subnetName

	$serviceName="<Table C – Item 2 – Cloud service name column>"
	$vnetName="<Table V – Item 1 – Value column>"
	New-AzureVM –ServiceName $serviceName -VMs $vm1 -VNetName $vnetName

	# Create the second SQL server
	$vmName="<Table M – Item 4 - Virtual machine name column>"
	$vmSize="<Table M – Item 4 - Minimum size column, specify one: Small, Medium, Large, ExtraLarge, A5, A6, A7, A8, A9>"
	$vm1=New-AzureVMConfig -Name $vmname -InstanceSize $vmsize -ImageName $image -AvailabilitySetName $availSet

	$cred1=Get-Credential –Message "Type the name and password of the local administrator account for the second SQL Server computer."
	$vm1 | Add-AzureProvisioningConfig -AdminUsername $cred1.GetNetworkCredential().Username -Password $cred1.GetNetworkCredential().Password -WindowsDomain -Domain $ADDomainName -DomainUserName $cred2.GetNetworkCredential().Username -DomainPassword $cred2.GetNetworkCredential().Password -JoinDomain $domainDNS

	$diskSize=<size of the additional data disk in GB>
	$diskLabel="<the label on the disk>"
	$lun=<Logical Unit Number (LUN) of the disk>
	$vm1 | Add-AzureDataDisk -CreateNew -DiskSizeInGB $diskSize -DiskLabel $diskLabel -LUN $lun -HostCaching None

	$vm1 | Set-AzureSubnet -SubnetNames $subnetName

	New-AzureVM –ServiceName $serviceName -VMs $vm1 -VNetName $vnetName

	# Create the cluster majority node server
	$vmName="<Table M – Item 5 - Virtual machine name column>"
	$vmSize="<Table M – Item 5 - Minimum size column, specify one: Small, Medium, Large, ExtraLarge, A5, A6, A7, A8, A9>"
	$image= Get-AzureVMImage | where { $_.ImageFamily -eq "Windows Server 2012 R2 Datacenter" } | sort PublishedDate -Descending | select -ExpandProperty ImageName -First 1
	$vm1=New-AzureVMConfig -Name $vmName -InstanceSize $vmSize -ImageName $image -AvailabilitySetName $availSet

	$cred1=Get-Credential –Message "Type the name and password of the local administrator account for the cluster majority node server."
	$vm1 | Add-AzureProvisioningConfig -AdminUsername $cred1.GetNetworkCredential().Username -Password $cred1.GetNetworkCredential().Password -WindowsDomain -Domain $ADDomainName -DomainUserName $cred2.GetNetworkCredential().Username -DomainPassword $cred2.GetNetworkCredential().Password -JoinDomain $domainDNS

	$vm1 | Set-AzureSubnet -SubnetNames $subnetName

	New-AzureVM –ServiceName $serviceName -VMs $vm1 -VNetName $vnetName

## 配置 SQL Server 计算机

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

	New-NetFirewallRule -DisplayName "SQL Server ports 1433, 4234, and 5022" -Direction Inbound –Protocol TCP –LocalPort 1433,1434,5022 -Action Allow

对于每个 SQL Server 虚拟机，以本地管理员身份注销。

有关优化 Azure 中的 SQL Server 性能的信息，请参阅 [Azure 虚拟机中 SQL Server 的性能最佳实践](https://msdn.microsoft.com/zh-cn/library/azure/dn133149.aspx)。你还可以为 SharePoint 场存储帐户禁用地域冗余存储 (GRS)，并使用存储空间来优化 IOP。

## 配置群集多数节点服务器

针对群集多数节点按照[使用远程桌面连接过程登录到虚拟机](/documentation/articles/virtual-machines-workload-intranet-sharepoint-phase2#logon)使用域帐户的凭据登录。

在群集多数节点上，使用[测试连接过程](/documentation/articles/virtual-machines-workload-intranet-sharepoint-phase2#testconn)测试与你的组织网络上的位置的连接。

## 创建 Windows Server 群集

SQL Server AlwaysOn 可用性组依赖于 Windows Server 的 Windows Server 故障转移群集 (WSFC) 功能。该功能允许多台计算机作为群集中的一个组参与。当一台计算机失败时，第二台计算机已准备好接替它。因此，第一个任务是在所有参与的计算机上启用故障转移群集功能，这些计算机包括：

- 主 SQL Server
- 辅助 SQL Server
- 群集多数节点

故障转移群集要求至少三个 VM。两台计算机托管 SQL Server。第二个 SQL Server VM 是同步的辅助副本，并确保在主计算机出现故障时无数据丢失。第三台计算机不需要托管 SQL Server。群集多数节点充当 WSFC 中的仲裁见证。由于 WSFC 群集依赖于仲裁来监视运行状况，因此必须始终有一个主体来确保 WSFC 群集处于联机状态。如果群集中只有两台计算机，并且其中一台出现故障，则在两台中计算机只有一台出现故障时可能没有主体。有关详细信息，请参阅 [WSFC 仲裁模式和投票配置 (SQL Server)](http://msdn.microsoft.com/zh-cn/library/hh270280.aspx)。

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
19.	在“选择服务器”页上，键入名称，然后单击**“添加”**以将辅助 SQL Server 和群集多数节点添加到群集。添加这两台计算机后，请单击**“下一步”**。如果无法添加计算机，并且错误消息为“远程注册表服务未运行”，请执行以下操作。登录到计算机，打开“服务”管理单元 (services.msc) 并启用远程注册表。有关详细信息，请参阅[无法连接到远程注册表服务](http://technet.microsoft.com/zh-cn/library/bb266998.aspx)。
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
6.	单击“虚拟机管理可用性”选项卡，然后在“帐户名称”中键入 [Domain]**\\sqlservice**。在“密码”和“确认密码”中键入 sqlservice 帐户密码，然后单击“确定”。
7.	在消息窗口中，单击**“是”**以重新启动 SQL Server 服务。
8.	登录到辅助 SQL Server 并重复此过程。

下图使用占位符计算机名称显示成功完成此阶段后生成的配置。

![](./media/virtual-machines-workload-intranet-sharepoint-phase3/workload-spsqlao_03.png)

## 后续步骤

若要继续配置此工作负荷，请转到[阶段 4：配置 SharePoint Server](/documentation/articles/virtual-machines-workload-intranet-sharepoint-phase4)。

## 其他资源

[在 Azure 中通过 SQL Server AlwaysOn 可用性组部署 SharePoint](/documentation/articles/virtual-machines-workload-intranet-sharepoint-overview)

[Azure 基础结构服务中托管的 SharePoint 场](/documentation/articles/virtual-machines-sharepoint-infrastructure-services)

[具有 SQL Server AlwaysOn 的 SharePoint 信息图](http://go.microsoft.com/fwlink/?LinkId=394788)

[适用于 SharePoint 2013 的 Windows Azure 体系结构](https://technet.microsoft.com/zh-cn/library/dn635309.aspx)

[Azure 基础结构服务实施准则](/documentation/articles/virtual-machines-infrastructure-services-implementation-guidelines)

[Azure 基础结构服务工作负荷：高可用性业务线应用程序](/documentation/articles/virtual-machines-workload-high-availability-LOB-application)

<!---HONumber=70-->