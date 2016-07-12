<properties 
	pageTitle="在 Azure 中配置 AlwaysOn 可用性组 (PowerShell)"
	description="使用 PowerShell 在 Azure 中创建 AlwaysOn 可用性组。"
	services="virtual-machines"
	documentationCenter="na"
	authors="rothja"
	manager="jeffreyg"
	editor="monicar" />
<tags 
	ms.service="virtual-machines-windows"
	ms.date="05/04/2016"
	wacn.date="06/29/2016" />

# 在 Azure 中配置 AlwaysOn 可用性组 (PowerShell)

> [AZURE.SELECTOR]
- [Resource Manager: 手动](/documentation/articles/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/)
- [经典: UI](/documentation/articles/virtual-machines-windows-classic-portal-sql-alwayson-availability-groups/)
- [经典: PowerShell](/documentation/articles/virtual-machines-windows-classic-ps-sql-alwayson-availability-groups/)

>[AZURE.NOTE]有关同一方案的基于 GUI 的教程，请参阅[在 Azure 中配置 AlwaysOn 可用性组 (GUI)](/documentation/articles/virtual-machines-windows-classic-portal-sql-alwayson-availability-groups/)。

Azure 虚拟机 (VM) 可帮助数据库管理员降低高可用性 SQL Server 系统的成本。本教程介绍如何在 Azure 环境中使用端到端 SQL Server AlwaysOn 实施可用性组。完成本教程后，Azure 中的 SQL Server AlwaysOn 解决方案将包括以下要素：

- 一个包含多个子网（包括前端子网和后端子网）的虚拟网络

- 包含 Active Directory (AD) 域的域控制器

- 部署到后端子网并加入 AD 域的两个 SQL Server VM

- 具有节点多数仲裁模型的 3 节点 WSFC 群集

- 具有可用性数据库的两个同步提交副本的可用性组

之所以选择此方案是因为其简易性，而非其成本效益或 Azure 上的其他功能。例如，你可最大程度减少 2 副本可用性组的虚拟机数目，以便通过将域控制器作为 2 节点 WSFC 群集中的仲裁文件共享见证服务器来节省 Azure 中的计算时间。通过此方法，上述配置中的 VM 数目可以减少一个。

本教程介绍设置上述解决方案所需的步骤，但不详细阐述每一步的细节。因此，我们没有列出 GUI 配置步骤，而是借助 PowerShell 脚本带你迅速完成每个步骤。假设如下：

- 你已有一个用于虚拟机订阅的 Azure 帐户。

- 已安装 [Azure PowerShell cmdlet](/documentation/articles/powershell-install-configure/)。

- 你已深入了解本地解决方案的 AlwaysOn 可用性组。有关详细信息，请参阅 [AlwaysOn 可用性组 (SQL Server)](https://msdn.microsoft.com/zh-cn/library/hh510230.aspx)。

## 连接到你的 Azure 订阅并创建虚拟网络

1. 在本地计算机上的 PowerShell 窗口中，导入 Azure 模块，将发布设置文件下载到计算机，然后通过导入所下载的发布设置将 PowerShell 会话连接至你的 Azure 订阅。

		Import-Module "C:\Program Files (x86)\Microsoft SDKs\Azure\PowerShell\Azure\Azure.psd1"
		Get-AzurePublishSettingsFile -Environment AzureChinaCloud
		Import-AzurePublishSettingsFile -Environment AzureChinaCloud <publishsettingsfilepath> 

	**Get-AzurePublishgSettingsFile** 命令自动生成管理证书，Azure 将其下载到你的计算机。浏览器将自动打开，提示输入 Azure 订阅的 Microsoft 帐户凭据。所下载的 **.publishsettings** 文件包含管理 Azure 订阅所需的一切信息。将该文件保存到本地目录后，使用 **Import-AzurePublishSettingsFile -Environment AzureChinaCloud** 命令将其导入。
	
	>[AZURE.NOTE]publishsettings 文件中含有你的凭据（未编码），这些凭据用于管理你的 Azure 订阅和服务。确保此文件安全的最佳做法是，将其暂时存储在您的源目录的外部（例如存储在 Libraries\\Documents 文件夹中），然后在完成导入后将其删除。恶意用户获得 publishsettings 文件的访问权限后，可编辑、创建和删除你的 Azure 服务。

1. 定义将用于创建云 IT 基础结构的一系列变量。

		$location = "China North"
		$affinityGroupName = "ContosoAG"
		$affinityGroupDescription = "Contoso SQL HADR Affinity Group"
		$affinityGroupLabel = "IaaS BI Affinity Group"
		$networkConfigPath = "C:\scripts\Network.netcfg"
		$virtualNetworkName = "ContosoNET"
		$storageAccountName = "<uniquestorageaccountname>"
		$storageAccountLabel = "Contoso SQL HADR Storage Account"
		$storageAccountContainer = "https://" + $storageAccountName + ".blob.core.chinacloudapi.cn/vhds/"
		$winImageName = (Get-AzureVMImage | where {$_.Label -like "Windows Server 2008 R2 SP1*"} | sort PublishedDate -Descending)[0].ImageName
		$sqlImageName = (Get-AzureVMImage | where {$_.Label -like "SQL Server 2012 SP1 Enterprise*"} | sort PublishedDate -Descending)[0].ImageName
		$dcServerName = "ContosoDC"
		$dcServiceName = "<uniqueservicename>" 
		$availabilitySetName = "SQLHADR"
		$vmAdminUser = "AzureAdmin" 
		$vmAdminPassword = "Contoso!000" 
		$workingDir = "c:\scripts"

	请注意以下几点，以确保后面的命令可以成功执行：
	
	- 变量 **$storageAccountName** 和 **$dcServiceName** 分别用于标识 Internet 上的云存储帐户和云服务器，因此必须唯一。
	
	- 为变量 **$affinityGroupName** 和 **$virtualNetworkName** 指定的名称（在稍后将使用的虚拟网络配置文档中配置）。
	
	- **$sqlImageName** 指定包含 SQL Server 2012 Service Pack 1 Enterprise Edition 的 VM 映像的更新名称。
	
	- 为简单起见，在整个教程中使用同一密码 **Contoso!000**。

1. 创建地缘组

		New-AzureAffinityGroup `
			-Name $affinityGroupName `
			-Location $location `
			-Description $affinityGroupDescription `
			-Label $affinityGroupLabel

1. 通过导入配置文件创建虚拟网络。

		Set-AzureVNetConfig `
			-ConfigurationPath $networkConfigPath

	配置文件包含以下 XML 文档。简言之，它在名为 **ContosoAG** 的地缘组中指定名为 **ContosoNET** 的虚拟网络，其地址空间为 **10.10.0.0/16**，有两个子网 **10.10.1.0/24** 和 **10.10.2.0/24**，分别是前端子网和后端子网。前端子网是放置 Microsoft SharePoint 等客户端应用程序的位置，后端子网是放置 SQL Server VM 的位置。如果之前更改过 **$affinityGroupName** 和 **$virtualNetworkName** 变量，则还必须更改以下相应名称。

		<NetworkConfiguration xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns="http://schemas.microsoft.com/ServiceHosting/2011/07/NetworkConfiguration">
		  <VirtualNetworkConfiguration>
		    <Dns />
		    <VirtualNetworkSites>
		      <VirtualNetworkSite name="ContosoNET" AffinityGroup="ContosoAG">
		        <AddressSpace>
		          <AddressPrefix>10.10.0.0/16</AddressPrefix>
		        </AddressSpace>
		        <Subnets>
		          <Subnet name="Front">
		            <AddressPrefix>10.10.1.0/24</AddressPrefix>
		          </Subnet>
		          <Subnet name="Back">
		            <AddressPrefix>10.10.2.0/24</AddressPrefix>
		          </Subnet>
		        </Subnets>
		      </VirtualNetworkSite>
		    </VirtualNetworkSites>
		  </VirtualNetworkConfiguration>
		</NetworkConfiguration>

1. 创建与所创建的地缘组关联的存储帐户，然后将其设置为订阅中的当前存储帐户。

		New-AzureStorageAccount `
			-StorageAccountName $storageAccountName `
			-Label $storageAccountLabel `
			-AffinityGroup $affinityGroupName 
		Set-AzureSubscription -Environment AzureChinaCloud `
			-SubscriptionName (Get-AzureSubscription).SubscriptionName `
			-CurrentStorageAccount $storageAccountName

1. 在新的云服务和可用性集中创建 DC 服务器。

		New-AzureVMConfig `
			-Name $dcServerName `
			-InstanceSize Medium `
			-ImageName $winImageName `
			-MediaLocation "$storageAccountContainer$dcServerName.vhd" `
			-DiskLabel "OS" | 
			Add-AzureProvisioningConfig `
				-Windows `
				-DisableAutomaticUpdates `
				-AdminUserName $vmAdminUser `
				-Password $vmAdminPassword |
				New-AzureVM `
					-ServiceName $dcServiceName `
					–AffinityGroup $affinityGroupName `
					-VNetName $virtualNetworkName

	这一系列的管接命令执行以下操作：
	
	- **New-AzureVMConfig** 创建 VM 配置。
	
	- **Add-AzureProvisioningConfig** 提供独立 Windows 服务器的配置参数。
	
	- **Add-AzureDataDisk** 添加将用于保存 Active Directory 数据的数据磁盘，缓存选项设置为 None。
	
	- **New-AzureVM** 创建新的云服务，并在新的云服务中创建新的 Azure VM。

1. 请等待新 VM 预配完毕，然后将远程桌面文件下载到工作目录中。由于新 Azure VM 的预配要用很长时间，while 循环会持续轮询新 VM，直到其使用就绪。

		$VMStatus = Get-AzureVM -ServiceName $dcServiceName -Name $dcServerName
		
		While ($VMStatus.InstanceStatus -ne "ReadyRole")
		{
		    write-host "Waiting for " $VMStatus.Name "... Current Status = " $VMStatus.InstanceStatus
		    Start-Sleep -Seconds 15
		    $VMStatus = Get-AzureVM -ServiceName $dcServiceName -Name $dcServerName
		}
		
		Get-AzureRemoteDesktopFile `
		    -ServiceName $dcServiceName `
		    -Name $dcServerName `
		    -LocalPath "$workingDir$dcServerName.rdp"

现在已成功预配 DC 服务器。接下来，请在 DC 服务器上配置 Active Directory 域。在本地计算机上使 PowerShell 窗口保持打开。稍后将再次使用它来创建两个 SQL Server VM。

## 配置域控制器

1. 通过启动远程桌面文件连接到 DC 服务器。使用创建新 VM 时指定的计算机管理员用户名 AzureAdmin 和密码 **Contoso!000**。

1. 在管理员模式下打开 PowerShell 窗口。

1. 运行以下 **DCPROMO.EXE** 命令来设置 **corp.contoso.com** 域，数据目录位于驱动器 M 上。

		dcpromo.exe `
			/unattend `
			/ReplicaOrNewDomain:Domain `
			/NewDomain:Forest `
			/NewDomainDNSName:corp.contoso.com `
			/ForestLevel:4 `
			/DomainNetbiosName:CORP `
			/DomainLevel:4 `
			/InstallDNS:Yes `
			/ConfirmGc:Yes `
			/CreateDNSDelegation:No `
			/DatabasePath:"C:\Windows\NTDS" `
			/LogPath:"C:\Windows\NTDS" `
			/SYSVOLPath:"C:\Windows\SYSVOL" `
			/SafeModeAdminPassword:"Contoso!000"

	命令完成后，VM 会自动重新启动。

1. 通过启动远程桌面文件再次连接到 DC 服务器。此时，以 **CORP\\Administrator** 身份登录。

1. 在管理员模式下打开 PowerShell 窗口，使用以下命令导入 Active Directory PowerShell 模块：

		Import-Module ActiveDirectory

1. 运行以下命令将三个用户添加到域。

		$pwd = ConvertTo-SecureString "Contoso!000" -AsPlainText -Force
		New-ADUser `
			-Name 'Install' `
			-AccountPassword  $pwd `
			-PasswordNeverExpires $true `
			-ChangePasswordAtLogon $false `
			-Enabled $true
		New-ADUser `
			-Name 'SQLSvc1' `
			-AccountPassword  $pwd `
			-PasswordNeverExpires $true `
			-ChangePasswordAtLogon $false `
			-Enabled $true
		New-ADUser `
			-Name 'SQLSvc2' `
			-AccountPassword  $pwd `
			-PasswordNeverExpires $true `
			-ChangePasswordAtLogon $false `
			-Enabled $true

	**CORP\\Install** 用于全面配置 SQL Server 服务实例、WSFC 群集和可用性组。**CORP\\SQLSvc1** 和 **CORP\\SQLSvc2** 用作两个 SQL Server VM 的 SQL Server 服务帐户。

1. 接下来，运行以下命令为 **CORP\\Install** 提供在域中创建计算机对象的权限。

		Cd ad:
		$sid = new-object System.Security.Principal.SecurityIdentifier (Get-ADUser "Install").SID
		$guid = new-object Guid bf967a86-0de6-11d0-a285-00aa003049e2
		$ace1 = new-object System.DirectoryServices.ActiveDirectoryAccessRule $sid,"CreateChild","Allow",$guid,"All"
		$corp = Get-ADObject -Identity "DC=corp,DC=contoso,DC=com"
		$acl = Get-Acl $corp
		$acl.AddAccessRule($ace1)
		Set-Acl -Path "DC=corp,DC=contoso,DC=com" -AclObject $acl 

	上面指定的 GUID 是计算机对象类型的 GUID。**CORP\\Install** 帐户需要有“读取所有属性”和“创建计算对象”权限才能为 WSFC 群集创建 Active Direct 对象。默认情况下，已经将“读取所有属性”权限授予 CORP\\Install，因此无需显式授予该权限。有关创建 WSFC 群集所需权限的详细信息，请参阅[故障转移群集循序渐进指南：在 Active Directory 中配置帐户](https://technet.microsoft.com/zh-cn/library/cc731002%28v=WS.10%29.aspx)。

	现在你已完成 Active Directory 和用户对象的配置，接下来，你将创建两个 SQL Server VM 并将其加入到此域中。

## 创建 SQL Server VM

1. 继续使用在本地计算机上已打开的 PowerShell 窗口。定义以下附加变量：

		$domainName= "corp"
		$FQDN = "corp.contoso.com"
		$subnetName = "Back"
		$sqlServiceName = "<uniqueservicename>"
		$quorumServerName = "ContosoQuorum"
		$sql1ServerName = "ContosoSQL1"
		$sql2ServerName = "ContosoSQL2"
		$availabilitySetName = "SQLHADR"
		$dataDiskSize = 100
		$dnsSettings = New-AzureDns -Name "ContosoBackDNS" -IPAddress "10.10.0.4"

	通常将 IP 地址 **10.10.0.4** 分配给在 Azure 虚拟网络的 **10.10.0.0/16** 子网中创建的第一个 VM。应通过运行 **IPCONFIG** 验证这是不是 DC 服务器的地址。

1. 运行以下管接命令在 WSFC 群集中创建名为 **ContosoQuorum** 的第一个 VM：

		New-AzureVMConfig `
			-Name $quorumServerName `
			-InstanceSize Medium `
			-ImageName $winImageName `
			-MediaLocation "$storageAccountContainer$quorumServerName.vhd" `
			-AvailabilitySetName $availabilitySetName `
			-DiskLabel "OS" | 
			Add-AzureProvisioningConfig `
				-WindowsDomain `
				-AdminUserName $vmAdminUser `
				-Password $vmAdminPassword `
				-DisableAutomaticUpdates `
				-Domain $domainName `
				-JoinDomain $FQDN `
				-DomainUserName $vmAdminUser `
				-DomainPassword $vmAdminPassword |
				Set-AzureSubnet `
					-SubnetNames $subnetName |
					New-AzureVM `
						-ServiceName $sqlServiceName `
						–AffinityGroup $affinityGroupName `
						-VNetName $virtualNetworkName `
						-DnsSettings $dnsSettings

	注意与上述命令相关的以下几点：
	
	- **New-AzureVMConfig** 创建具有所需可用性集名称的 VM 配置。后续 VM 将以该可用性集名称创建，因此会加入同一可用性集中。
	
	- **Add-AzureProvisioningConfig** 将 VM 加入创建的 Active Directory 域中。
	
	- **Set-AzureSubnet** 将 VM 放入 Back 子网。
	
	- **New-AzureVM** 创建新的云服务，并在新的云服务中创建新的 Azure VM。**DnsSettings** 参数指定新云服务中的服务器的 DNS 服务器具有 IP 地址 **10.10.0.4**，这是 DC 服务器的 IP 地址。需要该参数来启用云服务中的新 VM 才能成功加入 Active Directory 域。如果没有该参数，预配 VM 后必须在 VM 中手动设置 IPv4 设置才能将 DC 服务器作为主 DNS 服务器，然后 VM 才能加入 Active Directory 域。

1. 运行以下管接命令来创建名为 **ContosoSQL1** 和 **ContosoSQL2** 的 SQL Server VM。

		# Create ContosoSQL1...
		New-AzureVMConfig `
		    -Name $sql1ServerName `
		    -InstanceSize Large `
		    -ImageName $sqlImageName `
		    -MediaLocation "$storageAccountContainer$sql1ServerName.vhd" `
		    -AvailabilitySetName $availabilitySetName `
		    -HostCaching "ReadOnly" `
		    -DiskLabel "OS" | 
		    Add-AzureProvisioningConfig `
		        -WindowsDomain `
		        -AdminUserName $vmAdminUser `
		        -Password $vmAdminPassword `
		        -DisableAutomaticUpdates `
		        -Domain $domainName `
		        -JoinDomain $FQDN `
		        -DomainUserName $vmAdminUser `
		        -DomainPassword $vmAdminPassword |
		        Set-AzureSubnet `
		            -SubnetNames $subnetName |
		            Add-AzureEndpoint `
		                -Name "SQL" `
		                -Protocol "tcp" `
		                -PublicPort 1 `
		                -LocalPort 1433 | 
		                New-AzureVM `
		                    -ServiceName $sqlServiceName
		
		# Create ContosoSQL2...
		New-AzureVMConfig `
		    -Name $sql2ServerName `
		    -InstanceSize Large `
		    -ImageName $sqlImageName `
		    -MediaLocation "$storageAccountContainer$sql2ServerName.vhd" `
		    -AvailabilitySetName $availabilitySetName `
		    -HostCaching "ReadOnly" `
		    -DiskLabel "OS" | 
		    Add-AzureProvisioningConfig `
		        -WindowsDomain `
		        -AdminUserName $vmAdminUser `
		        -Password $vmAdminPassword `
		        -DisableAutomaticUpdates `
		        -Domain $domainName `
		        -JoinDomain $FQDN `
		        -DomainUserName $vmAdminUser `
		        -DomainPassword $vmAdminPassword |
		        Set-AzureSubnet `
		            -SubnetNames $subnetName |
		            Add-AzureEndpoint `
		                -Name "SQL" `
		                -Protocol "tcp" `
		                -PublicPort 2 `
		                -LocalPort 1433 | 
		                New-AzureVM `
		                    -ServiceName $sqlServiceName

	注意与上述命令相关的以下几点：

	- **New-AzureVMConfig** 使用与 DC 服务器相同的可用性集名称，并使用虚拟机库中的 SQL Server 2012 Service Pack 1 Enterprise Edition 映像。它还将操作系统磁盘设置为只读缓存（无写缓存）。建议将这些数据库文件迁移到一个附加到 VM 的独立数据磁盘中，并将其配置为无读或写缓存。不过，次优建议是移除操作系统磁盘上的写缓存，因为无法移除操作系统磁盘上的读缓存。
	
	- **Add-AzureProvisioningConfig** 将 VM 加入创建的 Active Directory 域中。
	
	- **Set-AzureSubnet** 将 VM 放入 Back 子网。
	
	- **Add-AzureEndpoint** 添加访问端点，以便客户端应用程序能够在 Internet 上访问这些 SQL Server 服务实例。为 ContosoSQL1 和 ContosoSQL2 提供了不同端口。
	
	- **New-AzureVM** 在与 ContosoQuorum 相同的云服务中创建新的 SQL Server VM。如果这些 VM 需要在同一可用性集中，则必须将它们放置在同一云服务中。

1. 请等待每个 VM 预配完毕，然后将其远程桌面文件下载到工作目录中。for 循环会循环访问三个新 VM，并为每个 VM 执行上级大括号内的命令。

		Foreach ($VM in $VMs = Get-AzureVM -ServiceName $sqlServiceName)
		{
		    write-host "Waiting for " $VM.Name "..."
		
		    # Loop until the VM status is "ReadyRole"
		    While ($VM.InstanceStatus -ne "ReadyRole")
		    {
		        write-host "  Current Status = " $VM.InstanceStatus
		        Start-Sleep -Seconds 15
		        $VM = Get-AzureVM -ServiceName $VM.ServiceName -Name $VM.InstanceName
		    }
		
		    write-host "  Current Status = " $VM.InstanceStatus
		
		    # Download remote desktop file
		    Get-AzureRemoteDesktopFile -ServiceName $VM.ServiceName -Name $VM.InstanceName -LocalPath "$workingDir$($VM.InstanceName).rdp"
		}

	这些 SQL Server VM 现在已预配并正在运行，但它们是使用默认选项与 SQL Server 一同安装的。

## 初始化 WSFC 群集 VM

在本部分中，需要修改将在 WSFC 群集和 SQL Server 安装中使用的三个服务器。具体而言：

- （所有服务器）需要安装**故障转移群集**功能。

- （所有服务器）需要添加 **CORP\\Install** 作为计算机**管理员**。

- （仅限 ContosoSQL1 和 ContosoSQL2）需要添加 **CORP\\Install** 作为默认数据库中的 **sysadmin** 角色。

- （仅限 ContosoSQL1 和 ContosoSQL2）需要添加 NT **AUTHORITY\\System** 作为具有以下权限的登录名：

	- 更改任何可用性组
	
	- 连接 SQL
	
	- 查看服务器状态

- （仅限 ContosoSQL1 和 ContosoSQL2）在 SQL Server VM 上已启用了 **TCP** 协议。但是，仍需打开防火墙以便远程访问 SQL Server。

现在，准备启动。从 **ContosoQuorum** 开始，执行以下步骤：

1. 通过启动远程桌面文件连接到 **ContosoQuorum**。使用创建 VM 时指定的计算机管理员用户名 **AzureAdmin** 和密码 **Contoso!000**。

1. 验证计算机是否已成功加入 **corp.contoso.com**。

1. 等待 SQL Server 安装完成自动初始化任务，然后继续。

1. 在管理员模式下打开 PowerShell 窗口。

1. 安装 Windows 故障转移群集功能。

		Import-Module ServerManager
		Add-WindowsFeature Failover-Clustering

1. 添加 **CORP\\Install** 作为本地管理员。

		net localgroup administrators "CORP\Install" /Add

1. 从 ContosoQuorum 注销。现已完成此服务器的操作。

		logoff.exe

接下来，请初始化 **ContosoSQL1** 和 **ContosoSQL2**。执行以下步骤，对于这两个 SQL Server VM，这些步骤相同。

1. 通过启动远程桌面文件连接到这两个 SQL Server VM。使用创建 VM 时指定的计算机管理员用户名 **AzureAdmin** 和密码 **Contoso!000**。

1. 验证计算机是否已成功加入 **corp.contoso.com**。

1. 等待 SQL Server 安装完成自动初始化任务，然后继续。

1. 在管理员模式下打开 PowerShell 窗口。

1. 安装 Windows 故障转移群集功能。

		Import-Module ServerManager
		Add-WindowsFeature Failover-Clustering

1. 添加 **CORP\\Install** 作为本地管理员。

		net localgroup administrators "CORP\Install" /Add

1. 导入 SQL Server PowerShell 提供程序。

		Set-ExecutionPolicy -Execution RemoteSigned -Force
		Import-Module -Name "sqlps" -DisableNameChecking

1. 将 **CORP\\Install** 作为 sysadmin 角色添加到默认 SQL Server 实例

		net localgroup administrators "CORP\Install" /Add
		Invoke-SqlCmd -Query "EXEC sp_addsrvrolemember 'CORP\Install', 'sysadmin'" -ServerInstance "."

1. 添加 **NT AUTHORITY\\System** 作为具有上述三个权限的登录名。

		Invoke-SqlCmd -Query "CREATE LOGIN [NT AUTHORITY\SYSTEM] FROM WINDOWS" -ServerInstance "."
		Invoke-SqlCmd -Query "GRANT ALTER ANY AVAILABILITY GROUP TO [NT AUTHORITY\SYSTEM] AS SA" -ServerInstance "." 
		Invoke-SqlCmd -Query "GRANT CONNECT SQL TO [NT AUTHORITY\SYSTEM] AS SA" -ServerInstance "."
		Invoke-SqlCmd -Query "GRANT VIEW SERVER STATE TO [NT AUTHORITY\SYSTEM] AS SA" -ServerInstance "."

1. 打开防火墙以便远程访问 SQL Server。

		netsh advfirewall firewall add rule name='SQL Server (TCP-In)' program='C:\Program Files\Microsoft SQL Server\MSSQL11.MSSQLSERVER\MSSQL\Binn\sqlservr.exe' dir=in action=allow protocol=TCP

1. 从两个 VM 注销。

		logoff.exe

最后，准备配置可用性组。将使用 SQL Server PowerShell 提供程序来执行 **ContosoSQL1** 上的所有工作。

## 配置可用性组

1. 通过启动远程桌面文件再次连接到 **ContosoSQL1**。使用 **CORP\\Install**（而不是计算机帐户）登录。

1. 在管理员模式下打开 PowerShell 窗口。

1. 定义以下变量：

		$server1 = "ContosoSQL1"
		$server2 = "ContosoSQL2"
		$serverQuorum = "ContosoQuorum"
		$acct1 = "CORP\SQLSvc1"
		$acct2 = "CORP\SQLSvc2"
		$password = "Contoso!000"
		$clusterName = "Cluster1"
		$timeout = New-Object System.TimeSpan -ArgumentList 0, 0, 30
		$db = "MyDB1"
		$backupShare = "\\$server1\backup"
		$quorumShare = "\\$server1\quorum"
		$ag = "AG1"

1. 导入 SQL Server PowerShell 提供程序。

		Set-ExecutionPolicy RemoteSigned -Force
		Import-Module "sqlps" -DisableNameChecking

1. 将 ContosoSQL1 的 SQL Server 服务帐户更改为 CORP\\SQLSvc1。

		$wmi1 = new-object ("Microsoft.SqlServer.Management.Smo.Wmi.ManagedComputer") $server1
		$wmi1.services | where {$_.Type -eq 'SqlServer'} | foreach{$_.SetServiceAccount($acct1,$password)}
		$svc1 = Get-Service -ComputerName $server1 -Name 'MSSQLSERVER'
		$svc1.Stop()
		$svc1.WaitForStatus([System.ServiceProcess.ServiceControllerStatus]::Stopped,$timeout)
		$svc1.Start(); 
		$svc1.WaitForStatus([System.ServiceProcess.ServiceControllerStatus]::Running,$timeout)

1. 将 ContosoSQL2 的 SQL Server 服务帐户更改为 CORP\\SQLSvc2。

		$wmi2 = new-object ("Microsoft.SqlServer.Management.Smo.Wmi.ManagedComputer") $server2
		$wmi2.services | where {$_.Type -eq 'SqlServer'} | foreach{$_.SetServiceAccount($acct2,$password)}
		$svc2 = Get-Service -ComputerName $server2 -Name 'MSSQLSERVER'
		$svc2.Stop()
		$svc2.WaitForStatus([System.ServiceProcess.ServiceControllerStatus]::Stopped,$timeout)
		$svc2.Start(); 
		$svc2.WaitForStatus([System.ServiceProcess.ServiceControllerStatus]::Running,$timeout)

1. 从[为 Azure VM 中的 AlwaysOn 可用性组创建 WSFC 群集](http://gallery.technet.microsoft.com/scriptcenter/Create-WSFC-Cluster-for-7c207d3a)，将 **CreateAzureFailoverCluster.ps1** 下载到本地工作目录中。该脚本将帮助你创建一个正常运行的 WSFC 群集。有关 WSFC 如何与 Azure 网络交互的重要信息，请参阅 [Azure 虚拟机中 SQL Server 的高可用性和灾难恢复](https://msdn.microsoft.com/zh-cn/library/azure/jj870962.aspx)。

1. 切换至工作目录并使用下载的脚本创建 WSFC 群集。

		Set-ExecutionPolicy Unrestricted -Force
		.\CreateAzureFailoverCluster.ps1 -ClusterName "$clusterName" -ClusterNode "$server1","$server2","$serverQuorum"

1. 为 **ContosoSQL1** 和 **ContosoSQL2** 上的默认 SQL Server 实例启用 AlwaysOn 可用性组。

		Enable-SqlAlwaysOn `
		    -Path SQLSERVER:\SQL\$server1\Default `
		    -Force
		Enable-SqlAlwaysOn `
		    -Path SQLSERVER:\SQL\$server2\Default `
		    -NoServiceRestart
		$svc2.Stop()
		$svc2.WaitForStatus([System.ServiceProcess.ServiceControllerStatus]::Stopped,$timeout)
		$svc2.Start(); 
		$svc2.WaitForStatus([System.ServiceProcess.ServiceControllerStatus]::Running,$timeout)

1. 创建备份目录，并为 SQL Server 服务帐户授予权限。你将使用该目录在辅助副本上准备可用性数据库。

		$backup = "C:\backup"
		New-Item $backup -ItemType directory
		net share backup=$backup "/grant:$acct1,FULL" "/grant:$acct2,FULL"
		icacls.exe "$backup" /grant:r ("$acct1" + ":(OI)(CI)F") ("$acct2" + ":(OI)(CI)F")

1. 在 **ContosoSQL1** 上创建一个名为 **MyDB1** 的数据库，获取完整备份和日志备份，然后使用 **WITH NORECOVERY** 选项在 **ContosoSQL2** 上还原它们。

		Invoke-SqlCmd -Query "CREATE database $db"
		Backup-SqlDatabase -Database $db -BackupFile "$backupShare\db.bak" -ServerInstance $server1
		Backup-SqlDatabase -Database $db -BackupFile "$backupShare\db.log" -ServerInstance $server1 -BackupAction Log
		Restore-SqlDatabase -Database $db -BackupFile "$backupShare\db.bak" -ServerInstance $server2 -NoRecovery
		Restore-SqlDatabase -Database $db -BackupFile "$backupShare\db.log" -ServerInstance $server2 -RestoreAction Log -NoRecovery

1. 在 SQL Server VM 上创建可用性组终结点，并在这些终结点上设置适当的权限。

		$endpoint = 
		    New-SqlHadrEndpoint MyMirroringEndpoint `
		    -Port 5022 `
		    -Path "SQLSERVER:\SQL\$server1\Default"
		Set-SqlHadrEndpoint `
		    -InputObject $endpoint `
		    -State "Started"
		$endpoint = 
		    New-SqlHadrEndpoint MyMirroringEndpoint `
		    -Port 5022 `
		    -Path "SQLSERVER:\SQL\$server2\Default"
		Set-SqlHadrEndpoint `
		    -InputObject $endpoint `
		    -State "Started"
		
		Invoke-SqlCmd -Query "CREATE LOGIN [$acct2] FROM WINDOWS" -ServerInstance $server1
		Invoke-SqlCmd -Query "GRANT CONNECT ON ENDPOINT::[MyMirroringEndpoint] TO [$acct2]" -ServerInstance $server1
		Invoke-SqlCmd -Query "CREATE LOGIN [$acct1] FROM WINDOWS" -ServerInstance $server2
		Invoke-SqlCmd -Query "GRANT CONNECT ON ENDPOINT::[MyMirroringEndpoint] TO [$acct1]" -ServerInstance $server2

1. 创建可用性副本。

		$primaryReplica = 
		    New-SqlAvailabilityReplica `
		    -Name $server1 `
		    -EndpointURL "TCP://$server1.corp.contoso.com:5022" `
		    -AvailabilityMode "SynchronousCommit" `
		    -FailoverMode "Automatic" `
		    -Version 11 `
		    -AsTemplate
		$secondaryReplica = 
		    New-SqlAvailabilityReplica `
		    -Name $server2 `
		    -EndpointURL "TCP://$server2.corp.contoso.com:5022" `
		    -AvailabilityMode "SynchronousCommit" `
		    -FailoverMode "Automatic" `
		    -Version 11 `
		    -AsTemplate

1. 最后，创建可用性组，并将辅助副本加入该可用性组。

		New-SqlAvailabilityGroup `
		    -Name $ag `
		    -Path "SQLSERVER:\SQL\$server1\Default" `
		    -AvailabilityReplica @($primaryReplica,$secondaryReplica) `
		    -Database $db
		Join-SqlAvailabilityGroup `
		    -Path "SQLSERVER:\SQL\$server2\Default" `
		    -Name $ag
		Add-SqlAvailabilityDatabase `
		    -Path "SQLSERVER:\SQL\$server2\Default\AvailabilityGroups\$ag" `
		    -Database $db

## 后续步骤
现在，你已通过在 Azure 中创建可用性组，成功实施了 SQL Server AlwaysOn。若要为此可用性组配置侦听器，请参阅[在 Azure 中配置 AlwaysOn 可用性组的 ILB 侦听器](/documentation/articles/virtual-machines-windows-classic-ps-sql-int-listener/)。

有关在 Azure 中使用 SQL Server 的其他信息，请参阅 [Azure 虚拟机上的 SQL Server](/documentation/articles/virtual-machines-windows-sql-server-iaas-overview/)。

<!---HONumber=70-->