<properties
	pageTitle="Azure AD Connect：开始使用快速设置 | Azure"
	description="了解如何下载、安装和运行 Azure AD Connect 的设置向导。"
	services="active-directory"
	documentationCenter=""
	authors="andkjell"
	manager="stevenpo"
	editor="curtand"/>

<tags
	ms.service="active-directory"
	ms.date="05/26/2016"
	wacn.date="07/18/2016"/>

# 通过快速设置开始使用 Azure AD Connect
当你有单林拓扑和用于身份验证的[密码同步](active-directory-aadconnectsync-implement-password-synchronization/)时，便可使用 Azure AD Connect **快速设置**。**快速设置**是默认选项，用于最常见的部署方案。只需按几下鼠标即可将本地目录扩展到云中。

在开始安装 Azure AD Connect 之前，请确保[下载 Azure AD Connect](http://go.microsoft.com/fwlink/?LinkId=615771) 并完成 [Azure AD Connect：硬件和先决条件](active-directory-aadconnect-prerequisites.md)中的预先准备步骤。

如果快速设置与拓扑不匹配，请参阅[相关文档](#related-documentation)中的其他方案。


## Azure AD Connect 的快速安装


1. 以本地管理员身份登录到要安装 Azure AD Connect 的服务器。应该在要用作同步服务器的服务器上执行此操作。
2. 导航到 **AzureADConnect.msi** 并双击它。
3. 在“欢迎”屏幕上，选中对应的框，同意许可条款，然后单击“继续”。
4. 在“快速设置”屏幕上，单击“使用快速设置”。  
![欢迎使用 Azure AD Connect](./media/active-directory-aadconnect-get-started-express/express.png)
5. 在“连接到 Azure AD”屏幕上，输入 Azure AD 的全局管理员用户名和密码。单击**“下一步”**。  
![连接到 Azure AD](./media/active-directory-aadconnect-get-started-express/connectaad.png)
如果你收到错误消息并且出现了连接问题，请参阅[排查连接问题](/documentation/articles/active-directory-aadconnect-troubleshoot-connectivity)。
6. 在“连接到 AD DS”屏幕上，输入企业管理员帐户的用户名和密码。可以使用 NetBios 或 FQDN 格式输入域部分，即 FABRIKAM\\administrator 或 fabrikam.com\\administrator。单击“下一步”。  
![连接到 AD DS](./media/active-directory-aadconnect-get-started-express/connectad.png)
7. 只有在未完成[必备组件](/documentation/articles/active-directory-aadconnect-prerequisites)中的[验证域](active-directory-add-domain.md)时，才会显示[“Azure AD 登录配置”](/documentation/articles/active-directory-aadconnect-user-signin#azure-ad-sign-in-configuration)页面。
![未验证的域](./media/active-directory-aadconnect-get-started-express/unverifieddomain.png)  
如果你看到此页，请查看标记为“未添加”和“未验证”的每个域。确保使用的域都已在 Azure AD 中验证。验证域后，请单击“刷新”符号。有关详细信息，请参阅[添加和验证域](active-directory-add-domain.md)
8. 在“已准备好配置”屏幕上，单击“安装”。
	- 在“已准备好配置”页上，可以取消选中“配置完成后立即开始同步过程”复选框。如果想要进行其他配置（例如[筛选](/documentation/articles/active-directory-aadconnectsync-configure-filtering)），应取消选中此复选框。如果取消选择此选项，向导将配置同步，但会保持禁用计划程序。在[重新运行安装向导](active-directory-aadconnectsync-installation-wizard.md)以手动启用计划程序之前，计划程序不会运行。
	- 如果本地 Active Directory 中有 Exchange，也可以选择启用 [**Exchange 混合部署**](https://technet.microsoft.com/library/jj200581.aspx)。如果你打算同时在云中和本地设置 Exchange 邮箱，请启用此选项。
![已准备好配置 Azure AD Connect](./media/active-directory-aadconnect-get-started-express/readytoconfigure.png)
9. 安装完成后，单击“退出”。
10. 安装完成后，请注销并再次登录，然后即可使用同步服务管理器或同步规则编辑器。

有关使用快速安装的视频，请参阅：



## 后续步骤
安装 Azure AD Connect 后，可以[验证安装并分配许可证](/documentation/articles/active-directory-aadconnect-whats-next/)。

了解有关[将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect/)的详细信息。

## 相关文档

主题 |  
--------- | ---------
Azure AD Connect 概述 | [将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect)
使用自定义设置安装 | [Azure AD Connect 的自定义安装](/documentation/articles/active-directory-aadconnect-get-started-custom)
从 DirSync 升级 | [从 Azure AD 同步工具 (DirSync) 升级](/documentation/articles/active-directory-aadconnect-dirsync-upgrade-get-started)
用于安装的帐户 | [有关 Azure AD Connect 帐户和权限的详细信息](/documentation/articles/active-directory-aadconnect-accounts-permissions)

<!---HONumber=Mooncake_0711_2016-->