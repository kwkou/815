<properties 
	pageTitle="Azure AD Connect - Windows 远程托管提示" 
	description="与 AD FS 配合使用时的 Azure AD Connect Windows 远程托管提示。" 
	services="active-directory" 
	documentationCenter="" 
	authors="billmath" 
	manager="stevenpo" 
	editor="curtand"/>

<tags 
	ms.service="active-directory" 
	ms.date="08/24/2015" 
	wacn.date="11/02/2015"/>

# Azure AD Connect - Windows 远程托管提示


当你使用 Azure AD Connect 部署 Active Directory 联合身份验证服务或 Web 应用程序代理时，请检查提示下面的要求，以确保连接和配置成功：

- 如果目标服务器已加入域，请确保已启用“Windows 远程托管” 
	* 在权限提升的 PSH 命令窗口中，使用命令“Enable-PSRemoting –force” 

- 如果目标服务器是未加入域的 WAP 计算机，则需要满足一些额外的要求
	- 在目标计算机（WAP 计算机）上： 

- 确保 winrm（Windows 远程管理/WS-Management）服务正在通过“服务”管理单元运行

- 在权限提升的 PSH 命令窗口中，使用命令“Enable-PSRemoting –force”
	- 在运行向导的计算机上（如果目标计算机未加入域或者是不受信任的域）： 

- 在权限提升的 PSH 命令窗口中，使用命令“Set-Item WSMan:\\localhost\\Client\\TrustedHosts –Value <DMZServerFQDN> -Force –Concatenate”
	- 在服务器管理器中：
		- 将外围网络 WAP 主机添加到计算机池（“服务器管理器”->“管理”->“添加服务器”...使用 DNS 选项卡） 
		- 服务器管理器中的“所有服务器”选项卡：右键单击 WAP 服务器并选择“以下列身份进行管理...”，然后输入 WAP 计算机的本地（非域）凭据 
		- 若要验证远程 PSH 连接，请在服务器管理器的“所有服务器”选项卡中：右键单击 WAP 服务器，然后选择“Windows PowerShell”。此时应会打开远程 PSH 会话，以确保可以建立远程 PowerShell 会话。 

**其他资源**


* [有关 Azure AD Connect 帐户和权限的更多信息](/documentation/articles/active-directory-aadconnect-account-summary)
* [Azure AD Connect 的自定义安装](/documentation/articles/active-directory-aadconnect-get-started-custom)
* [MSDN 上的 Azure AD Connect](https://msdn.microsoft.com/zh-cn/library/azure/dn832695.aspx) 

<!---HONumber=67-->