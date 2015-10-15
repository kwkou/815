<properties
	pageTitle="SharePoint Intranet 场工作负荷阶段 4：配置 SharePoint Server"
	description="在部署仅限 Intranet 的 SharePoint 2013 场的这个第四阶段，你将创建 SharePoint Server 虚拟机和新的 SharePoint 场。"
	documentationCenter=""
	services="virtual-machines"
	authors="JoeDavies-MSFT"
	manager="timlt"
	editor=""
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines"
	ms.date="07/22/2015"
	wacn.date="09/18/2015"/>

# SharePoint Intranet 场工作负荷阶段 4：配置 SharePoint Server

在部署仅限 Intranet 的 SharePoint 2013 场（在 Azure 基础结构服务中通过 SQL Server AlwaysOn 可用性组进行）的这个阶段，你将构建出 SharePoint 场的应用程序层和 Web 层，并使用 SharePoint 配置向导创建场。

你必须在转到[阶段 5](/documentation/articles/virtual-machines-workload-intranet-sharepoint-phase5) 之前完成此阶段。请参阅[在 Azure 中通过 SQL Server AlwaysOn 可用性组部署 SharePoint](/documentation/articles/virtual-machines-workload-intranet-sharepoint-overview) 以了解所有阶段的相关信息。

## 在 Azure 中创建 SharePoint Server 虚拟机

有四个 SharePoint Server 虚拟机。两个 SharePoint Server 虚拟机用于前端 Web 服务器，另两个用于管理和托管 SharePoint 应用程序。每个层中的两个 SharePoint Server 可提供高可用性。

使用以下 Azure PowerShell 命令块为四个 SharePoint Server 创建虚拟机。为变量指定值，并删除 < and > 字符。请注意，此 Azure PowerShell 命令集使用下表中的值：

- 表 M，用于虚拟机
- 表 V，用于虚拟网络设置
- 表 S，用于子网
- 表 A，用于可用性集
- 表 C，用于云服务

回想一下，你在[阶段 2：配置域控制器](/documentation/articles/virtual-machines-workload-intranet-sharepoint-phase2)中定义了表 M，在[阶段 1：配置 Azure](/documentation/articles/virtual-machines-workload-intranet-sharepoint-phase1) 中定义了表 V、表 S、表 A 和表 C。

如果已提供所有适当的值，请在 Azure PowerShell 命令提示符下运行生成的块。

	# Create the first SharePoint application server
	$vmName="<Table M – Item 6 - Virtual machine name column>"
	$vmSize="<Table M – Item 6 - Minimum size column, specify one: Small, Medium, Large, ExtraLarge, A5, A6, A7, A8, A9>"
	$availSet="<Table A – Item 3 – Availability set name column>"
	$image= Get-AzureVMImage | where { $_.Label -eq "SharePoint Server 2013 Trial" } | sort PublishedDate -Descending | select -ExpandProperty ImageName -First 1
	$vm1=New-AzureVMConfig -Name $vmName -InstanceSize $vmSize -ImageName $image -AvailabilitySetName $availSet

	$cred1=Get-Credential –Message "Type the name and password of the local administrator account for the first SharePoint application server."
	$cred2=Get-Credential –Message "Now type the name and password of an account that has permissions to add this virtual machine to the domain."
	$ADDomainName="<name of the AD domain that the server is joining (example CORP)>"
	$domainDNS="<FQDN of the AD domain that the server is joining (example corp.contoso.com)>"
	$vm1 | Add-AzureProvisioningConfig -AdminUsername $cred1.GetNetworkCredential().Username -Password $cred1.GetNetworkCredential().Password -WindowsDomain -Domain $ADDomainName -DomainUserName $cred2.GetNetworkCredential().Username -DomainPassword $cred2.GetNetworkCredential().Password -JoinDomain $domainDNS

	$subnetName="<Table 6 – Item 1 – Subnet name column>"
	$vm1 | Set-AzureSubnet -SubnetNames $subnetName

	$serviceName="<Table C – Item 3 – Cloud service name column>"
	$vnetName="<Table V – Item 1 – Value column>"
	New-AzureVM –ServiceName $serviceName -VMs $vm1 -VNetName $vnetName

	# Create the second SharePoint application server
	$vmName="<Table M – Item 7 - Virtual machine name column>"
	$vmSize="<Table M – Item 7 - Minimum size column, specify one: Small, Medium, Large, ExtraLarge, A5, A6, A7, A8, A9>"
	$vm1=New-AzureVMConfig -Name $vmName -InstanceSize $vmSize -ImageName $image -AvailabilitySetName $availSet

	$cred1=Get-Credential –Message "Type the name and password of the local administrator account for the second SharePoint application server."
	$vm1 | Add-AzureProvisioningConfig -AdminUsername $cred1.GetNetworkCredential().Username -Password $cred1.GetNetworkCredential().Password -WindowsDomain -Domain $ADDomainName -DomainUserName $cred2.GetNetworkCredential().Username -DomainPassword $cred2.GetNetworkCredential().Password -JoinDomain $domainDNS

	$vm1 | Set-AzureSubnet -SubnetNames $subnetName

	New-AzureVM –ServiceName $serviceName -VMs $vm1 -VNetName $vnetName

	# Create the first SharePoint web server
	$vmName="<Table M – Item 8 - Virtual machine name column>"
	$vmSize="<Table M – Item 8 - Minimum size column, specify one: Small, Medium, Large, ExtraLarge, A5, A6, A7, A8, A9>"
	$availSet="<Table A – Item 4 – Availability set name column>"
	$vm1=New-AzureVMConfig -Name $vmName -InstanceSize $vmSize -ImageName $image -AvailabilitySetName $availSet

	$cred1=Get-Credential –Message "Type the name and password of the local administrator account for the first SharePoint web server."
	$vm1 | Add-AzureProvisioningConfig -AdminUsername $cred1.GetNetworkCredential().Username -Password $cred1.GetNetworkCredential().Password -WindowsDomain -Domain $ADDomainName -DomainUserName $cred2.GetNetworkCredential().Username -DomainPassword $cred2.GetNetworkCredential().Password -JoinDomain $domainDNS

	$vm1 | Set-AzureSubnet -SubnetNames $subnetName

	New-AzureVM –ServiceName $serviceName -VMs $vm1 -VNetName $vnetName

	# Create the second SharePoint web server
	$vmName="<Table M – Item 9 - Virtual machine name column>"
	$vmSize="<Table M – Item 9 - Minimum size column, specify one: Small, Medium, Large, ExtraLarge, A5, A6, A7, A8, A9>"
	$vm1=New-AzureVMConfig -Name $vmName -InstanceSize $vmSize -ImageName $image -AvailabilitySetName $availSet

	$cred1=Get-Credential –Message "Type the name and password of the local administrator account for the second SharePoint web server."
	$vm1 | Add-AzureProvisioningConfig -AdminUsername $cred1.GetNetworkCredential().Username -Password $cred1.GetNetworkCredential().Password -WindowsDomain -Domain $ADDomainName -DomainUserName $cred2.GetNetworkCredential().Username -DomainPassword $cred2.GetNetworkCredential().Password -JoinDomain $domainDNS

	$vm1 | Set-AzureSubnet -SubnetNames $subnetName

	New-AzureVM –ServiceName $serviceName -VMs $vm1 -VNetName $vnetName

利用[“使用远程桌面连接登录到虚拟机”过程](/documentation/articles/virtual-machines-workload-intranet-sharepoint-phase2#logon)四次（对每个 SharePoint Server 利用一次），使用 [Domain]\\sp\_farm\_db 帐户凭据登录。这些凭据是在[阶段 2：配置域控制器](/documentation/articles/virtual-machines-workload-intranet-sharepoint-phase2)中创建的。

使用[测试连接过程](/documentation/articles/virtual-machines-workload-intranet-sharepoint-phase2#testconn)四次（对每个 SharePoint Server 使用一次），测试与你的组织网络上的位置的连接。

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

## 配置内部负载平衡

可配置内部负载平衡，使到 SharePoint 场的客户端流量平均分布到两个前端 Web 服务器上。这需要你指定内部负载平衡实例，该实例由名称和它自己的 IP 地址（从子网地址空间中分配）组成。若要确定为内部负载平衡器选择的 IP 地址是否可用，请使用以下 Azure PowerShell 命令：

	$vnet="<Table V – Item 1 – Value column>"
	$testIP="<a chosen IP address from the subnet address space, Table S - Item 1 – Subnet address space column>"
	Test-AzureStaticVNetIP –VNetName $vnet –IPAddress $testIP

如果在显示的 **Test-AzureStaticVNetIP** 命令中 **IsAvailable** 字段是 **True**，则可以使用 IP 地址。

在本地计算机上的 Azure PowerShell 命令提示符下，运行以下一组命令：

	$serviceName="<Table C – Item 3 – Cloud service name column>"
	$ilb="<name of your internal load balancer instance>"
	$subnet="<Table S – Item 1 – Subnet name column>"
	$IP="<an available IP address for your ILB instance>"
	Add-AzureInternalLoadBalancer –ServiceName $serviceName -InternalLoadBalancerName $ilb –SubnetName $subnet –StaticVNetIPAddress $IP

	$prot="tcp"
	$locport=80
	$pubport=80
	# This example assumes unsecured HTTP traffic to the SharePoint farm.

	$epname="SPWeb1"
	$vmname="<Table M – Item 8 – Virtual machine name column>"
	Get-AzureVM –ServiceName $serviceName –Name $vmname | Add-AzureEndpoint -Name $epname -LBSetName $ilb -Protocol $prot -LocalPort $locport -PublicPort $pubport –DefaultProbe -InternalLoadBalancerName $ilb | Update-AzureVM

	$epname="SPWeb2"
	$vmname="<Table M – Item 9 – Virtual machine name column>"
	Get-AzureVM –ServiceName $serviceName –Name $vmname | Add-AzureEndpoint -Name $epname -LBSetName $ilb -Protocol $prot -LocalPort $locport -PublicPort $pubport –DefaultProbe -InternalLoadBalancerName $ilb | Update-AzureVM

接下来，将 DNS 地址记录添加到你组织的 DNS 基础结构中，以便将 SharePoint 场的完全限定域名（例如 sp.corp.contoso.com）解析为分配给内部负载平衡器实例的 IP 地址（前面的 Azure PowerShell 命令块中 **$IP** 的值）。

这是成功完成此阶段后生成的配置。

![](./media/virtual-machines-workload-intranet-sharepoint-phase4/workload-spsqlao_04.png)

## 后续步骤

若要继续配置此工作负荷，请转到[阶段 5：创建可用性组并添加 SharePoint 数据库](/documentation/articles/virtual-machines-workload-intranet-sharepoint-phase5)。

## 其他资源

[在 Azure 中通过 SQL Server AlwaysOn 可用性组部署 SharePoint](/documentation/articles/virtual-machines-workload-intranet-sharepoint-overview)

[Azure 基础结构服务中托管的 SharePoint 场](/documentation/articles/virtual-machines-sharepoint-infrastructure-services)

[具有 SQL Server AlwaysOn 的 SharePoint 信息图](http://go.microsoft.com/fwlink/?LinkId=394788)

[适用于 SharePoint 2013 的 Windows Azure 体系结构](https://technet.microsoft.com/zh-cn/library/dn635309.aspx)

[Azure 基础结构服务实施准则](/documentation/articles/virtual-machines-infrastructure-services-implementation-guidelines)

[Azure 基础结构服务工作负荷：高可用性业务线应用程序](/documentation/articles/virtual-machines-workload-high-availability-LOB-application)

<!---HONumber=70-->