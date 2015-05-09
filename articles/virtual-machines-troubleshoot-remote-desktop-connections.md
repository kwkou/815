<properties 
	pageTitle="对与基于 Windows 的 Azure 虚拟机的远程桌面连接进行故障排除" 
	description="了解如何使用诊断和隔离问题源的步骤来还原与 Azure 虚拟机的远程桌面 (RDP) 连接。"
	services="virtual-machines" 
	documentationCenter="" 
	authors="JoeDavies-MSFT" 
	manager="timlt" 
	editor=""/>
	
<tags ms.service="virtual-machines" ms.date="03/25/2015" wacn.date="04/11/2015"/>


# 对与基于 Windows 的 Azure 虚拟机的远程桌面连接进行故障排除

本主题介绍一种有条不紊的方法，用于更正和确定与基于 Windows 的 Azure 虚拟机的远程桌面连接问题的根本原因。

## 步骤 1：运行 Azure IaaS 诊断程序包

若要解决创建远程桌面连接时遇到的许多常见问题，Microsoft 已创建 [Azure IaaS (Windows) 诊断程序包](https://home.diagnostics.support.microsoft.com/SelfHelp?knowledgebaseArticleFilter=2976864)。 

1.	单击此页上的**"Microsoft Azure IaaS (Windows) 诊断程序包"**以创建新的诊断会话。
2.	在**"你遇到 Azure VM 的以下哪些问题?"**页上，选择**"与 Azure VM 的 RDP 连接(需要重启)"**问题。 

有关详细信息，请参阅 [Microsoft Azure IaaS (Windows) 诊断程序包知识库文章](https://support.microsoft.com/zh-CN/kb/2976864)。 

如果运行 Azure IaaS 诊断程序包未解决此问题，或者你无法运行诊断程序包，则可能需要以下步骤中所述的更详细的疑难解答。

> [AZURE.NOTE] 必须在运行 Windows 8、Windows 8.1、Windows Server 2012 或 Windows Server 2012 R2 的计算机上运行 Azure IaaS (Windows) 诊断程序包。

## 步骤 2：确定来自远程桌面客户端的错误消息

根据获得的错误消息使用这些部分。

### 远程桌面连接消息窗口：由于没有可用于提供许可证的远程桌面许可证服务器，远程会话已断开连接。

原因：用于远程桌面服务器角色的 120 天许可宽限期已过期，你需要安装许可证。
 
一种临时解决方法是，从 Azure 管理门户保存 RDP 文件的本地副本，然后在 Windows PowerShell 命令提示符下运行此命令以进行连接。 

	mstsc <File name>.RDP /admin

这将仅禁用该连接的许可。 

如果你实际上不需要两个以上与虚拟机的远程桌面连接，则可以使用服务器管理器删除远程桌面服务器角色。

### 远程桌面连接消息窗口：远程桌面找不到计算机"名称"。

原因：你的计算机上的远程桌面客户端无法解析 RDP 文件设置中的计算机名称。 

下面是 Azure 管理门户生成的示例 RDP 文件：

	full address:s:tailspin-azdatatier.chinacloudapp.cn:55919
	prompt for credentials:i:1

地址部分由包含虚拟机（在此示例中为 tailspin-azdatatier.cloudapp.net）的云服务的完全限定域名和远程桌面通信 (55919) 终结点的外部 TCP 端口组成。

此问题的可能解决方案：

- 如果你在组织的 Intranet 上，请确保你的计算机有权访问代理服务器，并可以向其发送 HTTPS 流量。
- 如果你使用的是本地存储的 RDP 文件，请尝试使用 Azure 管理门户生成的 RDP 文件，以确保使用正确的云服务名称和虚拟机的终结点端口。

### Windows 安全消息窗口：你的凭据无效。

原因：你所连接到的虚拟机无法验证你提交的帐户名称和密码。

基于 Windows 的计算机可以验证本地帐户或基于域的帐户的凭据。 

- 对于本地帐户，请使用 <计算机名>\<帐户名称> 语法（示例：SQL1\Admin4798）。 
- 对于域帐户，请使用 <域名>\<帐户名称> 语法（示例：CONTOSO\johndoe）。

对于提升为新的 AD 林中的域控制器的计算机，你执行提升时登录到的本地管理员帐户将转换为新的林和域中具有相同密码的等效帐户。以前的本地管理员帐户将被删除。例如，如果你使用本地管理员帐户 DC1\DCAdmin 登录并将虚拟机提升为 corp.contoso.com 域的新林中的域控制器，则将删除 DC1\DCAdmin 本地帐户并使用同一密码创建新的域帐户 CORP\DCAdmin。

请仔细检查该帐户名称，以确保它是虚拟机可以验证为有效帐户的名称。请仔细检查密码以确保它是正确的密码。 

如果需要更改本地管理员帐户的密码，请参阅[如何为 Windows 虚拟机重置密码或远程桌面服务](/documentation/articles/virtual-machines-windows-reset-password)。

### 远程桌面连接消息窗口：此计算机无法连接到远程计算机。

原因：用于连接的帐户没有远程桌面登录权限。 

每台 Windows 计算机都具有 Remote Desktop Users 本地组，其中包含有权使用远程桌面连接登录的帐户和组。本地 Administrators 组的成员也具有访问权限，即使这些帐户未作为 Remote Desktop Users 本地组的成员列出也是如此。对于已加入域的计算机，本地 Administrators 组还包含该域的域管理员。

确保你用于启动连接的帐户具有远程桌面登录权限。使用域管理员或本地管理员帐户作为创建远程桌面连接的临时解决方法，并使用"计算机管理"管理单元将所需帐户添加到 Remote Desktop Users 本地组（**系统工具 > 本地用户和组 > 组 > Remote Desktop Users**）中。

### 远程桌面连接消息窗口：由于以下原因之一，远程桌面无法连接到远程计算机...

原因：远程桌面客户端无法访问虚拟机上的"远程桌面服务"服务。此问题可能是由于许多根本原因导致。 

以下是所涉及的组件集。

![](./media/virtual-machines-troubleshoot-remote-desktop-connections/tshootrdp_0.png)
 
以下各节将单步执行隔离并确定此问题的各种根本原因的步骤并提供解决方案和解决方法。

## 步骤 3：在详细进行故障排除前的预备步骤

执行以下步骤：

- 在 Azure 管理门户或 Azure 预览版门户中检查虚拟机的状态
- [重启虚拟机](https://msdn.microsoft.com/zh-CN/library/azure/dn763934.aspx)
- [调整虚拟机的大小](https://msdn.microsoft.com/zh-CN/library/dn168976.aspx)

在执行这些步骤之后，重试该远程桌面连接。 

## 步骤 4：详细的疑难解答 

远程桌面客户端无法访问 Azure 虚拟机上的"远程桌面服务"服务可能是由于以下问题或配置错误源导致：

- 远程桌面客户端计算机
- 组织 Intranet 边缘设备
- 云服务终结点和访问控制列表 (ACL)
- 网络安全组
- 基于 Windows 的 Azure 虚拟机

### 源 1：远程桌面客户端计算机

要使你的计算机不会成为问题或配置错误的源，请验证你的计算机是否可以与另一台基于 Windows 的本地计算机建立远程桌面连接。 

![](./media/virtual-machines-troubleshoot-remote-desktop-connections/tshootrdp_1.png)
 
如果不能，请检查你的计算机上是否有以下项：

- 阻止远程桌面流量的本地防火墙设置
- 本地安装的阻止远程桌面连接的客户端代理软件
- 本地安装的阻止远程桌面连接的网络监视软件
- 其他类型的阻止远程桌面连接的安全软件，该软件监视流量或允许/禁止特定类型的流量

对于所有这些情况，请尝试暂时禁用该软件，然后尝试与本地计算机建立远程桌面连接以确定根本原因。然后，与网络管理员协作，更正该软件的设置，以允许远程桌面连接。

### 源 2：组织 Intranet 边缘设备

要使组织 Intranet 边缘设备不会成为问题或配置错误的源，请验证直接连接到 Internet 的计算机是否可以与 Azure 虚拟机建立远程桌面连接。

![](./media/virtual-machines-troubleshoot-remote-desktop-connections/tshootrdp_2.png)
 
如果没有直接连接到 Internet 的计算机，则可以轻松地在其自己的云服务中创建新的 Azure 虚拟机并使用它。有关详细信息，请参阅[在 Azure 中创建运行 Windows 的虚拟机](/documentation/articles/virtual-machines-windows-tutorial)。测试完成后，请删除该虚拟机和云服务。

如果你可以创建与直接连接到 Internet 的计算机的远程桌面连接，请检查你组织的 Intranet 边缘设备中是否有以下项：

- 阻止与 Internet 的 HTTPS 连接的内部防火墙
- 阻止远程桌面连接的代理服务器
- 边界网络中的设备上运行的阻止远程桌面连接的入侵检测或网络监视软件

与网络管理员协作，更正组织 Intranet 边缘设备的设置，以允许与 Internet 建立基于 HTTPS 的远程桌面连接。

### 源 3：云服务终结点和 ACL

要使云服务终结点和 ACL 不会成为问题或配置错误的源，请验证同一个云服务中的另一个 Azure 虚拟机是否可以与你的 Azure 虚拟机建立远程桌面连接。

![](./media/virtual-machines-troubleshoot-remote-desktop-connections/tshootrdp_3.png)
 
如果你在同一个云服务中没有另一个虚拟机，可以轻松创建一个新虚拟机。有关详细信息，请参阅[在 Azure 中创建运行 Windows 的虚拟机](/documentation/articles/virtual-machines-windows-tutorial)。测试完成后，请删除多余的虚拟机。

如果可以创建与同一云服务中的虚拟机的远程桌面连接，请检查以下项：

- 目标虚拟机上的远程桌面通信的终结点配置。终结点的专用 TCP 端口必须与虚拟机上"远程桌面服务"服务侦听的 TCP 端口（默认为 3389）匹配。 
- 目标虚拟机上的远程桌面通信终结点的 ACL。ACL 允许你指定基于源 IP 地址允许或拒绝从 Internet 传入的流量。错误配置 ACL 可能会阻止传入远程桌面流量到达终结点。检查你的 ACL 以确保允许从你的代理服务器或其他边缘服务器的公共 IP 地址传入的流量。有关详细信息，请参阅[关于网络访问控制列表 (ACL)](https://msdn.microsoft.com/zh-CN/library/azure/dn376541.aspx)。

要使终结点不会成为问题的源，请删除当前终结点，然后创建一个新终结点，并选择范围 49152-65535 中的随机端口作为外部端口号。有关详细信息，请参阅[在 Azure 中的虚拟机上设置终结点](/documentation/articles/virtual-machines-set-up-endpoints)。

### 源 4：网络安全组

使用网络安全组可以对允许的入站和出站流量进行更精细的控制。你可以创建跨 Azure 虚拟网络中的子网和云服务的规则。检查你的网络安全组规则，以确保允许来自 Internet 的远程桌面流量。

有关详细信息，请参阅[关于网络安全组](https://msdn.microsoft.com/zh-CN/library/azure/dn848316.aspx)。

### 源 5：基于 Windows 的 Azure 虚拟机

最后一组可能的问题是在 Azure 虚拟机本身上。

![](./media/virtual-machines-troubleshoot-remote-desktop-connections/tshootrdp_5.png)
 
首先，如果你无法针对**"与 Azure VM 的 RDP 连接(需要重启)"**问题运行 [Azure IaaS (Windows) 诊断程序包](https://home.diagnostics.support.microsoft.com/SelfHelp?knowledgebaseArticleFilter=2976864)，请按照[如何为 Windows 虚拟机重置密码或远程桌面服务](/documentation/articles/virtual-machines-windows-reset-password)中的说明进行操作，重置虚拟机上的"远程桌面服务"服务。这将：

- 启用"远程桌面"Windows 防火墙默认规则（TCP 端口 3389）。
- 通过将 HKLM\System\CurrentControlSet\Control\Terminal Server\fDenyTSConnections 注册表值设置为 0，启用远程桌面连接。

从你的计算机重试连接。如果不成功，以下是一些可能的问题：

- "远程桌面服务"服务未在目标虚拟机上运行。
- "远程桌面服务"服务不侦听 TCP 端口 3389。
- Windows 防火墙或其他本地防火墙使用阻止远程桌面通信的出站规则。
- Azure 虚拟机上运行的入侵检测或网络监视软件正在阻止远程桌面连接。

若要更正这些可能的问题，可以使用 Azure 虚拟机的远程 PowerShell 会话。首先，必须安装虚拟机托管云服务的证书。转到[为 Azure 虚拟机配置安全远程 PowerShell 访问](http://gallery.technet.microsoft.com/scriptcenter/Configures-Secure-Remote-b137f2fe)，并将 **InstallWinRMCertAzureVM.ps1** 脚本文件下载到本地计算机上的文件夹中。

接下来，安装 Azure PowerShell（如果尚未安装）。请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/install-configure-powershell)。

接下来，打开 Azure PowerShell 命令提示符，并将当前文件夹更改为 **InstallWinRMCertAzureVM.ps1** 脚本文件所在的位置。若要运行 PowerShell 脚本，必须设置正确的执行策略。运行 **Get-ExecutionPolicy** 命令，以确定当前的策略级别。有关设置相应级别的信息，请参阅 [Set-ExecutionPolicy](https://technet.microsoft.com/zh-CN/library/hh849812.aspx)。

接下来，填写你的 Azure 订阅名称、云服务名称和虚拟机名称（删除 < 和 > 字符），然后运行这些命令。

	$subscr="<Name of your Azure subscription>"
	$serviceName="<Name of the cloud service that contains the target virtual machine>"
	$vmName="<Name of the target virtual machine>"
	.\InstallWinRMCertAzureVM.ps1 -SubscriptionName $subscr -ServiceName $serviceName -Name $vmName

你可以从 **Get-AzureSubscription** 命令显示的 SubscriptionName 属性获取正确的订阅名称。可以从 **Get-AzureVM** 命令显示的 ServiceName 列中获取虚拟机的云服务名称。

为了证明你拥有此新证书，请打开侧重于当前用户的"证书"管理单元，然后在**"受信任的根证书颁发机构\证书"**文件夹中查找。你应看到在"颁发给"列中具有你的云服务的 DNS 名称的证书（示例：cloudservice4testing.cloudapp.net）。

接下来，使用以下命令启动远程 PowerShell 会话。

	$uri = Get-AzureWinRMUri -ServiceName $serviceName -Name $vmName
	$creds = Get-Credential
	Enter-PSSession -ConnectionUri $uri -Credential $creds

输入有效的管理员凭据之后，你应看到如下内容作为 Azure PowerShell 提示符：

	[cloudservice4testing.cloudapp.net]: PS C:\Users\User1\Documents>

提示符的第一部分指示你现在正在对包含目标虚拟机的云服务发出 PowerShell 命令。你的云服务名称将是不同于"cloudservice4testing.cloudapp.net"的内容。 

现在，你可以发出 PowerShell 命令，以调查上面提到的其他问题，并进行配置更正。

### 手动更正"远程桌面服务"服务侦听 TCP 端口

如果你无法针对**"与 Azure VM 的 RDP 连接(需要重启)"**问题运行 [Azure IaaS (Windows) 诊断程序包](https://home.diagnostics.support.microsoft.com/SelfHelp?knowledgebaseArticleFilter=2976864)，请在远程 PowerShell 会话提示符下，运行此命令。

	Get-ItemProperty -Path "HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" -Name "PortNumber"

PortNumber 属性显示当前端口号。如果需要，可使用此命令将远程桌面端口号更改回其默认值 (3389)。

	Set-ItemProperty -Path "HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" -Name "PortNumber" -Value 3389

使用此命令验证是否已将端口更改为 3389。

	Get-ItemProperty -Path "HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" -Name "PortNumber"

使用此命令退出远程 PowerShell 会话。

	Exit-PSSession

验证 Azure 虚拟机的远程桌面终结点是否也使用 TCP 端口 3398 作为其内部端口。然后，重启 Azure 虚拟机，并重试远程桌面连接。

## 步骤 5：将你的问题提交到 Azure 支持论坛

若要从全世界的 Azure 专家那里获取帮助，你可以将你的问题提交到 MSDN Azure 论坛或 Stack Overflow 论坛。有关详细信息，请参阅 [Azure 论坛](https://social.msdn.microsoft.com/Forums/zh-CN/home?forum=windowsazucezhchs)。

## 步骤 6：提出 Azure 支持事件

如果你已针对**"与 Azure VM 的 RDP 连接(需要重启)"**问题运行 [Azure IaaS (Windows) 诊断程序包](https://home.diagnostics.support.microsoft.com/SelfHelp?knowledgebaseArticleFilter=2976864)或已完成本文中的步骤 2 到步骤 5，并已将你的问题提交到 Azure 支持论坛，但仍无法创建远程桌面连接，则要考虑的一种替代方法是：是否可以重新创建虚拟机。

如果无法重新创建虚拟机，则可能是提出 Azure 支持事件的时候了。

若要提出事件，请转到 [Azure 支持站点](/support/) ，然后单击**"获取支持"**。


## 其他资源

[Azure IaaS (Windows) 诊断程序包](https://home.diagnostics.support.microsoft.com/SelfHelp?knowledgebaseArticleFilter=2976864)

[如何为 Windows 虚拟机重置密码或远程桌面服务](/documentation/articles/virtual-machines-windows-reset-password)

[如何安装和配置 Azure PowerShell](/documentation/articles/install-configure-powershell)

[虚拟机文档](/documentation/services/virtual-machines/)

[Azure 虚拟机常见问题](https://msdn.microsoft.com/zh-CN/library/azure/dn683781.aspx)



<!--HONumber=51-->