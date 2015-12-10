<properties 
	pageTitle="使用 Azure MFA 时的安全最佳实践" 
	description="本文档提供有关配合使用 Azure MFA 与 Azure 帐户的最佳实践" 
	services="multi-factor-authentication" 
	documentationCenter="" 
	authors="billmath" 
	manager="stevenpo" 
	editor="curtland"/>

<tags 
	ms.service="multi-factor-authentication" 
	ms.date="10/15/2015" 
	wacn.date="11/12/2015"/>

# 将 Azure Multi-Factor Authentication 与 Azure AD 帐户配合使用时的安全最佳实践

在需要增强身份验证过程时，Multi-Factor Authentication 是大多数组织的首选。Azure Multi-Factor Authentication (MFA) 能使公司符合其安全和法规遵从要求，同时为用户提供简单的登录体验。本文介绍当你计划采用 Azure MFA 时应该考虑的一些最佳实践。

## 云中 Azure Multi-Factor Authentication 的最佳实践
为了对所有用户提供 Multi-Factor Authentication 并利用 Azure Multi-factor Authentication 的扩展功能，你需要为用户启用 Azure Multi-Factor Authentication。


### 用户帐户
为用户启用 MFA 时，用户帐户可以处于三种核心状态之一：已禁用、已启用或强制。 
使用以下指导原则来确保为部署使用最适当的选项：

- 当用户状态设置为已禁用时，该用户将不使用 Multi-Factor Authentication。这是默认状态。
- 当用户状态设置为已启用时，表示该用户已启用但尚未完成注册过程。这些用户在下次登录时，系统将提示其完成注册过程。此设置不会影响非浏览器应用。所有应用将继续工作，直到注册过程完成。
- 当用户状态设置为强制时，表示该用户可能（但不一定）已完成注册。如果他们已完成注册过程，表示他们正在使用 Multi-Factor Authentication。否则，在他们下次登录时，系统将提示其完成注册过程。此设置不会影响非浏览器应用。在创建并使用应用密码之前，这些应用无法工作。
- 使用[云中的 Azure Multi-Factor Authentication 入门](multi-factor-authentication-get-started-cloud.md)一文中所述的用户通知模板，向用户发送有关采用 MFA 的电子邮件。

### 可支持性

由于大多数用户习惯只使用密码进行身份验证，因此你的公司必须让所有用户了解此过程。如果用户熟悉该过程，则他们就不会在出现 MFA 相关的小问题时经常呼叫技术支持。 
但是，在某些情况下，需要暂时禁用 MFA。使用以下指导原则了解如何处理这种情况：

- 确保你的技术支持人员经过培训，可以处理移动应用或电话未收到通知或来电，以及由于上述原因使用户无法登录的情况。他们可以启用“一次性跳过”选项，让用户通过“跳过”Multi-Factor Authentication 来进行身份验证，不过只能跳过一次。跳过是暂时性的，将在指定的秒数后过期。 
- 如果需要，可以使用 Azure MFA 中的受信任 IP 功能。此功能允许托管或联合租户的管理员跳过对从公司本地 Intranet 登录的用户进行的 Multi-Factor Authentication。这些功能适用于拥有 Azure AD Premium、Enterprise Mobility Suite 或 Azure Multi-Factor Authentication 许可证的 Azure AD 租户。


## 本地 Azure Multi-Factor Authentication 的最佳实践
如果你的公司决定利用自己的基础结构来启用 MFA，则需要在本地部署 Azure Multi-factor Authentication 服务器。下图显示了 MFA 服务器的组件：

![Multi-Factor Auth 提供程序](./media/multi-factor-authentication-security-best-practices/server.png)
*默认未安装 **已安装但默认未启用


Azure Multi-Factor Authentication 服务器可用于保护 Azure AD 帐户所访问的云资源和本地资源。但是，这只能使用联合身份验证来实现。也就是说，你必须安装 AD FS 并将它与 Azure AD 租户联合。
设置 Multi-Factor Authentication 服务器时，请注意以下事项：

- 如果你使用 Active Directory 联合身份验证服务来保护 Azure AD 资源，则第一重身份验证将使用 AD FS 在本地执行，第二重身份验证遵循声明在本地执行。
- Azure Multi-Factor Authentication 服务器不一定非要安装在 AD FS 联合服务器上，但必须在运行 AD FS 的 Windows Server 2012 R2 上安装适用于 AD FS 的 Multi-Factor Authentication 适配器。你可以将服务器安装在其他计算机上（只要它是受支持的版本），并将 AD FS 适配器单独安装在 AD FS 联合服务器上。有关如何单独安装适配器的说明，请参阅以下过程。
- Multi-Factor Authentication AD FS 适配器安装向导在 Active Directory 中创建名为 PhoneFactor 管理员的安全组，并将你的联合服务的 AD FS 服务帐户添加到该组中。建议你在域控制器上确认 PhoneFactor 管理员组已真正创建，而且 AD FS 服务帐户是该组的成员。需要时，在域控制器上手动将 AD FS 服务帐户添加到 PhoneFactor 管理员组。

### 用户门户
此门户在 Internet Information Server (IIS) 网站中运行，支持自助功能并提供全面的用户管理功能。请使用以下指导原则来配置此组件：

- 需要 IIS 6 或更高版本
- 必须安装并注册 ASP.NET v2.0.507207
- 此服务器可以部署在外围网络中。
- 如果此服务器与 Azure 之间的通信实施防火墙过滤，则需要打开 TCP 出站端口 443，以允许使用以下 URL 通信：
	- https://pfd.phonefactor.net 
	- https://pfd2.phonefactor.net 
	- https://css.phonefactor.net
- 如果端口 443 上限制了出站防火墙，则需要允许以下 IP 地址范围的出站流量：
	- 134\.170.116.0/25
	- 134\.170.165.0/25
	- 70\.37.154.128/25



### 应用密码
如果你的组织使用 SSO 与 Azure AD 的联合，并且你想要使用 Azure MFA，则在使用应用密码时，请注意以下事项（请记住，这只适用于使用联合 (SSO) 的情况）：

- 应用密码由 Azure AD 进行验证，从而绕过了联合。仅在设置应用密码时，才会使用联合。
- 联合 (SSO) 用户密码将存储在组织 ID 中。如果用户离开公司，则该信息必须通过 DirSync 实时流到组织 ID 中。帐户禁用/删除可能需要长达 3 小时才能同步，从而延迟了 Azure AD 中应用密码的禁用/删除。
- 应用密码不遵循“本地客户端访问控制”设置
- 本地身份验证日志记录/审核功能不可用于应用密码
- 对于 Microsoft Lync 2013 客户端，更多最终用户培训是必需的。 
- 在对客户端使用 Multi-Factor Authentication 时，某些先进的体系结构设计可能需要将组织用户名和密码与应用密码结合使用，具体取决于进行身份验证的位置。对于针对本地基础结构进行身份验证的客户端，你会使用组织用户名和密码。对于针对 Azure AD 进行身份验证的客户端，你会使用应用密码。
- 默认情况下，用户无法创建应用密码。如果在某些情况下，你的公司要求或者你需要允许用户创建应用密码，请确保选中“允许用户创建用于登录到非浏览器应用程序的应用密码”选项。

## 其他注意事项
使用以下列表了解本地部署的每个组件的其他一些注意事项和最佳实践：

方法|说明
:------------- | :------------- | 
[Active Directory 联合身份验证服务](multi-factor-authentication-get-started-adfs.md)|有关设置使用 AD FS 的 Azure Multi-Factor Authentication 的信息。
[RADIUS 身份验证](multi-factor-authentication-get-started-server-radius.md)| 有关设置和配置使用 RADIUS 的 Azure MFA 服务器的信息。
[IIS 身份验证](multi-factor-authentication-get-started-server-iis.md)|有关设置和配置使用 IIS 的 Azure MFA 服务器的信息。
[Windows 身份验证](multi-factor-authentication-get-started-server-windows.md)| 有关设置和配置使用 Windows 身份验证的 Azure MFA 服务器的信息。
[LDAP 身份验证](multi-factor-authentication-get-started-server-ldap.md)|有关设置和配置使用 LDAP 身份验证的 Azure MFA 服务器的信息。
[使用 RADIUS 的远程桌面网关和 Azure Multi-Factor Authentication 服务器](multi-factor-authentication-get-started-server-rdg.md)| 有关设置和配置使用 RADIUS 的、具有远程桌面网关的 Azure MFA 服务器的信息。
[与 Windows Server Active Directory 同步](multi-factor-authentication-get-started-server-dirint.md)|有关在 Active Directory 与 Azure MFA 服务器之间设置和配置同步的信息。
[部署 Azure Multi-Factor Authentication 服务器移动应用 Web 服务](multi-factor-authentication-get-started-server-webservice.md)|有关设置和配置 Azure MFA 服务器 Web 服务的信息。
[使用 Azure Multi-Factor Authentication 的高级 VPN 配置](multi-factor-authentication-advanced-vpn-configurations.md)|有关使用 LDAP 或 RADIUS 配置 Cisco ASA、Citrix Netscaler 和 Juniper/Pulse Secure VPN 设备的信息。 


## 其他资源
尽管本文重点介绍了 Azure MFA 的一些最佳实践，但其他一些资源也可以帮助你规划 MFA 的部署。以下列表提供了在此过程中也许能够帮到你的一些重要文章：

- [Azure Multi-Factor Authentication 中的报告](multi-factor-authentication-manage-reports.md)
- [Azure Multi-Factor Authentication 的设置体验](multi-factor-authentication-end-user-first-time.md)
- [Azure Multi-Factor Authentication 常见问题](multi-factor-authentication-faq.md)

<!---HONumber=82-->