<properties
	pageTitle="对 Azure VM 的远程桌面连接进行故障排除 | Azure"
	description="对 Windows VM 的远程桌面连接错误进行故障排除。获取快速缓解措施，根据错误消息获取帮助和进行详细的网络故障排除。"
	keywords="远程桌面错误,远程桌面连接错误,无法连接到 VM,远程桌面故障排除"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="iainfoulds"
	manager="timlt"
	editor=""
	tags="top-support-issue,azure-service-management,azure-resource-manager"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="04/12/2016"
	wacn.date="06/13/2016"/>

# 对运行 Windows 的 Azure 虚拟机的远程桌面连接进行故障排除

与基于 Windows 的 Azure 虚拟机的远程桌面协议 (RDP) 连接可能会由于各种原因而失败。此问题可能出在 VM 上的远程桌面服务、网络连接或主机计算机上的远程桌面客户端。本文将帮助你找出失败原因并予以更正。


[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-both-include.md)]

本文适用于运行 Windows 的 Azure 虚拟机。对于运行 Linux 的 Azure 虚拟机，请参阅 [Troubleshoot Secure Shell connections to a Linux-based Azure virtual machine（对基于 Linux 的 Azure 虚拟机的安全外壳连接进行故障排除）](/documentation/articles/virtual-machines-linux-troubleshoot-ssh-connection/)。

如果你对本文中的任何观点存在疑问，可以联系 [MSDN Azure 和 CSDN Azure 论坛](/support/forums/)上的 Azure 专家。或者，你也可以提出 Azure 支持事件。请转到 [Azure 支持站点](/support/contact/)并单击“获取支持”。

##<a id="quickfixrdp"></a>修复常见的远程桌面错误

本部分列出了常见的远程桌面连接问题的快速修复步骤。

### 对使用经典部署模型创建的虚拟机进行故障排除

这些步骤可能会帮助解决使用经典部署模型创建的 Azure 虚拟机中的大部分远程桌面连接失败。在执行每个步骤之后，请尝试重新连接到 VM。

- 从 [Azure 门户预览](https://portal.azure.cn)重置远程桌面服务可修复 RDP 服务器的启动问题。选择“浏览”>“虚拟机(经典)”> 你的 Windows 虚拟机 >“重置远程...”。

- 重新启动虚拟机可解决其他启动问题。选择“浏览”>“虚拟机(经典)”> 你的 Windows 虚拟机 >“重新启动”。

- 将虚拟机重新部署到新的 Azure 节点。请参阅 [Redeploy Virtual Machine to new Azure node（将虚拟机重新部署到新的 Azure 节点）](/documentation/articles/virtual-machines-windows-redeploy-to-new-node/)。

	请注意，完成此操作后，你会丢失临时磁盘数据，而系统则会更新与虚拟机关联的动态 IP 地址。

- 查看 VM 的控制台日志或屏幕快照可更正启动问题。选择“浏览”>“虚拟机(经典)”> 你的 Windows 虚拟机 >“设置”>“启动诊断”。


- 检查 VM 的资源运行状况以了解是否有任何平台问题。选择“浏览”>“虚拟机(经典)”> 你的 Windows 虚拟机 >“设置”>“检查运行状况”。

### 对使用 Resource Manager 部署模型创建的虚拟机进行故障排除

[AZURE.INCLUDE [arm-api-version-powershell](../includes/arm-api-version-powershell.md)]

这些步骤可能会帮助解决使用 Resource Manager 部署模型创建的 Azure 虚拟机中的大部分远程桌面连接失败。在执行每个步骤之后，请尝试重新连接到 VM。

> [AZURE.TIP] 如果门户预览中的“连接”按钮灰显，并且你未通过 [Express Route](/documentation/articles/expressroute-introduction/) 或[站点到站点 VPN](/documentation/articles/vpn-gateway-howto-site-to-site-resource-manager-portal/) 连接连接到 Azure，则必须先为 VM 创建并分配一个公共 IP 地址才能使用 RDP。请阅读有关 [Azure 中的公共 IP 地址](/documentation/articles/virtual-network-ip-addresses-overview-arm/)的详细信息。

- 使用 PowerShell 重置远程访问。
	- 如果尚未安装 PowerShell，请使用 Azure Active Directory 方法[安装 Azure PowerShell 并连接到你的 Azure 订阅](/documentation/articles/powershell-install-configure/)。请注意，在 Azure PowerShell 版本 1.0.x 中，无需切换到 Resource Manager 模式。

	- 使用以下任一 PowerShell 命令重置 RDP 连接。将 `myRG`、`myVM`、`myVMAccessExtension` 和位置替换为与你的设置相关的值。

	```
	Set-AzureRmVMExtension -ResourceGroupName "myRG" -VMName "myVM" -Name "myVMAccessExtension" -ExtensionType "VMAccessAgent" -Publisher "Microsoft.Compute" -typeHandlerVersion "2.0" -Location Westus
	```
	或者

  	```
	Set-AzureRmVMAccessExtension -ResourceGroupName "myRG" -VMName "myVM" -Name "myVMAccess" -Location Westus
	```

	> [AZURE.NOTE] 如果这是你第一次重置 RDP 连接，VMAccessAgent 可能还不存在。在前面的示例中，`myVMAccessExtension` 或 `MyVMAccess` 是为新扩展指定的名称，该扩展将在安装代理的过程中安装。这通常设置为 VM 的名称。如果你以前使用过 VMAccessAgent，可以使用 `Get-AzureRmVM -ResourceGroupName "myRG" -Name "myVM"` 检查 VM 的属性，从而获取现有的扩展名称。然后在输出的“Extensions”节下查找。由于一个 VM 上只能有一个 VMAccessAgent，因此在使用 `-ForceReRun` 时，还需要添加 `Set-AzureRmVMExtension.` 参数。此参数强制重新注册代理。

- 重新启动虚拟机可解决其他启动问题。选择“浏览”>“虚拟机”> 你的 Windows 虚拟机 >“重新启动”。

- 调整 VM 大小可修复任何主机问题。选择“浏览”>“虚拟机”> 你的 Windows 虚拟机 >“设置”>“大小”。

- 查看 VM 的控制台日志或屏幕截图可更正启动问题。选择“浏览”>“虚拟机”> 你的 Windows 虚拟机 >“设置”>“启动诊断”。


如果上述步骤无法解决远程桌面连接失败，请转到下一部分。

## 对特定的远程桌面连接错误进行故障排除

下面是在你尝试使用远程桌面连接到 Azure 虚拟机时可能会看到的最常见错误：

- [远程桌面连接错误：由于没有可用于提供许可证的远程桌面许可证服务器，远程会话已断开连接](#rdplicense)。

- [远程桌面连接错误：远程桌面找不到计算机“名称”](#rdpname)。

- [远程桌面连接错误：发生身份验证错误。无法联系本地安全机构](#rdpauth)。

- [Windows 安全性错误：你的凭据无效](#wincred)。

- [远程桌面连接错误：此计算机无法连接到远程计算机](#rdpconnect)。

###<a id="rdplicense"></a>远程桌面连接错误：由于没有可用于提供许可证的远程桌面许可证服务器，远程会话已断开连接。

原因：用于远程桌面服务器角色的 120 天许可宽限期已过期，你需要安装许可证。

一种解决方法是，从门户预览保存 RDP 文件的本地副本，然后在 PowerShell 命令提示符下运行此命令以进行连接。这将仅禁用该连接的许可：

		mstsc <File name>.RDP /admin

如果你实际上不需要两个以上同时与 VM 的远程桌面连接，则可以使用服务器管理器删除远程桌面服务器角色。

有关详细信息，请参阅博客文章 [Azure VM fails with "No Remote Desktop License Servers available"（Azure VM 失败并出现“没有可用的远程桌面许可证服务器”）](http://blogs.msdn.com/b/wats/archive/2014/01/21/rdp-to-azure-vm-fails-with-quot-no-remote-desktop-license-servers-available-quot.aspx)。

###<a id="rdpname"></a>远程桌面连接错误：远程桌面找不到计算机“名称”。

原因：你的计算机上的远程桌面客户端无法解析 RDP 文件设置中的计算机名称。

可能的解决方案：

- 如果你在组织的 Intranet 上，请确保你的计算机有权访问代理服务器，并可以向其发送 HTTPS 流量。

- 如果你使用的是本地存储的 RDP 文件，请尝试使用门户预览生成的 RDP 文件。这将确保你使用的是虚拟机或云服务的正确 DNS 名称以及虚拟机的终结点端口。下面是门户预览生成的示例 RDP 文件：

		full address:s:tailspin-azdatatier.chinacloudapp.cn:55919
		prompt for credentials:i:1

此 RDP 文件的地址部分包含：

- 包含 VM 的云服务的完全限定域名（在本例中为“tailspin-azdatatier.chinacloudapp.cn”）。

- 远程桌面流量终结点的外部 TCP 端口 (55919)。

###<a id="rdpauth"></a>远程桌面连接错误：发生身份验证错误。无法联系本地安全机构。

原因：目标 VM 在凭据的用户名部分找不到安全机构。

如果用户名格式为安全机构\用户名（例如：CORP\\User1），则 *SecurityAuthority* 部分将是虚拟机的计算机名（表示本地安全机构）或 Active Directory 域名。

可能的解决方案：

- 如果帐户在 VM 的本地，请确保 VM 名称拼写正确。

- 如果该帐户在 Active Directory 域上，请检查域名的拼写是否正确。

- 如果它是 Active Directory 域帐户且域名的拼写正确，请验证该域中是否提供域控制器。此问题在包含域控制器的 Azure 虚拟网络中很常见，域控制器不可用的原因是尚未根据解决方法启动它，你可以使用本地管理员帐户，而不要使用域帐户。

###<a id="wincred"></a>Windows 安全性错误：你的凭据无效。

原因：目标 VM 无法验证你的帐户名和密码。

基于 Windows 的计算机可以验证本地帐户或域帐户的凭据。

- 对于本地帐户，请使用“计算机名\\用户名”语法（例如：SQL1\\Admin4798）。
- 对于域帐户，请使用域名\用户名语法（例如：CONTOSO\\johndoe）。

如果你已将 VM 提升为新的 Active Directory 林中的域控制器，则用于登录的本地管理员帐户将转换为新的林和域中具有相同密码的等效帐户。然后将删除本地帐户。

例如，如果你使用本地帐户 DC1\\DCAdmin 登录并将虚拟机提升为 corp.contoso.com 域的新林中的域控制器，则将删除 DC1\\DCAdmin 本地帐户并使用同一密码创建新的域帐户 (CORP\\DCAdmin)。

请确保该帐户名称是虚拟机可以验证为有效帐户的名称并且密码正确。

如果需要更改本地管理员帐户的密码，请参阅[如何为 Windows 虚拟机重置密码或远程桌面服务](/documentation/articles/virtual-machines-windows-reset-rdp/)。

###<a id="rdpconnect"></a>远程桌面连接错误：此计算机无法连接到远程计算机。

原因：用于连接的帐户没有远程桌面登录权限。

每台 Windows 计算机都具有远程桌面用户本地组，其中包含可以远程登录它的帐户和组。本地管理员组的成员也具有访问权限，即使这些帐户未在远程桌面用户本地组中列出也是如此。对于已加入域的计算机，本地管理员组还包含该域的域管理员。

确保你用于连接的帐户具有远程桌面登录权限。解决方法之一是使用域管理员或本地管理员帐户通过远程桌面建立连接。然后使用 Microsoft 管理控制台管理单元（“系统工具”>“本地用户和组”>“组”>“远程桌面用户”）将所需帐户添加到远程桌面用户本地组。

## 排查一般性远程桌面错误

如果未发生上述任何错误，而你仍无法通过远程桌面连接到 VM，请阅读详细的[远程桌面故障排除指南](/documentation/articles/virtual-machines-windows-detailed-troubleshoot-rdp/)。


## 其他资源

[Azure IaaS (Windows) 诊断程序包](https://home.diagnostics.support.microsoft.com/SelfHelp?knowledgebaseArticleFilter=2976864)

[如何为 Windows 虚拟机重置密码或远程桌面服务](/documentation/articles/virtual-machines-windows-reset-rdp/)

[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)

[对基于 Linux 的 Azure 虚拟机的安全外壳连接进行故障排除](/documentation/articles/virtual-machines-linux-troubleshoot-ssh-connection/)

[对在 Azure 虚拟机上运行的应用程序的访问进行故障排除](/documentation/articles/virtual-machines-linux-troubleshoot-app-connection/)

<!---HONumber=Mooncake_0606_2016-->