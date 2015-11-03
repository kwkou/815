<properties 
	pageTitle="Azure AD Connect Health AD FS 代理安装 | Microsoft Azure" 
	description="本页与 Azure AD Connect Health 相关，介绍如何安装 Active Directory 联合身份验证服务 (AD FS) 代理。" 
	services="active-directory" 
	documentationCenter="" 
	authors="billmath" 
	manager="stevenpo" 
	editor="curtand"/>

<tags 
	ms.service="active-directory" 
	ms.date="08/14/2015" 
	wacn.date="11/02/2015"/>






# 适用于 AD FS 的 Azure AD Connect Health 代理安装

本文档将指导你在服务器上安装和配置适用于 AD FS 的 Azure AD Connect Health 代理。

>[AZURE.NOTE]请记住在查看 Azure AD Connect Health 实例中的任何数据之前，你将需要在目标服务器上安装 Azure AD Connect Health 代理。请务必在安装代理之前，满足[此处](/documentaion/articles/active-directory-aadconnect-health#requirements)所述的要求。可以从[此处](http://go.microsoft.com/fwlink/?LinkID=518973)下载该代理。


## 代理安装
若要启动代理安装，请双击下载的 .exe 文件。在第一个屏幕上，单击“安装”。

![验证 Azure AD Connect Health](./media/active-directory-aadconnect-health-requirements/install1.png)

安装完成后，单击“立即配置”。

![验证 Azure AD Connect Health](./media/active-directory-aadconnect-health-requirements/install2.png)

这将启动后跟一些将执行 Register-AzureADConnectHealthADFSAgent 的 PowerShell 的命令提示符。系统将提示你登录到 Azure。继续登录。

![验证 Azure AD Connect Health](./media/active-directory-aadconnect-health-requirements/install3.png)


登录后，将继续 PowerShell。完成后你可以关闭 PowerShell，配置已完成。

此时，应自动启动服务且代理将在此时监视和收集数据。请注意如果未满足已在前面几节中所述的所有先决条件，你将在 PowerShell 窗口看到警告。请务必在安装代理之前，满足[此处](/documentaion/articles/active-directory-aadconnect-health#requirements)所述的要求。以下屏幕截图是这些错误的一个示例。

![验证 Azure AD Connect Health](./media/active-directory-aadconnect-health-requirements/install4.png)

若要验证代理是否已安装，打开服务并检查以下方面。如果你完成了配置，这些服务应运行。否则，配置完成之前它们将无法启动。

- Azure AD Connect Health AD FS Diagnostics 服务
- Azure AD Connect Health AD FS Insights 服务
- Azure AD Connect Health AD FS Monitoring 服务
 
![验证 Azure AD Connect Health](./media/active-directory-aadconnect-health-requirements/install5.png)


## 在 Windows Server 2008 R2 服务器上的代理安装

对于 Windows Server 2008 R2 服务器执行以下操作：

1. 确保正在运行 Service Pack 1 或更高版本的服务器。
1. 关闭代理安装的 IE ESC：
1. 在安装 AD Health 代理之前，在每台服务器上安装 Windows PowerShell 4.0。若要安装 Windows PowerShell 4.0：
 - 使用以下链接下载脱机安装程序，安装 [Microsoft.NET Framework 4.5](https://www.microsoft.com/download/details.aspx?id=40779)。
 - （从 Windows 功能）安装 PowerShell ISE
 - 安装 [Windows Management Framework 4.0。](https://www.microsoft.com/download/details.aspx?id=40855)
 - 在服务器上安装 Internet Explorer 版本 10 或更高版本。这是 Health 服务通过使用你的 Azure 管理员凭据对你进行身份验证所必需的。
1. 有关在 Windows Server 2008 R2 上安装 Windows PowerShell 4.0 的其他信息，请参阅[此处](http://social.technet.microsoft.com/wiki/contents/articles/20623.step-by-step-upgrading-the-powershell-version-4-on-2008-r2.aspx)的 wiki 文章。

## 相关链接

* [Azure AD Connect Health](/documentation/articles/active-directory-aadconnect-health)
* [Azure AD Connect Health 操作](/documentation/articles/active-directory-aadconnect-health-operations)
* [在 AD FS 中使用 Azure AD Connect Health](/documentation/articles/active-directory-aadconnect-health-adfs)
* [Azure AD Connect Health 常见问题](/documentation/articles/active-directory-aadconnect-health-faq)

<!---HONumber=76-->