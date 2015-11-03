<properties 
	pageTitle="Azure AD Connect - 用户登录" 
	description="Azure AD Connect 用户登录的自定义设置。" 
	services="active-directory" 
	documentationCenter="" 
	authors="billmath" 
	manager="stevenpo" 
	editor="curtand"/>

<tags 
	ms.service="active-directory"  
	ms.date="08/24/2015" 
	wacn.date="11/02/2015"/>



# Azure AD Connect 用户登录选项

Azure AD Connect 可让用户使用同一组密码登录云和本地资源。你可以从多个不同方式中选择启用此设置的方式。


### 密码同步
使用密码同步可将用户密码的哈希从本地 Active Directory 同步到 Azure AD。当密码在本地更改或重置时，新密码将立即同步到 Azure AD，使用户用来访问云资源的密码始终与本地相同。密码绝不将发送到 Azure AD，也不会以明文存储在 Azure AD 中。
可将密码同步与密码写回一起使用，以便在 Azure AD 中启用自助密码重置。

<center>![云](./media/active-directory-aadconnect-user-signin/passwordhash.png)</center>

[有关密码同步的详细信息](https://msdn.microsoft.com/zh-cn/library/azure/dn246918.aspx)


### 使用 Windows Server 2012 R2 场中新的或现有 AD FS 的联合
通过联合登录，用户可以使用本地密码登录基于 Azure AD 服务；使用公司网络时，无需再次输入密码就可登录服务。使用 AD FS 的联合选项可让你在 Windows Server 2012 R2 场中部署新的或指定现有的 AD FS。如果你选择指定现有的场，Azure AD Connect 将在场与 Azure AD 之间配置信任，使用户能够登录。

<center>![云](./media/active-directory-aadconnect-user-signin/federatedsignin.png)</center>

#### 若要在 Windows Server 2012 R2 中部署使用 AD FS 的联合身份验证，你需要以下各项
如果你要部署新场：

- 用于联合服务器的 Windows Server 2012 R2 服务器。
- 用于 Web 应用程序代理的 Windows Server 2012 R2 服务器。
- 一个 .pfx 文件，其中包含目标联合服务名称（例如 fs.contoso.com）的 SSL 证书。

如果你要部署新场或使用现有场：

- 联合服务器上的本地管理员凭据。
- 要部署 Web 应用程序代理角色的任何工作组（未加入域）服务器上的本地管理员凭据。
- 执行向导的计算机必须能够通过 Windows 远程管理连接到要安装 AD FS 或 Web 应用程序代理的任何其他计算机。

#### 使用早期版本的 AD FS 或第三方解决方案登录
如果你已使用早期版本的 AD FS（例如 AD FS 2.0）或第三方联合身份验证提供者配置了云登录，则可以通过 Azure AD Connect 选择跳过用户登录配置。这样，你便可以获取最新的同步和 Azure AD Connect 的其他功能，同时仍可使用现有的解决方案进行登录。

### 为你的组织选择用户登录方法
由于大多数组织只想让用户登录 Office 365、SaaS 应用程序和其他基于 Azure AD 的资源，因此建议使用默认的密码同步选项。然而，某些组织出于特定的原因而需要使用联合登录选项（例如 AD FS）。其中包括：

- 组织已部署 AD FS 或第三方联合身份验证提供者
- 安全策略禁止将密码哈希同步到云
- 要求用户从公司网络访问已加入域的计算机中的云资源时，使用无缝 SSO（不显示附加的密码提示）
- 你需要一些特定的 AD FS 功能
	- 使用第三方提供者或智能卡的本地 Multi-Factor Authentication（了解 Windows Server 2012 R2 中适用于 AD FS 的第三方 MFA 提供者）
	- Active Directory 集成功能，例如软帐户锁定或 AD 密码及工作时数策略
	- 使用设备注册、Azure AD 联接或 Intune MDM 策略对本地资源和云资源进行条件性访问
 

<!---HONumber=67-->