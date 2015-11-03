<properties
	pageTitle="无法通过 RDP 连接到 Azure VM |Windows Azure"
	description="对运行 Windows 的 Azure 虚拟机的远程桌面或 RDP 连接进行故障排除。"
	services="virtual-machines"
	documentationCenter=""
	authors="dsk-2015"
	manager="timlt"
	editor=""
	tags="azure-service-management,azure-resource-manager"/>

<tags
	ms.service="virtual-machines"
	ms.date="09/16/2015"
	wacn.date="11/02/2015"/>

# 对运行 Windows 的 Azure 虚拟机的远程桌面连接进行故障排除

有多种原因可能导致远程桌面 (RDP) 无法连接到运行 Windows 的 Azure 虚拟机。本文将帮助你找出原因并更正它们。

> [AZURE.NOTE]本文仅适用于运行 Windows 的 Azure 虚拟机。有关对运行 Linux 的 Azure 虚拟机的连接进行故障排除，请参阅[此文](/documentation/articles/virtual-machines-troubleshoot-ssh-connections)。

## 与 Azure 客户支持联系

如果你对本文中的任何点需要更多帮助，可以联系 [MSDN Azure 和堆栈溢出论坛](/support/forums/)上的 Azure 专家 。

或者，你也可以提出 Azure 支持事件。转至 [Azure 支持站点](/support/contact/)并单击“获取支持”。有关使用 Azure 支持的信息，请阅读 [Microsoft Azure 支持常见问题](/support/faq/)。


## 基本步骤

以下基本步骤可帮助解决大多数远程桌面连接失败问题：

- 从 [Azure 门户](https://manage.windowsazure.cn)重置远程桌面服务。单击“浏览全部”>“虚拟机(经典)”> 你的 Windows 虚拟机 >“重置远程访问”。

![重置远程访问](./media/virtual-machines-troubleshoot-remote-desktop-connections/Portal-RDP-Reset-Windows.png)

- [重新启动虚拟机](https://msdn.microsoft.com/zh-cn/library/azure/dn763934.aspx)。

- [调整虚拟机的大小](https://msdn.microsoft.com/zh-cn/library/dn168976.aspx)。


## 在 Windows 上运行 Azure IaaS 诊断程序包

如果从运行 Windows 8、Windows 8.1、Windows Server 2012 或 Windows Server 2012 R2 的计算机进行故障排除，则可以尝试运行 [Azure IaaS (Windows) 诊断程序包](http://support.microsoft.com/kb/2976864)。此程序包可以解决远程桌面存在的许多常见问题。

1.	在[“支持诊断”](https://home.diagnostics.support.microsoft.com/SelfHelp?knowledgebaseArticleFilter=2976864)页上单击“Microsoft Azure IaaS (Windows)诊断程序包”。单击“创建”以创建新的诊断会话。你可以将此会话与不同目标计算机**共享**，或者将它**下载**到本地计算机上。
2.	**运行**此会话，**接受** Microsoft 许可协议，并**启动**诊断工具。
3.	在弹出窗口中对你的 Azure 订阅进行身份验证并按照提示进行操作。
4.	在“你遇到 Azure VM 的以下哪些问题?”页上，选择“与 Azure VM 的 RDP 连接(需要重启)”问题。

如果 Azure IaaS 诊断程序包无法执行或不是很有帮助，则可以转到下一节来根据从远程桌面客户端获得的错误信息解决该问题。


## 常见 RDP 错误

以下是在尝试通过远程桌面连接到 Azure 虚拟机时可能遇到的最常见错误：

1. [远程桌面连接错误：由于没有可用于提供许可证的远程桌面许可证服务器，远程会话已断开连接](#rdplicense)。

2. [远程桌面连接错误：远程桌面找不到计算机“名称”](#rdpname)。

3. [远程桌面连接错误：发生身份验证错误。无法联系本地安全机构](#rdpauth)。

4. [Windows 安全性错误：你的凭据无效](#wincred)。

5. [远程桌面连接错误：此计算机无法连接到远程计算机](#rdpconnect)。

<a id="rdplicense"></a>
### 远程桌面连接错误：由于没有可用于提供许可证的远程桌面许可证服务器，远程会话已断开连接。

原因：用于远程桌面服务器角色的 120 天许可宽限期已过期，你需要安装许可证。

一种解决方法是，从 Azure 门户保存 RDP 文件的本地副本，然后在 Windows PowerShell 命令提示符下运行此命令以进行连接。

		mstsc <File name>.RDP /admin

这将仅禁用该连接的许可。

如果你实际上不需要两个以上与虚拟机的远程桌面连接，则可以使用服务器管理器删除远程桌面服务器角色。

另请参阅 [Azure VM 失败并出现“没有可用的远程桌面许可证服务器”](http://blogs.msdn.com/b/wats/archive/2014/01/21/rdp-to-azure-vm-fails-with-quot-no-remote-desktop-license-servers-available-quot.aspx)博客文章。

<a id="rdpname"></a>
### 远程桌面连接错误：远程桌面找不到计算机“名称”。

原因：你的计算机上的远程桌面客户端无法解析 RDP 文件设置中的计算机名称。

可能的解决方案：

- 如果你在组织的 Intranet 上，请确保你的计算机有权访问代理服务器，并可以向其发送 HTTPS 流量。
- 如果你使用的是本地存储的 RDP 文件，请尝试使用 Azure 门户生成的 RDP 文件。这将确保你使用的是虚拟机或云服务的正确 DNS 名称以及虚拟机的终结点端口。下面是 Azure 门户生成的示例 RDP 文件：

	full address:s:tailspin-azdatatier.chinacloudapp.cn:55919 prompt for credentials:i:1

此 RDP 文件的地址部分由包含 VM 的云服务的完全限定域名（在此示例中为 tailspin-azdatatier.chinacloudapp.cn）和远程桌面通信终结点的外部 TCP 端口 (55919) 组成。

<a id="rdpauth"></a>
### 远程桌面连接错误：发生身份验证错误。无法联系本地安全机构。

原因：目标 VM 在凭据的用户名部分找不到安全机构。

如果用户名格式为“安全机构\\用户名”（例如：CORP\\User1），则 *SecurityAuthority* 部分将是虚拟机的计算机名（表示本地安全机构）或 Active Directory 域名。

可能的解决方案：

- 如果你的用户帐户在 VM 的本地，请检查 VM 名称是否拼写正确。
- 如果该帐户在 Active Directory 域上，请检查域名的拼写是否正确。
- 如果它是 Active Directory 域帐户且域名的拼写正确，请验证该域中是否提供域控制器。在包含域控制器且某个域控制器计算机未启动的 Azure 虚拟网络中，这是一个常见的问题。一种解决方法是，可以使用本地管理员帐户而不是域帐户。

<a id="wincred"></a>
### Windows 安全性错误：你的凭据无效。

原因：目标 VM 无法验证你的帐户名和密码。

基于 Windows 的计算机可以验证本地帐户或域帐户的凭据。

- 对于本地帐户，请使用“计算机名\\用户名”语法（例如：SQL1\\Admin4798）。
- 对于域帐户，请使用“域名\\用户名”语法（例如：CONTOSO\\johndoe）。

如果你已将虚拟机提升为新的 Active Directory 林中的域控制器，则用于登录的本地管理员帐户将转换为新的林和域中具有相同密码的等效帐户。然后将删除本地管理员帐户。例如，如果你使用本地管理员帐户 DC1\\DCAdmin 登录并将虚拟机提升为 corp.contoso.com 域的新林中的域控制器，则将删除 DC1\\DCAdmin 本地帐户并使用同一密码创建新的域帐户 (CORP\\DCAdmin)。

请确保该帐户名称是虚拟机可以验证为有效帐户的名称并且密码正确。

如果需要更改本地管理员帐户的密码，请参阅[如何为 Windows 虚拟机重置密码或远程桌面服务](/documentation/articles/virtual-machines-windows-reset-password)。

<a id="rdpconnect"></a>
### 远程桌面连接错误：此计算机无法连接到远程计算机。

原因：用于连接的帐户没有远程桌面登录权限。

每台 Windows 计算机都具有 Remote Desktop Users 本地组，其中包含可以远程登录它的帐户和组。本地 Administrators 组的成员也具有访问权限，即使这些帐户未在 Remote Desktop Users 本地组中列出也是如此。对于已加入域的计算机，本地管理员组还包含该域的域管理员。

确保你用于连接的帐户具有远程桌面登录权限。一种解决方法是，使用域帐户或本地管理员帐户通过远程桌面连接，然后使用“计算机管理”管理单元（“系统工具”>“本地用户和组”>“组” > Remote Desktop Users）将所需的帐户添加到 Remote Desktop Users 本地组。


## 详细的疑难解答

如果未发生上述任何错误，而你仍无法通过远程桌面连接到 VM，请阅读[此文](/documentation/articles/virtual-machines-rdp-detailed-troubleshoot)以找出其他原因。


## 其他资源

[Azure IaaS (Windows) 诊断程序包](https://home.diagnostics.support.microsoft.com/SelfHelp?knowledgebaseArticleFilter=2976864)

[如何为 Windows 虚拟机重置密码或远程桌面服务](/documentation/articles/virtual-machines-windows-reset-password)

[如何安装和配置 Azure PowerShell](/documentation/articles/install-configure-powershell)

[对于基于 Linux 的 Azure 虚拟机的 Secure Shell (SSH) 连接进行故障排除](/documentation/articles/virtual-machines-troubleshoot-ssh-connections)

[对在 Azure 虚拟机上运行的应用程序的访问进行故障排除](/documentation/articles/virtual-machines-troubleshoot-access-application)

<!---HONumber=76-->