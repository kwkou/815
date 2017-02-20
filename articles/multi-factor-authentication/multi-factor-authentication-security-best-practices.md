<properties
    pageTitle="MFA 最佳安全做法 | Azure"
    description="本文档提供有关配合使用 Azure MFA 与 Azure 帐户的最佳实践"
    services="multi-factor-authentication"
    documentationcenter=""
    author="kgremban"
    manager="femila"
    editor="yossib" />  

<tags
    ms.assetid="3be7d968-96bb-4320-8701-869fd04a2595"
    ms.service="multi-factor-authentication"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="10/31/2016"
    wacn.date="02/17/2017"
    ms.author="kgremban" />

# 将 Azure 多重身份验证与 Azure AD 帐户配合使用时的安全最佳实践
在增强身份验证过程时，双重验证是大多数组织的首选。Azure 多重身份验证 (MFA) 能使公司符合其安全和法规遵从要求，同时为用户提供简单的登录体验。本文介绍计划采用 Azure MFA 时应该考虑的一些最佳做法。

## 云中 Azure 多重身份验证的最佳实践
若要为所有用户都提供双重验证，并利用 Azure MFA 提供的扩展功能，需要为所有用户启用 Azure MFA。可使用下列实现之一来完成此操作：

- Multi-Factor Auth 提供程序

### Multi-Factor Auth 提供程序
![Multi-Factor Auth 提供程序](./media/multi-factor-authentication-security-best-practices/authprovider.png)  


如果没有 Azure AD Premium 或企业移动性套件，则在云中采用 Azure MFA 的第一个步骤是创建 MFA 身份验证提供程序。尽管 MFA 默认可供拥有 Azure Active Directory 的全局管理员使用，但为组织部署 MFA 时，需要将双重验证功能扩展到所有用户。为此，需要多重身份验证提供程序。选择 Auth 提供程序时，需要选择一个目录并注意以下事项：

- 无需 Azure AD 目录即可创建 Multi-Factor Auth 提供程序。但如果想使用以下功能，需要将多重身份验证提供程序与 Azure AD 目录相关联：
  - 将双重验证扩展到所有用户
  - 为全局管理员提供其他功能，例如管理门户、自定义问候语和报告。
- 仅当要将本地 Active Directory 环境与 Azure AD 目录同步时，才需要 DirSync 或 AAD 同步。如果使用不与 Active Directory 的本地实例同步的 Azure AD 目录，则不需要 DirSync 或 AAD 同步。
- 请确保为企业选择适当的使用模型（基于身份验证或基于启用的用户）。选择使用模型后，无法对其进行更改。
  - 基于身份验证：按每次验证收费。如果想要对访问特定应用的用户，而不是对特定用户使用双重验证，请使用此模型。
  - 基于允许使用的用户：按允许使用 Azure MFA 的每名用户收费。如果有且仅有部分用户拥有 Azure AD Premium 或企业移动性套件许可证，请使用此模型。

### 用户帐户
为用户启用 Azure MFA 时，用户帐户可以处于三种核心状态之一：已禁用、已启用或强制。
使用以下指导原则，确保对部署使用最适当的选项：

- 用户状态设置为“已禁用”时，该用户不执行双重验证。这是默认状态。
- 用户状态设置为“已启用”时，表示该用户可使用双重验证，但尚未完成注册过程。这些用户在下次登录时，系统将提示其完成注册过程。此设置不会影响非浏览器应用。所有应用继续工作，直到注册过程完成。
- 用户状态设置为“强制”时，表示该用户可能（但不一定）已完成注册。如果他们已完成注册过程，则表示他们正在执行执行双重验证。否则，在他们下次登录时，系统将提示其完成注册过程。此设置不会影响非浏览器应用。在创建并使用应用密码之前，这些应用无法工作。

使用[云中的 Azure 多重身份验证入门](/documentation/articles/multi-factor-authentication-get-started-cloud/)一文中所述的用户通知模板，向用户发送有关采用 MFA 的电子邮件。

### 可支持性
由于大多数用户习惯只使用密码进行身份验证，因此公司必须让所有用户了解此过程。如果用户熟悉该过程，则他们就不会在出现 MFA 相关的小问题时经常呼叫技术支持。但是，在某些情况下，需要暂时禁用 MFA。使用以下指导原则了解如何处理这些情况：

- 确保技术支持人员经过培训，可以处理移动应用或手机未收到通知或来电，以及由于上述原因使用户无法登录的情况。技术支持可启用“一次性跳过”选项，让用户通过“跳过”双重验证来进行身份验证，不过只能跳过一次。跳过是暂时性的，将在指定的秒数后过期。

## 本地 Azure 多重身份验证的最佳实践
如果公司决定利用自己的基础结构来启用 MFA，则需要在本地部署 Azure 多重身份验证服务器。下图显示了 MFA 服务器组件：

![Multi-Factor Auth 提供程序](./media/multi-factor-authentication-security-best-practices/server.png) 
\*默认未安装 \**已安装但默认未启用

Azure 多重身份验证服务器可用于保护 Azure AD 帐户所访问的云资源和本地资源。但是，这只能使用联合身份验证来实现。也就是说，你必须安装 AD FS 并将它与 Azure AD 租户联合。设置多重身份验证服务器时，请注意以下事项：

- 如果使用 Active Directory 联合身份验证服务来保护 Azure AD 资源，则第一重身份验证将使用 AD FS 在本地执行。第二重身份验证遵循声明在本地执行。
- 不要求在 AD FS 联合服务器上安装 Azure 多重身份验证服务器。但必须在运行 AD FS 的 Windows Server 2012 R2 上安装用于 AD FS 的多重身份验证适配器。可将服务器安装在其他计算机上（只要它是受支持的版本），并将 AD FS 适配器单独安装在 AD FS 联合服务器上。有关如何单独安装适配器的说明，请参阅以下过程。
- 多重身份验证 AD FS 适配器安装向导将在 Active Directory 中创建名为 PhoneFactor Admins 的安全组，然后将 AD FS 服务帐户添加到此组。建议在域控制器上检查是否确实创建了 PhoneFactor Admins 组，以及 AD FS 服务帐户是否是此组的成员。需要时，在域控制器上手动将 AD FS 服务帐户添加到 PhoneFactor 管理员组。

### 用户门户
此门户在 Internet Information Server (IIS) 网站中运行，支持自助功能并提供全面的用户管理功能。请使用以下指导原则来配置此组件：

- 需要 IIS 6 或更高版本
- 必须安装并注册 ASP.NET v2.0.507207
- 此服务器可以部署在外围网络中。

## 其他资源
尽管本文重点介绍了 Azure MFA 的一些最佳实践，但其他一些资源也可以帮助你规划 MFA 的部署。以下列表提供了在此过程中也许能够帮到你的一些重要文章：

- [Azure 多重身份验证的设置体验](/documentation/articles/multi-factor-authentication-end-user-first-time/)
- [Azure 多重身份验证常见问题](/documentation/articles/multi-factor-authentication-faq/)

<!---HONumber=Mooncake_Quality_Review_0125_2017-->
