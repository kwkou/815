<properties 
	pageTitle="如何为 Windows 虚拟机重置密码或远程桌面服务" 
	description="使用 PowerShell 命令快速为 Windows 虚拟机重置本地管理员密码或远程桌面服务。" 
	services="virtual-machines" 
	documentationCenter="" 
	authors="JoeDavies-MSFT" 
	manager="timlt" 
	editor=""/>
	
<tags ms.service="virtual-machines" ms.date="03/25/2015" wacn.date="04/11/2015"/>


# 如何为 Windows 虚拟机重置密码或远程桌面服务

如果你由于忘记了密码或远程桌面服务配置有问题而无法连接到 Windows 虚拟机，可以使用 VMAccess 扩展重置本地管理员密码或重置远程桌面服务配置。
 
## 要求

在开始之前，你需要具备以下项：

- Azure PowerShell 模块 0.8.5 版或更高版本。可以使用 **Get-Module azure | format-table version** 命令查看已安装的 Azure PowerShell 的版本。有关说明以及指向最新版本的链接，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。 
- 新的本地管理员帐户密码。如果要重置远程桌面服务配置，则不需要此项。 
- VM 代理。 

VMAccess 扩展无需安装就可使用它。当你运行使用 **Set-AzureVMExtension** cmdlet 的 Azure PowerShell 命令时，只要 VM 代理安装在虚拟机上，就会自动加载该扩展。
 
首先，请验证是否已安装 VM 代理。填写云服务名称和虚拟机名称，然后在管理员级别的 Azure PowerShell 命令提示符下运行以下命令。替换引号内的所有内容，包括 < 和 > 字符。

	$CSName = "<cloud service name>"
	$VMName = "<virtual machine name>"
	$vm = Get-AzureVM -ServiceName $CSName -Name $VMName 
	write-host $vm.VM.ProvisionGuestAgent

如果你不知道云服务和虚拟机名称，运行 **Get-AzureVM** 可显示当前订阅中所有虚拟机的该信息。

如果 **write-host** 命令显示 **True**，则已安装 VM 代理。如果该命令显示 **False**，请参阅 Azure 博客文章 [VM 代理和扩展 - 第 2 部分](http://azure.microsoft.com/blog/2014/04/15/vm-agent-and-extensions-part-2/)中的说明和下载链接。

如果你使用 Azure 管理门户创建了虚拟机，请运行以下附加命令：

	$vm.GetInstance().ProvisionGuestAgent = $true

此命令可防止在以下节运行 Set-AzureVMExtension 命令时出现"在设置 IaaS VM Access 扩展前，必须对 VM 对象启用设置来宾代理"错误。 

现在，你可以执行这些任务：

- 重置本地管理员帐户密码
- 重置远程桌面服务配置

## 重置本地管理员帐户密码

填写当前的本地管理员帐户名称和新密码，然后运行这些命令。

	$cred=Get-Credential -Message "Type the name of the current local administrator account and the new password."	
	Set-AzureVMAccessExtension -vm $vm -UserName $cred.GetNetworkCredential().Username -Password $cred.GetNetworkCredential().Password  | Update-AzureVM

- 如果你键入不同于当前帐户的名称，VMAccess 扩展将重命名本地管理员帐户，将密码分配给该帐户，并发出远程桌面注销命令。
- 如果本地管理员帐户处于禁用状态，则 VMAccess 扩展将启用它。
 
这些命令也重置远程桌面服务配置。

## 重置远程桌面服务配置

若要重置远程桌面服务配置，请运行此命令。

	Set-AzureVMAccessExtension -vm $vm | Update-AzureVM

VMAccess 扩展在 VM　上运行这两个命令：

- **netsh advfirewall firewall set rule group="Remote Desktop" new enable=Yes**

	此命令启用允许传入远程桌面流量的内置 Windows 防火墙组，该组使用 TCP 端口 3389。

- **Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -name "fDenyTSConnections" -Value 0**

	此命令将 fDenyTSConnections 注册表值设为 0，以启用远程桌面连接。

如果这未解决你的远程桌面访问问题，请运行　[Azure IaaS (Windows) 诊断程序包](https://home.diagnostics.support.microsoft.com/SelfHelp?knowledgebaseArticleFilter=2976864)。 

1.	单击此页上的**"Microsoft Azure IaaS (Windows)诊断程序包"**以创建新的诊断会话。
2.	在**"你遇到 Azure VM 的以下哪些问题?"**页上，选择**"与 Azure VM 的 RDP 连接(需要重启)"**问题。 

有关详细信息，请参阅 [Microsoft Azure IaaS (Windows) 诊断程序包知识库文章](https://support.microsoft.com/zh-CN/kb/2976864)。 

如果你无法运行 Azure IaaS (Windows) 诊断程序包或运行该程序包未解决你的问题，请参阅[对与基于 Windows 的 Azure 虚拟机的远程桌面连接进行故障排除](/documentation/articles/virtual-machines-troubleshoot-remote-desktop-connections)。


## 其他资源

[Azure VM 扩展和功能](https://msdn.microsoft.com/zh-CN/library/azure/dn606311.aspx)

[使用 RDP 或 SSH 连接到 Azure 虚拟机](https://msdn.microsoft.com/zh-CN/library/azure/dn535788.aspx)



<!--HONumber=51-->