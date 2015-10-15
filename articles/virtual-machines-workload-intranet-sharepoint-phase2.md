<properties
	pageTitle="SharePoint Intranet 场工作负荷阶段 2：配置域控制器"
	description="在部署仅限 Intranet 的 SharePoint 2013 场（在 Azure 基础结构服务中通过 SQL Server AlwaysOn 可用性组进行）的这个第二阶段，你将创建并配置两个 Active Directory 域控制器。"
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

# SharePoint Intranet 场工作负荷阶段 2：配置域控制器

在部署仅限 Intranet 的 SharePoint 2013 场（在 Azure 基础结构服务中通过 SQL Server AlwaysOn 可用性组进行）的这个阶段，你将在服务管理中配置 Azure 虚拟网络中的两个域控制器。然后，针对 SharePoint 场的客户端 Web 请求可在 Azure 虚拟网络中进行身份验证，而无需通过与本地网络的 VPN 或 Azure ExpressRoute 连接发送身份验证流量。

你必须在转到[阶段 3](/documentation/articles/virtual-machines-workload-intranet-sharepoint-phase3) 之前完成此阶段。请参阅[在 Azure 中通过 SQL Server AlwaysOn 可用性组部署 SharePoint](/documentation/articles/virtual-machines-workload-intranet-sharepoint-overview) 以了解所有阶段的相关信息。

## 在 Azure 中创建域控制器虚拟机

首先，你需要填写表 M 的**“虚拟机名称”**列，并根据需要在**“最小大小”**列中修改虚拟机大小。

项目 | 虚拟机名称 | 库映像 | 最小大小
--- | --- | --- | ---
1\. | \_\_\_\_\_\_\_\_\_\_\_\_\_\_（第一个域控制器，示例 DC1） | Windows Server 2012 R2 Datacenter | A2（中）
2\. | \_\_\_\_\_\_\_\_\_\_\_\_\_\_（第二个域控制器，示例 DC2） | Windows Server 2012 R2 Datacenter | A2（中）
3\. | \_\_\_\_\_\_\_\_\_\_\_\_\_\_（第一台 SQL Server 计算机，示例 SQL1） | Microsoft SQL Server 2014 Enterprise - Windows Server 2012 R2 | 	A7
4\. | \_\_\_\_\_\_\_\_\_\_\_\_\_\_（第二台 SQL Server 计算机，示例 SQL2） | Microsoft SQL Server 2014 Enterprise - Windows Server 2012 R2 | 	A7
5\. | \_\_\_\_\_\_\_\_\_\_\_\_\_\_（群集的多数节点见证，示例 MN1） | Windows Server 2012 R2 Datacenter | A1（小）
6\. | \_\_\_\_\_\_\_\_\_\_\_\_\_\_（第一台 SharePoint 应用程序服务器，示例 APP1） | Microsoft SharePoint Server 2013 试用版 - Windows Server 2012 R2 | A4（超大）
7\. | \_\_\_\_\_\_\_\_\_\_\_\_\_\_（第二台 SharePoint 应用程序服务器，示例 APP2） | Microsoft SharePoint Server 2013 试用版 - Windows Server 2012 R2 | A4（超大）
8\. | \_\_\_\_\_\_\_\_\_\_\_\_\_\_（第一个 SharePoint Web 服务器，示例 WEB1） | Microsoft SharePoint Server 2013 试用版 - Windows Server 2012 R2 | A4（超大）
9\. | \_\_\_\_\_\_\_\_\_\_\_\_\_\_（第二个 SharePoint Web 服务器，示例 WEB2） | Microsoft SharePoint Server 2013 试用版 - Windows Server 2012 R2 | A4（超大）

**表 M - Azure 中的 SharePoint 2013 Intranet 场的虚拟机**

有关虚拟机大小的完整列表，请参阅[虚拟机的大小](/documentation/articles/virtual-machines-size-specs)。

使用以下 Azure PowerShell 命令块为两个域控制器创建虚拟机。为变量指定值，并删除 < and > 字符。请注意，此 Azure PowerShell 命令集使用下表中的值：

- 表 M，用于虚拟机
- 表 V，用于虚拟网络设置
- 表 S，用于子网
- 表 A，用于可用性集
- 表 C，用于云服务

回想一下，你在[阶段 1：配置 Azure](/documentation/articles/virtual-machines-workload-intranet-sharepoint-phase1) 中定义了表 V、表 S、表 A 和表 C。

如果已提供所有适当的值，请在 Azure PowerShell 命令提示符下运行生成的块。

	# Create the first domain controller
	$vmName="<Table M – Item 1 - Virtual machine name column>"
	$vmSize="<Table M – Item 1 - Minimum size column, specify one: Small, Medium, Large, ExtraLarge, A5, A6, A7, A8, A9>"
	$availSet="<Table A – Item 1 – Availability set name column>"
	$image= Get-AzureVMImage | where { $_.ImageFamily -eq "Windows Server 2012 R2 Datacenter" } | sort PublishedDate -Descending | select -ExpandProperty ImageName -First 1
	$vm1=New-AzureVMConfig -Name $vmName -InstanceSize $vmSize -ImageName $image -AvailabilitySetName $availSet

	$cred=Get-Credential –Message "Type the name and password of the local administrator account for the first domain controller."
	$vm1 | Add-AzureProvisioningConfig -Windows -AdminUsername $cred.GetNetworkCredential().Username -Password $cred.GetNetworkCredential().Password

	$diskSize=<size of the additional data disk in GB>
	$diskLabel="<the label on the disk>"
	$lun=<Logical Unit Number (LUN) of the disk>
	$vm1 | Add-AzureDataDisk -CreateNew -DiskSizeInGB $diskSize -DiskLabel $diskLabel -LUN $lun -HostCaching None

	$subnetName="<Table S – Item 1 – Subnet name column>"
	$vm1 | Set-AzureSubnet -SubnetNames $subnetName

	$vm1 | Set-AzureStaticVNetIP -IPAddress <Table V – Item 6 – Value column>

	$serviceName="<Table C – Item 1 – Cloud service name column>"
	$vnetName="<Table V – Item 1 – Value column>"
	New-AzureVM –ServiceName $serviceName -VMs $vm1 -VNetName $vnetName

	# Create the second domain controller
	$vmName="<Table M – Item 2 - Virtual machine name column>"
	$vmSize="<Table M – Item 2 - Minimum size column, specify one: Small, Medium, Large, ExtraLarge, A5, A6, A7, A8, A9>"
	$image= Get-AzureVMImage | where { $_.ImageFamily -eq "Windows Server 2012 R2 Datacenter" } | sort PublishedDate -Descending | select -ExpandProperty ImageName -First 1

	$vm1=New-AzureVMConfig -Name $vmName -InstanceSize $vmSize -ImageName $image -AvailabilitySetName $availSet

	$cred=Get-Credential –Message "Type the name and password of the local administrator account for the second domain controller."
	$vm1 | Add-AzureProvisioningConfig -Windows -AdminUsername $cred.GetNetworkCredential().Username -Password $cred.GetNetworkCredential().Password

	$diskSize=<size of the additional data disk in GB>
	$diskLabel="<the label on the disk>"
	$lun=<Logical Unit Number (LUN) of the disk>
	$vm1 | Add-AzureDataDisk -CreateNew -DiskSizeInGB $diskSize -DiskLabel $diskLabel -LUN $lun -HostCaching None

	$vm1 | Set-AzureSubnet -SubnetNames $subnetName

	$vm1 | Set-AzureStaticVNetIP -IPAddress <Table V – Item 7 – Value column>

	New-AzureVM –ServiceName $serviceName -VMs $vm1 -VNetName $vnetName

## 配置第一个域控制器

使用本地管理员帐户的凭据登录到第一个域控制器计算机。

### <a id="logon"></a>使用远程桌面连接登录到虚拟机

1.	在 Azure 门户的左窗格中，单击“虚拟机”。
2.	若要连接到 VM，请单击其名称旁边的**“状态”**列中的**“正在运行”**。
3.	在页面底部的命令栏中，单击**“连接”**。
4.	此时门户会通知你正在检索 .rdp 文件。单击**“确定”**。
5.	此时浏览器对话框会出现并询问“要打开还是要保存来自 manage.windowsazure.com 的 ComputerName.rdp?” 单击**“打开”**。
6.	在**“远程桌面连接”**对话框中，单击**“连接”**。
7.	在**“Windows 安全性”**对话框中，单击**“使用另一帐户”**。
8.	在**“用户名”**中，键入 VM 的名称以及随 VM 一起创建的本地管理员帐户（本地计算机帐户）的用户名。使用以下格式：*计算机名*\*本地管理员帐户名*
9.	在**“密码”**中，键入本地管理员帐户的密码。
10.	单击**“确定”**。
11.	在**“远程桌面连接”**对话框中，单击**“是”**。“远程桌面”会话窗口中将出现新计算机的桌面。

接下来，你需要将额外的数据磁盘添加到第一个域控制器中。

### <a id="datadisk"></a>初始化空磁盘

1.	在服务器管理器的左窗格中，单击**“文件和存储服务”**，然后单击**“磁盘”**。
2.	在内容窗格的**“磁盘”**组中，单击**“磁盘 2”**（其**“分区”**设置为**“未知”**）。
3.	单击**“任务”**，然后单击**“新建卷”**。
4.	在新建卷向导的“开始之前”页上，单击“下一步”。
5.	在“选择服务器和磁盘”页上，单击“磁盘 2”，然后单击“下一步”。出现提示时，单击“确定”。
6.	在“指定卷的大小”页上，单击“下一步”。
7.	在“分配到驱动器号或文件夹”页上，单击“下一步”。
8.	在“选择文件系统设置”页上，单击“下一步”。
9.	在“确认选择”页上，单击“创建”。
10.	初始化完成后，单击“关闭”。

接下来，测试第一个域控制器与你的组织网络上的位置的连接。

### <a id="testconn"></a>测试连接

1.	从桌面上打开 Windows PowerShell 命令提示符。
2.	使用 **ping** 命令 ping 你的组织网络上的资源的名称和 IP 地址。

此过程可确保 DNS 名称解析正常工作（即，已正确为虚拟机配置了本地 DNS 服务器）并且可以从跨界虚拟网络收发数据包。

接下来，从第一个域控制器上的 Windows PowerShell 命令提示符，运行以下命令：

	$domname="<DNS domain name of the domain for which this computer will be a domain controller>"
	Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
	Install-ADDSDomainController -InstallDns –DomainName $domname  -DatabasePath "F:\NTDS" -SysvolPath "F:\SYSVOL" -LogPath "F:\Logs"

计算机将重新启动。

## 配置第二个域控制器

使用本地管理员帐户的凭据登录到第二个域控制器计算机。有关说明，请参阅[使用远程桌面连接过程登录到虚拟机](#logon)。

接下来，你需要将额外的数据磁盘添加到第二个域控制器中。请参阅[初始化空磁盘过程](#datadisk)。

接下来，测试第二个域控制器与你的组织网络上的位置的连接。请参阅[测试连接过程](#testconn)。使用此过程可确保 DNS 名称解析正常工作（即，已正确为虚拟机配置了本地 DNS 服务器）并且可以从跨界虚拟网络收发数据包。

接下来，从第二个域控制器上的 Windows PowerShell 命令提示符，运行以下命令：

	$domname="<DNS domain name of the domain for which this computer will be a domain controller>"
	Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
	Install-ADDSDomainController -InstallDns –DomainName $domname  -DatabasePath "F:\NTDS" -SysvolPath "F:\SYSVOL" -LogPath "F:\Logs"

计算机将重新启动。

## 配置 SharePoint 场帐户和权限

SharePoint 场将需要以下用户帐户：

- sp\_farm：用于管理 SharePoint 场的用户帐户。
- sp\_farm\_db：对 SQL Server 实例拥有 sysadmin 权限的用户帐户。
- sp\_install：具有安装角色和功能所需的域管理权限的用户帐户。
- sqlservice：SQL Server 实例可以以其身份运行的用户帐户。

接下来，使用域控制器所属的域的域管理员帐户登录到任何计算机，打开管理员级别的 Windows PowerShell 命令提示符，并*一次一个地*运行以下命令：

	New-ADUser -SamAccountName sp_farm -AccountPassword (read-host "Set user password" -assecurestring) -name "sp_farm" -enabled $true -PasswordNeverExpires $true -ChangePasswordAtLogon $false

	New-ADUser -SamAccountName sp_farm_db -AccountPassword (read-host "Set user password" -assecurestring) -name "sp_farm_db" -enabled $true -PasswordNeverExpires $true -ChangePasswordAtLogon $false

	New-ADUser -SamAccountName sp_install -AccountPassword (read-host "Set user password" -assecurestring) -name "sp_install" -enabled $true -PasswordNeverExpires $true -ChangePasswordAtLogon $false

	New-	ADUser -SamAccountName sqlservice -AccountPassword (read-host "Set user password" -assecurestring) -name "sqlservice" -enabled $true -PasswordNeverExpires $true -ChangePasswordAtLogon $false

对于每个命令，系统会提示你输入密码。记录这些帐户名和密码，并将其存储在安全位置。

接下来，执行以下步骤以将其他帐户属性添加到新的用户帐户。

1.	在“开始”屏幕上，键入**“Active Directory 用户”**，然后单击**“Active Directory 用户和计算机”**。
2.	在树窗格中，打开你的域，然后单击**“用户”**。
3.	在内容窗格中，右键单击 **sp\_install**，然后单击**“添加到组”**。
4.	在**“选择组”**对话框中，键入**“域管理员”**，然后单击两次**“确定”**。
5.	在该对话框中，单击**“查看”并单击“高级功能”**。使用该选项可以查看 Active Directory 对象的属性窗口中所有隐藏的容器和隐藏选项卡。
6.	右键单击你的域名，然后单击**“属性”**。
7.	在**“属性”**对话框中，单击**“安全性”**选项卡，然后单击**“高级”**按钮。
8.	在**“<YourDomain> 的高级安全设置”**窗口中，单击**“添加”**。
9.	在“<YourDomain> 的权限条目”窗口中，单击“选择主体”。
10.	在文本框中，键入 **<YourDomain>\\sp\_install**，然后单击**“确定”**。
11.	对于**“创建计算机对象”**选择**“允许”**，然后单击三次**“确定”**。

接下来，更新你的虚拟网络的 DNS 服务器，使 Azure 可以为虚拟机分配两个新域控制器的 IP 地址用作其 DNS 服务器。请注意，此过程使用表 V（用于虚拟网络设置）中的值。

1.	在 Azure 门户的左窗格中，单击“网络”，然后单击你的虚拟网络的名称（表 V - 项目 1 -“值”列）。
2.	单击**“配置”**。
3.	在**“DNS 服务器”**中，删除位于你的本地网络上的 DNS 服务器所对应的条目。
4.	在**“DNS 服务器”**中，添加包含下列两个表项目的友好名称和 IP 地址的两个条目：
 - 表V - 项目 6 -“值”列
 - 表V - 项目 7 -“值”列
5.	在底部的命令栏中，单击**“保存”**。
6.	在 Azure 门户的左窗格中，单击“虚拟机”，然后单击第一个域控制器名称旁边的“状态”列。
7.	在命令栏中，单击**“重新启动”**。
8.	启动第一个域控制器时，单击第二个域控制器的名称旁边的**“状态”**列。
9.	在命令栏中，单击**“重新启动”**。等到第二个域控制器已启动。

请注意，你应重新启动两个域控制器，这样将不会为它们配置本地 DNS 服务器作为 DNS 服务器。因为它们本身就是 DNS 服务器，将它们提升为域控制器时，已自动为其配置本地 DNS 服务器作为 DNS 转发器。

接下来，需要创建一个 Active Directory 复制站点，以确保 Azure 虚拟网络中的服务器使用本地域控制器。使用 sp\_install 帐户登录到主域控制器，并在管理员级别的 Windows PowerShell 命令提示符下运行以下命令：

	$vnet="<Table V – Item 1 – Value column>"
	$vnetSpace="<Table V – Item 5 – Value column>"
	New-ADReplicationSite -Name $vnet
	New-ADReplicationSubnet –Name $vnetSpace –Site $vnet

下图使用占位符计算机名称显示成功完成此阶段后生成的配置。

![](./media/virtual-machines-workload-intranet-sharepoint-phase2/workload-spsqlao_02.png)

## 后续步骤

若要继续配置此工作负荷，请转到[阶段 3：配置 SQL Server 基础结构](/documentation/articles/virtual-machines-workload-intranet-sharepoint-phase3)。

## 其他资源

[在 Azure 中通过 SQL Server AlwaysOn 可用性组部署 SharePoint](/documentation/articles/virtual-machines-workload-intranet-sharepoint-overview)

[Azure 基础结构服务中托管的 SharePoint 场](/documentation/articles/virtual-machines-sharepoint-infrastructure-services)

[具有 SQL Server AlwaysOn 的 SharePoint 信息图](http://go.microsoft.com/fwlink/?LinkId=394788)

[适用于 SharePoint 2013 的 Windows Azure 体系结构](https://technet.microsoft.com/zh-cn/library/dn635309.aspx)

[Azure 基础结构服务实施准则](/documentation/articles/virtual-machines-infrastructure-services-implementation-guidelines)

[Azure 基础结构服务工作负荷：高可用性业务线应用程序](/documentation/articles/virtual-machines-workload-high-availability-LOB-application)

<!---HONumber=70-->