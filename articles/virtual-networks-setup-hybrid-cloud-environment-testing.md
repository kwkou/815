<properties 
	pageTitle="混合云测试环境 | Azure" 
	description="了解如何创建混合云环境，以便 IT 专业人员或开发人员使用简化的本地网络完成测试。" 
	services="virtual-network" 
	documentationCenter="" 
	authors="JoeDavies-MSFT" 
	manager="timlt" 
	editor=""
	tags="azure-service-management"/>

<tags
	ms.service="virtual-network"
	ms.date="03/28/2016"
	wacn.date="05/24/2016"/>

# 设置用于测试的混合云环境

> [AZURE.IMPORTANT]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型。Azure 建议大多数新部署使用资源管理器模型。
 

本主题将指导你逐步使用 Azure 创建混合云环境，以便进行测试。这是生成的配置。

![](./media/virtual-networks-setup-hybrid-cloud-environment-testing/CreateHybridCloudVNet_5.png)

这会通过你在 Internet 上的位置模拟真实的混合生产环境。它包括：

-  简化的本地网络（Corpnet 子网）。
-  在 Azure 中托管的跨界虚拟网络 (TestVNET)。
-  站点到站点 VPN 连接。
-  TestVNET 虚拟网络中的辅助域控制器。

此配置提供了基础和通用的起始点，你可以据此执行以下操作：

-  在混合云环境中开发和测试应用程序。
-  针对基于混合云的 IT 工作负载，创建一部分位于 Corpnet 子网上，一部分位于 TestVNET 虚拟网络中的计算机的测试配置。

设置此混合云测试环境需要完成以下五个主要阶段：

1.	在 Corpnet 子网上配置计算机。
2.	配置 RRAS1。
3.	创建跨界 Azure 虚拟网络。
4.	创建站点到站点 VPN 连接。
5.	配置 DC2。 

如果你还没有 Azure 订阅，可以通过[试用 Azure](/pricing/1rmb-trial/) 注册试用版。

>[AZURE.NOTE] Azure 中的虚拟机和虚拟网关在运行时会持续产生货币成本。此成本是针对你的试用版。若要在不使用的情况下降低运行此测试环境的成本，请参阅本主题中的[最大程度地降低此环境的持续使用成本](#costs)，了解详细信息。

此配置要求使用一个由最多四台计算机组成的测试子网，这些计算机使用公共 IP 地址直接连接到 Internet。如果没有这些资源，你也可以[设置用于测试的模拟混合云环境](/documentation/articles/virtual-networks-setup-simulated-hybrid-cloud-environment-testing/)。模拟混合云测试环境只需要 Azure 订阅。

## 阶段 1：在 Corpnet 子网上配置计算机

按照[测试实验室指南：Windows Server 2012 R2 的基本配置](http://www.microsoft.com/download/details.aspx?id=39638)的“配置 Corpnet 子网的步骤”一节中的说明，在名为 Corpnet 的子网上配置 DC1、APP1 和 CLIENT1 计算机。**此子网必须与组织网络隔离，因为它将通过 RRAS1 计算机直接连接到 Internet。**

接下来，使用 CORP\\User1 凭据登录到 DC1。若要配置 CORP 域，以便计算机和用户使用其本地域控制器进行身份验证，请从管理员级 Windows PowerShell 命令提示符运行这些命令。

	New-ADReplicationSite -Name "TestLab" 
	New-ADReplicationSite -Name "TestVNET"
	New-ADReplicationSubnet -Name "10.0.0.0/8" -Site "TestLab"
	New-ADReplicationSubnet -Name "192.168.0.0/16" -Site "TestVNET

这是你当前的配置。

![](./media/virtual-networks-setup-hybrid-cloud-environment-testing/CreateHybridCloudVNet_1.png)
 
## 阶段 2：配置 RRAS1

RRAS1 在 Corpnet 子网和 TestVNET 虚拟网络的计算机之间提供通信路由和 VPN 设备服务。RRAS1 必须安装两个网络适配器。

首先，在 RRAS1 上安装操作系统。

1.	启动 Windows Server 2012 R2 的安装。
2.	按照说明操作以完成安装过程，指定本地管理员帐户的强密码。使用本地管理员帐户登录。
3.	将 RRAS1 连接到具有 Internet 访问权限的网络，并运行 Windows 更新，为 Windows Server 2012 R2 安装最新的更新。
4.	将一个网络适配器连接到 Corpnet 子网，将另一个直接连接到 Internet。RRAS1 可以位于 Internet 防火墙后面，但不能在网络地址转换器 (NAT) 后面。

接下来，配置 RRAS1 的 TCP/IP 属性。你将需要一个公共的 IP 地址配置，包括 Internet 服务提供商 (ISP) 的地址、子网掩码（或前缀长度），以及默认网关和 DNS 服务器。

在 RRAS1 上的管理员级 Windows PowerShell 命令提示符处使用这些命令。在运行这些命令之前，请填写变量值并删除 < and > 字符。你可以从 **Get-NetAdapter** 命令的显示中获取网络适配器的当前名称。

	$corpnetAdapterName="<Name of the adapter attached to the Corpnet subnet>"
	$internetAdapterName="<Name of the adapter attached to the Internet>"
	[Ipaddress]$publicIP="<Your public IP address>"
	$publicIPpreflength=<Prefix length of your public IP address>
	[IPAddress]$publicDG="<Your ISP default gateway>"
	[IPAddress]$publicDNS="<Your ISP DNS server(s)>"
	Rename-NetAdapter -Name $corpnetAdapterName -NewName Corpnet
	Rename-NetAdapter -Name $internetAdapterName -NewName Internet
	New-NetIPAddress -InterfaceAlias "Internet" -IPAddress $publicIP -PrefixLength $publicIPpreflength -DefaultGateway $publicDG
	Set-DnsClientServerAddress -InterfaceAlias Internet -ServerAddresses $publicDNS
	New-NetIPAddress -InterfaceAlias "Corpnet" -IPAddress 10.0.0.2 -AddressFamily IPv4 -PrefixLength 24
	Set-DnsClientServerAddress -InterfaceAlias "Corpnet" -ServerAddresses 10.0.0.1
	Set-DnsClient -InterfaceAlias "Corpnet" -ConnectionSpecificSuffix corp.contoso.com
	New-NetFirewallRule -DisplayName "Allow ICMPv4-Input" -Protocol ICMPv4
	New-NetFirewallRule -DisplayName "Allow ICMPv4-Output" -Protocol ICMPv4 -Direction Outbound
	Disable-NetAdapterBinding -Name "Internet" -ComponentID ms_msclient
	Disable-NetAdapterBinding -Name "Internet" -ComponentID ms_server
	ping dc1.corp.contoso.com

对于最后一个命令，请验证是否有四个响应来自 IP 地址 10.0.0.1。

这是你当前的配置。

![](./media/virtual-networks-setup-hybrid-cloud-environment-testing/CreateHybridCloudVNet_2.png)

## 阶段 3：创建跨界 Azure 虚拟网络

首先，使用 Azure 订阅凭据登录到 [Azure 经典管理门户](https://manage.windowsazure.cn/)，然后创建名为 TestVNET 的虚拟网络。

1.	在 Azure 经典管理门户的任务栏中，单击“新建”>“网络服务”>“虚拟网络”>“自定义创建”。
2.	在“虚拟网络详细信息”页的“名称”中键入“TestVNET”。
3.	在“位置”中，选择你所在位置的相应数据中心。
4.	单击“下一步”箭头。
5.	在“DNS 服务器和 VPN 连接”页的“DNS 服务器”中，在“选择或输入名称”中键入“DC1”，在“IP 地址”中键入“10.0.0.1”，然后选择“配置站点到站点 VPN”。
6.	在“本地网络”中，选择“指定新的本地网络”。 
7.	单击“下一步”箭头。
8.	在“站点到站点连接”页上执行以下操作：
	- 在“名称”中，键入“Corpnet”。 
	- 在“VPN 设备 IP 地址”中，键入分配给 RRAS1 的 Internet 接口的公共 IP 地址。
	- 在“地址空间”的“起始 IP”列中，键入“10.0.0.0”。在“CIDR (地址计数)”中，选择“/8”，然后单击“添加地址空间”。
9.	单击“下一步”箭头。
10.	在“虚拟网络地址空间”页上执行以下操作：
	- 在“地址空间”的“起始 IP”中，选择“192.168.0.0”。
	- 在“子网”中单击“子网 1”，然后将名称替换为“TestSubnet”。 
	- 在 TestSubnet 的“CIDR (地址计数)”列中，单击“/24 (256)”。
	- 单击**“添加网关子网”**。
11.	单击“完成”图标。请等到虚拟网络创建完以后再继续。

接下来，请按[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/) 中的说明在本地计算机上安装 Azure PowerShell。

接下来，请为 TestVNET 虚拟网络创建新的云服务。你必须选取一个唯一的名称。例如，你可将其命名为TestVNET-*UniqueSequence*，其中 *UniqueSequence* 是你组织的缩写。例如，如果你的组织名称为 Tailspin Toys，则可以将云服务命名为 TestVNET-Tailspin。

你可以使用以下 Azure PowerShell 命令在本地计算机上测试该名称的唯一性。

	Test-AzureName -Service <Proposed cloud service name>

如果此命令返回“False”，则你建议的名称是唯一的。使用此命令创建云服务。

	New-AzureService -Service <Unique cloud service name> -Location "<Same location name as your virtual network>"

接下来，为 TestVNET 虚拟网络创建新的存储帐户。你必须选取一个唯一的名称。你可以使用此命令测试存储帐户名称的唯一性。

	Test-AzureName -Storage <Proposed storage account name>

如果此命令返回“False”，则你建议的名称是唯一的。使用以下命令创建并设置存储帐户。

	New-AzureStorageAccount -StorageAccountName <Unique storage account name> -Location "<Same location name as your virtual network>"
	Set-AzureStorageAccount -StorageAccountName <Unique storage account name>

这是你当前的配置。

![](./media/virtual-networks-setup-hybrid-cloud-environment-testing/CreateHybridCloudVNet_3.png)

 
## 阶段 4：创建站点到站点 VPN 连接

首先，创建一个虚拟网络网关。

1.	在本地计算机上的 Azure 经典管理门户中，单击左窗格中的“网络”，然后验证 TestVNET 的“状态”列是否已设置为“已创建”。
2.	单击“TestVNET”。在“仪表板”页上，你会看到状态“未创建网关”。
3.	在任务栏中，单击“创建网关”，然后单击“动态路由”。出现提示时单击“是”。请一直等到网关完成且其状态更改为“正在连接”。这可能需要几分钟的时间。
4.	记下“仪表板”页中的“网关 IP 地址”。这是 TestVNET 虚拟网络的 Azure VPN 网关的公共 IP 地址。你需要此 IP 地址来配置 RRAS1。
5.	在任务栏中，单击“管理密钥”，然后单击密钥旁边的复制图标，将密钥复制到剪贴板中。将此密钥粘贴到文档中并进行保存。你需要此密钥值来配置 RRAS1。 

接下来，使用路由和远程访问服务配置 RRAS1，让其充当 Corpnet 子网的 VPN 设备。以本地管理员身份登录到 RRAS1，并在 Windows PowerShell 命令提示符下运行这些命令。

	Import-Module ServerManager
	Install-WindowsFeature RemoteAccess -IncludeManagementTools
	Add-WindowsFeature -name Routing -IncludeManagementTools

接下来对 RRAS1 进行配置，以便接收来自 Azure VPN 网关的站点到站点 VPN 连接。重新启动 RRAS1，然后以本地管理员身份登录，并在 Windows PowerShell 命令提示符下运行这些命令。你需要提供 Azure VPN 网关的 IP 地址和密钥值。

	$PresharedKey="<Key value>"
	Import-Module RemoteAccess
	Install-RemoteAccess -VpnType VpnS2S
	Add-VpnS2SInterface -Protocol IKEv2 -AuthenticationMethod PSKOnly -Persistent -NumberOfTries 3 -ResponderAuthenticationMethod PSKOnly -Name S2StoTestVNET -Destination "<IP address of the Azure VPN gateway>" -IPv4Subnet @("192.168.0.0/24:100") -SharedSecret $PresharedKey
	Set-VpnServerIPsecConfiguration -EncryptionType MaximumEncryption
	Set-VpnServerIPsecConfiguration -SADataSizeForRenegotiationKilobytes 33553408
	New-ItemProperty -Path HKLM:\System\CurrentControlSet\Services\RemoteAccess\Parameters\IKEV2 -Name SkipConfigPayload -PropertyType DWord -Value 1
	Restart-Service RemoteAccess

接下来，转到本地计算机上的 Azure 经典管理门户并等待，一直等到 TestVNET 虚拟网络显示状态“已连接”。

接下来，将 RRAS1 配置为支持到各个 Internet 位置的已转换流量。在 RRAS1 上执行以下操作：

1.	在“开始”屏幕中，键入“rras”，然后单击“路由和远程访问”。 
2.	在控制台树中，打开服务器名称，然后单击“IPv4”。
3.	右键单击“常规”，然后单击“新建路由协议”。
4.	单击“NAT”，然后单击“确定”。
5.	在控制台树中，右键单击“NAT”，然后依次单击“新建接口”、“Corpnet”，再单击“确定”两次。
6.	右键单击“NAT”，然后依次单击“新建接口”、“Internet”、“确定”。
7.	在“NAT”选项卡上，单击“连接到 Internet 的公用接口”，选择“在此接口上启用 NAT”，然后单击“确定”。


接下来，将 DC1、APP1 和 CLIENT1 配置为使用 RRAS1 作为其默认网关。
 
在 DC1 的管理员级 Windows PowerShell 命令提示符下运行这些命令。

	New-NetRoute -DestinationPrefix "0.0.0.0/0" -InterfaceAlias "Ethernet" -NextHop 10.0.0.2
	Set-DhcpServerv4OptionValue -Router 10.0.0.2

如果接口名称不是“Ethernet”，请使用 **Get-NetAdapter** 命令确定接口名称。

在 APP1 的管理员级 Windows PowerShell 命令提示符下运行此命令。

	New-NetRoute -DestinationPrefix "0.0.0.0/0" -InterfaceAlias "Ethernet" -NextHop 10.0.0.2

在 CLIENT1 的管理员级 Windows PowerShell 命令提示符下运行此命令。

	ipconfig /renew

这是你当前的配置。
 

![](./media/virtual-networks-setup-hybrid-cloud-environment-testing/CreateHybridCloudVNet_4.png)


## 阶段 5：配置 DC2

首先，在本地计算机的 Azure PowerShell 命令提示符下使用这些命令创建适用于 DC2 的 Azure 虚拟机。

	$ServiceName="<Your cloud service name from Phase 3>"
	$image = Get-AzureVMImage | where { $_.ImageFamily -eq "Windows Server 2012 R2 Datacenter" } | sort PublishedDate -Descending | select -ExpandProperty ImageName -First 1
	$vm1=New-AzureVMConfig -Name DC2 -InstanceSize Medium -ImageName $image
	$cred=Get-Credential -Message "Type the name and password of the local administrator account for DC2."
	$vm1 | Add-AzureProvisioningConfig -Windows -AdminUsername $cred.GetNetworkCredential().Username -Password $cred.GetNetworkCredential().Password 
	$vm1 | Add-AzureProvisioningConfig -Windows -AdminUsername $LocalAdminName -Password $LocalAdminPW	
	$vm1 | Set-AzureSubnet -SubnetNames TestSubnet
	$vm1 | Set-AzureStaticVNetIP -IPAddress 192.168.0.4
	$vm1 | Add-AzureDataDisk -CreateNew -DiskSizeInGB 20 -DiskLabel ADFiles -LUN 0 -HostCaching None
	New-AzureVM -ServiceName $ServiceName -VMs $vm1 -VNetName TestVNET


接下来，登录到新的 DC2 虚拟机上。

1.	在 Azure 经典管理门户的左窗格中，单击“虚拟机”，然后单击 DC2 的“状态”列中的“正在运行”。
2.	在任务栏中，单击“连接”。 
3.	当系统提示你打开 DC2.rdp 时，单击“打开”。
4.	遇到远程桌面连接消息框提示时，单击“连接”。
5.	当系统提示你输入凭据时，请使用以下凭据：
	- 名称：**DC2\**[本地管理员帐户名称]
	- 密码：[本地管理员帐户密码]
6.	当系统使用引用证书的远程桌面连接消息框提示你时，单击“是”。

接下来，配置 Windows 防火墙规则，以允许进行基本的连接测试所需的流量。在 DC2 上的管理员级 Windows PowerShell 命令提示符下运行以下命令：

	Set-NetFirewallRule -DisplayName "File and Printer Sharing (Echo Request - ICMPv4-In)" -enabled True
	ping dc1.corp.contoso.com

使用 ping 命令时，会从 IP 地址 10.0.0.1 传回四个成功的答复。这是对站点到站点 VPN 连接的流量测试。

接下来，将额外的数据磁盘添加为驱动器盘符为 F: 的新卷。

1.	在服务器管理器的左窗格中，单击**“文件和存储服务”**，然后单击**“磁盘”**。
2.	在内容窗格的“磁盘”组中，单击“磁盘 2”（其“分区”设置为“未知”）。
3.	单击**“任务”**，然后单击**“新建卷”**。
4.	在新建卷向导的“开始之前”页上，单击**“下一步”**。
5.	在“选择服务器和磁盘”页上，单击“磁盘 2”，然后单击“下一步”。出现提示时，单击“确定”。
6.	在“指定卷的大小”页上，单击**“下一步”**。
7.	在“分配到驱动器号或文件夹”页上，单击**“下一步”**。
8.	在“选择文件系统设置”页上，单击**“下一步”**。
9.	在“确认选择”页上，单击**“创建”**。
10.	完成后，单击**“关闭”**。

接下来，将 DC2 配置为 corp.contoso.com 域的副本域控制器。在 DC2 上的 Windows PowerShell 命令提示符下运行这些命令。

	Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
	Install-ADDSDomainController -Credential (Get-Credential CORP\User1) -DomainName "corp.contoso.com" -InstallDns:$true -DatabasePath "F:\NTDS" -LogPath "F:\Logs" -SysvolPath "F:\SYSVOL"

请注意，系统会提示你输入 CORP\\User1 密码和目录服务还原模式 (DSRM) 密码，然后重新启动 DC2。

由于 TestVNET 虚拟网络有自己的 DNS 服务器 (DC2)，因此必须将 TestVNET 虚拟网络配置为使用此 DNS 服务器。

1.	在 Azure 经典管理门户的左窗格中，单击“网络”，然后单击“TestVNET”。
2.	单击**“配置”**。
3.	在“DNS 服务器”中，删除“10.0.0.1”条目。
4.	在“DNS 服务器”中添加一个条目，该条目使用“DC2”作为名称，使用“192.168.0.4”作为 IP 地址。 
5.	在底部的命令栏中，单击**“保存”**。
6.	在 Azure 经典管理门户的左窗格中，单击“虚拟机”，然后单击 DC2 旁边的“状态”列。
7.	在命令栏中，单击**“重新启动”**。等待 DC2 重新启动。


这是你当前的配置。

![](./media/virtual-networks-setup-hybrid-cloud-environment-testing/CreateHybridCloudVNet_5.png)

 
现在，你的混合云环境已准备好进行测试。

##<a id="costs"></a> 最大程度地降低此环境的持续使用成本

若要将在此环境中运行虚拟机的成本降到最低，请尽快执行所需的测试和演示，然后删除它们或关闭虚拟机（在不使用时进行）。例如，你可以在每个营业日结束时使用 Azure 自动化和 Runbook 来自动关闭 Test\_VNET 虚拟网络中的虚拟机。

在实施时，Azure VPN 网关将由两台 Azure 虚拟机组成，因此会产生持续的货币成本。有关详细信息，请参阅[定价 - 虚拟网络](/home/features/networking/pricing/)。若要将 VPN 网关的成本降到最低，请创建测试环境并尽快执行所需测试和演示，或者通过这些步骤删除该网关。

1.	在本地计算机上的 Azure 经典管理门户中，依次单击左窗格中的“网络”、“TestVNET”、“仪表板”。
2.	在任务栏中，单击“删除网关”。出现提示时单击“是”。等待，直到网关删除完毕且其状态更改为“未创建网关”。

如果你删除网关后又想要还原测试环境，则必须先创建新的网关。

1.	从本地计算机上的 Azure 经典管理门户中，单击左窗格中的“网络”，然后单击“TestVNET”。在“仪表板”页上，你会看到状态“未创建网关”。
2.	在任务栏中，单击“创建网关”，然后单击“动态路由”。出现提示时单击“是”。请一直等到网关完成且其状态更改为“正在连接”。这可能需要几分钟的时间。
3.	记下“仪表板”页中的“网关 IP 地址”。这是 TestVNET 虚拟网络的 Azure VPN 网关的全新公共 IP 地址。你需要此 IP 地址来重新配置 RRAS1。
4.	在任务栏中，单击“管理密钥”，然后单击密钥旁边的复制图标，将密钥复制到剪贴板中。将此密钥值粘贴到文档中并进行保存。你需要此密钥值来重新配置 RRAS1。 

接下来，以本地管理员身份登录到 RRAS1，然后在管理员级 Windows PowerShell 命令提示符下运行这些命令，以便使用新的公共 IP 地址和预共享的密钥重新配置 RRAS1。

	$PresharedKey="<Key value>"
	Set-VpnS2SInterface -Name S2StoTestVNET -Destination "<IP address of the Azure VPN gateway>" -SharedSecret $PresharedKey

接下来，请转到本地计算机上的 Azure 经典管理门户中等待，一直等到 TestVNET 虚拟网络显示状态为“已连接”。
 
## 后续步骤

- 在此环境中设置 [SharePoint intranet 场](/documentation/articles/virtual-networks-setup-sharepoint-hybrid-cloud-testing/)、[基于 Web 的 LOB 应用程序](/documentation/articles/virtual-networks-setup-lobapp-hybrid-cloud-testing/)或者 [Office 365 目录同步 (DirSync) 服务器](/documentation/articles/virtual-networks-setup-dirsync-hybrid-cloud-testing/)。

<!---HONumber=Mooncake_0307_2016-->