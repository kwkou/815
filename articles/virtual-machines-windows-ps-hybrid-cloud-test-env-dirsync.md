<!-- ARM: tested -->

<properties 
	pageTitle="Office 365 DirSync 测试环境 | Azure" 
	description="了解如何在混合云中配置 Office 365 目录同步 (DirSync) 服务器，以便进行 IT 专业人员测试或开发测试。" 
	services="virtual-machines-windows" 
	documentationCenter="" 
	authors="JoeDavies-MSFT" 
	manager="timlt" 
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="04/01/2016"
	wacn.date="06/06/2016"/>

# 在混合云中设置 Office 365 目录同步 (DirSync) 以便进行测试

本主题将指导你逐步创建混合云环境，以便测试在 Azure 中托管的带密码同步的 Office 365 目录同步 (DirSync)。这是生成的配置。

![](./media/virtual-machines-windows-ps-hybrid-cloud-test-env-dirsync/virtual-machines-windows-ps-hybrid-cloud-test-env-dirsync-ph3.png)
 
该配置通过你在 Internet 上的位置模拟 Azure 生产环境中的 DirSync 服务器。它包括：

- 简化的本地网络（Corpnet 子网）。
- 在 Azure 中托管的跨界虚拟网络 (TestVNET)。
- 站点到站点 VPN 连接。
- Office 365 FastTrack 试用订阅。
- TestVNET 虚拟网络中运行 Azure AD Connect 工具的 DirSync 服务器和辅助域控制器。

此配置提供了基础和通用的起始点，你可以据此执行以下操作：

- 开发和测试适用于 Office 365 且依赖于与本地 Active Directory 域的同步（使用密码同步）的应用程序。
- 对这个基于云的 IT 工作负载进行测试。

设置此混合云测试环境需要完成以下三个主要阶段：

1.	设置用于测试的混合云环境。
2.	配置 Office 365 FastTrack 试用版。
3.	配置 DirSync 服务器 (DS1)。

如果你还没有 Azure 订阅，可以通过[试用 Azure](/pricing/1rmb-trial/) 注册试用帐户。

## 阶段 1：设置混合云环境

使用[设置用于测试的混合云环境](/documentation/articles/virtual-machines-windows-ps-hybrid-cloud-test-env-base/)主题中的说明。由于此测试环境不需要 Corpnet 子网上存在 APP1 服务器，因此现在即可将其关闭。

这是你当前的配置。

![](./media/virtual-machines-windows-ps-hybrid-cloud-test-env-dirsync/virtual-machines-windows-ps-hybrid-cloud-test-env-dirsync-ph1.png)

> [AZURE.NOTE] 就阶段 1 来说，你也可以设置[模拟混合云测试环境](/documentation/articles/virtual-machines-windows-ps-hybrid-cloud-test-env-sim/)。

## 阶段 2：配置 Office 365 FastTrack 试用版

若要启动 Office 365 FastTrack 试用版，你需要一个虚构的公司名称以及一个 Microsoft 帐户。我们建议你将 Contoso 这个在 Microsoft 示例内容中使用的虚构的公司名称变一下，将其用作你的公司名称，不过这不是必需的。

接下来，注册新的 Microsoft 帐户。转到 **http://outlook.com**，使用 user123@outlook.com 这样的电子邮件地址创建一个帐户。你将使用此帐户注册 Office 365 FastTrack 试用版。

接下来，注册新的 Office 365 Enterprise E3 试用版。

1.	使用 CORP\\User1 帐户凭据登录 CLIENT1。
2.	打开 Internet Explorer 并转到 **https://go.microsoft.com/fwlink/p/?LinkID=403802**。
3.	逐步完成注册 Office 365 Enterprise E3 试用版的过程。

当系统提示输入**业务电子邮件地址**时，指定新的 Microsoft 帐户。

当系统提示创建 ID 时，键入初始 Office 365 帐户的名称，然后键入虚构的公司名称和密码。在安全的位置记录生成的电子邮件地址（例如 user123@contoso123.partner.onmschina.cn）和密码。你将需要此信息才能完成阶段 3 中的 Azure AD Connect 配置。

完成后，你应该会看到 Office 365 门户主页。在顶部功能区中，单击“管理”，然后单击“Office 365”。此时会出现 Office 365 管理中心页。让此页在 CLIENT1 上保持打开状态。

这是你当前的配置。

![](./media/virtual-machines-windows-ps-hybrid-cloud-test-env-dirsync/virtual-machines-windows-ps-hybrid-cloud-test-env-dirsync-ph2.png)

## 阶段 3：配置目录同步服务器 (DS1)

[AZURE.INCLUDE [arm-api-version-powershell](../includes/arm-api-version-powershell.md)]

从 Azure 门户预览启动 DC2 计算机（如果需要）。

接下来，在本地计算机上的 Azure PowerShell 命令提示符下使用这些命令创建适用于 DS1 的 Azure 虚拟机。在运行这些命令之前，请填写变量值并删除 < and > 字符。

	$rgName="<your resource group name>"
	$locName="<your Azure location, such as China North>"
	$saName="<your storage account name>"
	
	$vnet=Get-AzureRMVirtualNetwork -Name "TestVNET" -ResourceGroupName $rgName
	$subnet=Get-AzureRmVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name "TestSubnet"
	$pip=New-AzureRMPublicIpAddress -Name DS1-NIC -ResourceGroupName $rgName -Location $locName -AllocationMethod Dynamic
	$nic=New-AzureRMNetworkInterface -Name DS1-NIC -ResourceGroupName $rgName -Location $locName -Subnet $subnet -PublicIpAddress $pip
	$vm=New-AzureRMVMConfig -VMName DS1 -VMSize Standard_A1
	$storageAcc=Get-AzureRMStorageAccount -ResourceGroupName $rgName -Name $saName
	$cred=Get-Credential -Message "Type the name and password of the local administrator account for DS1."
	$vm=Set-AzureRMVMOperatingSystem -VM $vm -Windows -ComputerName DS1 -Credential $cred -ProvisionVMAgent -EnableAutoUpdate
	$vm=Set-AzureRMVMSourceImage -VM $vm -PublisherName MicrosoftWindowsServer -Offer WindowsServer -Skus 2012-R2-Datacenter -Version "latest"
	$vm=Add-AzureRMVMNetworkInterface -VM $vm -Id $nic.Id
	$osDiskUri=$storageAcc.PrimaryEndpoints.Blob.ToString() + "vhds/DS1-TestLab-OSDisk.vhd"
	$vm=Set-AzureRMVMOSDisk -VM $vm -Name DS1-TestVNET-OSDisk -VhdUri $osDiskUri -CreateOption fromImage
	New-AzureRMVM -ResourceGroupName $rgName -Location $locName -VM $vm

接下来，在 Azure 门户预览中使用本地管理员帐户的凭据连接到 DS1 虚拟机。

接下来，配置 Windows 防火墙规则，以允许进行基本的连接测试所需的流量。在 DS1 上的管理员级 Windows PowerShell 命令提示符下，运行这些命令。

	Set-NetFirewallRule -DisplayName "File and Printer Sharing (Echo Request - ICMPv4-In)" -enabled True
	ping dc2.corp.contoso.com

使用 ping 命令时，会从 IP 地址 192.168.0.4 传回四个成功的答复。

接下来，在 Windows PowerShell 提示符下使用以下命令将 DS1 加入 CORP Active Directory 域。

	Add-Computer -DomainName corp.contoso.com
	Restart-Computer

当系统提示你为 **Add-Computer** 命令提供域帐户凭据时，请使用 CORP\\User1 帐户。

重新启动后，在 Azure 门户预览中使用 CORP\\User1 帐户和密码连接到 DS1。

接下来，在管理员级别的 Windows PowerShell 命令提示符下使用以下命令在 DS1 上安装 .NET 3.5。

	Add-WindowsFeature NET-Framework-Core

接下来，为你的 Office 365 FastTrack 试用版启用目录同步。

1.	在 CLIENT1 的“Office 365 管理中心”页的左窗格中，单击“用户”，然后单击“活动用户”。
2.	对于 **Active Directory 同步**，请单击“设置”。
3.	在“设置和管理 Active Directory 同步”页的步骤 3 中，单击“激活”。
4.	遇到提示“是否要激活 Active Directory 同步?”时，单击“激活”。执行此操作之后，“Active Directory 同步已激活”会出现在步骤 3 中。
5.	让“设置和管理 Active Directory 同步”页在 CLIENT1 上处于打开状态。

接下来，在 DC1 上的 Windows PowerShell 命令提示符下**逐个**运行以下命令，以便创建新的名为 contoso\_users 的组织单位，然后为 Marci Kaufman 和 Lynda Meyer 添加两个新的用户帐户。

	New-ADOrganizationalUnit -Name contoso_users -Path "DC=corp,DC=contoso,DC=com"
	New-ADUser -SamAccountName marcik -AccountPassword (Read-Host "Set user password" -AsSecureString) -name "Marci Kaufman" -enabled $true -PasswordNeverExpires $true -ChangePasswordAtLogon $false -Path "OU=contoso_users,DC=corp,DC=contoso,DC=com"
	New-ADUser -SamAccountName lyndam -AccountPassword (Read-Host "Set user password" -AsSecureString) -name "Lynda Meyer" -enabled $true -PasswordNeverExpires $true -ChangePasswordAtLogon $false -Path "OU=contoso_users,DC=corp,DC=contoso,DC=com"

当你运行每个 **New-ADUser** Windows PowerShell 命令时，系统会提示你输入新用户的密码。记录这些密码并将其存储在安全的位置。稍后你将需要它们。

接下来，在 DS1 上安装并配置 Azure AD Connect 工具。

1.	运行 Internet Explorer，在**地址**栏中键入 **https://www.microsoft.com/download/details.aspx?id=47594**，然后按 Enter。
2.	运行 Azure AD Connect 安装程序。
3.	从桌面上双击“Azure AD Connect”。
4.	在“欢迎使用”页上，选择“我同意许可条款和隐私声明”，然后单击“继续”。
5.	在“快速设置”页上，单击“使用快速设置”。
6.	在“连接到 Azure AD”页上，键入你在阶段 2 设置 Office 365 FastTrack 试用版时创建的初始帐户的电子邮件地址和密码。单击“下一步”。
7.	在“连接到 AD DS”页上，在“用户名”中键入“CORP\\User1”，在“密码”中键入 User1 帐户密码。单击“下一步”。
8.	在“准备配置”页上，复查设置，然后单击“安装”。
9.	在“配置完成”页上，单击“退出”。

接下来，验证 CORP 域中的用户帐户是否同步到 Office 365。请注意，可能需要等待几分钟才会进行同步。

在 CLIENT1 的“设置和管理 Active Directory 同步”页上，单击该页步骤 6 中的“用户”链接。如果目录同步已成功，你会看到类似于下面这样的内容。

![](./media/virtual-machines-windows-ps-hybrid-cloud-test-env-dirsync/virtual-machines-windows-ps-hybrid-cloud-test-env-dirsync-example.png)

“状态”列指示该帐户是通过与 Active Directory 域同步而获得的。

接下来，使用 Lynda Myer Active Directory 帐户演示 Office 365 密码同步。

1.	在 CLIENT1 的“活动用户”页上，选择 **Lynda Meyer** 帐户。
2.	在 Lynda Meyer 帐户的属性中的“已分配许可证”下，单击“编辑”。
3.	在“分配许可证”选项卡的“设置用户位置”中选择一个位置（如美国）。
4.	选择“Microsoft Office 365 计划 E3”，然后单击“保存”。
5.	关闭 Internet Explorer。
6.	运行 Internet Explorer 并转到 **http://portal.partner.microsoftonline.cn**。
7.	使用 Lynda Meyer 的 Office 365 凭据登录。她的用户名将是 lyndam@<*Your Fictional Name*>.partner.onmschina.cn。该密码是 Lynda Meyer Active Directory 用户帐户密码。
8.	成功登录后，你会看到 Office 365 门户主页，其中显示“让我们今天有所作为”。

这是你当前的配置。

![](./media/virtual-machines-windows-ps-hybrid-cloud-test-env-dirsync/virtual-machines-windows-ps-hybrid-cloud-test-env-dirsync-ph3.png)
 
现在，此环境已准备就绪，你可以对依赖于 Office 365 DirSync 功能的 Office 365 应用程序执行测试，也可以通过 DS1 测试 DirSync 功能和性能。

## 后续步骤

- [在生产环境中](http://technet.microsoft.com/zh-cn/library/dn635310.aspx)部署此工作负荷。

<!---HONumber=Mooncake_0425_2016-->