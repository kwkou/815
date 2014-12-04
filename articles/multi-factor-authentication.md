<properties linkid="" urlDisplayName="" pageTitle="" metaKeywords="" description="" metaCanonical="" services="" documentationCenter="" title="What is Azure Multi-Factor Authentication?" authors="billmath" solutions="" manager="terrylan" editor="lisatoft" />

# 什么是 Azure 多因素身份验证？

多因素身份验证或双因素身份验证是需要使用多个验证方法的身份验证方法，为用户登录和事务额外提供一层重要的安全保障。它需要以下验证方法中的两种或更多种来进行工作：

-   你知道的某物（通常为密码）
-   你具有的某物（无法轻易复制的可信设备，如电话）
-   你自身的特征（生物辨识系统）

多因素身份验证的安全性在于它的分层方法。破坏多因素身份验证系统对于攻击者来说是巨大的挑战。即使攻击者设法得到用户的密码，如果没有同时占有可信设备也没有用处。相反，如果用户不巧丢失了设备，捡到该设备的人也无法使用它，除非他或她知道用户的密码。
Azure 多因素身份验证是多因素身份验证服务，它要求用户使用移动应用程序、手机或短信验证登录。它可用于 Azure Active Directory 以通过 Azure 多因素身份验证服务器来保护本地资源安全，它还用于使用 SDK 的自定义应用程序和目录。

![Azure 多因素身份验证][Azure 多因素身份验证]

### 保护云 Azure Active Directory

为 Azure AD 标识启用多因素身份验证，系统将提示用户设置其下次登录时的额外验证。使用多因素身份验证可保护对 Azure、Microsoft Online Services（如 Office 365 和 Dynamics CRM Online）以及集成了 Azure AD 的第三方云服务的访问而无需进行其他设置。可以快速地为大量全球用户和应用程序启用多因素身份验证。[了解详细信息][了解详细信息]

### 保护本地资源和 Active Directory

使用 Azure 多因素身份验证服务器为本地资源（如 IIS 和 Active Directory）启用多因素身份验证。Azure 多因素身份验证服务器允许管理员与 IIS 身份验证集成，以保护 Microsoft IIS Web 应用程序，还可以与 RADIUS 身份验证、LDAP 身份验证和 Windows 身份验证集成。[了解详细信息][1]

### 保护自定义应用程序

</p>
SDK 支持与你的云服务的直接集成。将 Active Authentication 电话呼叫和短信验证内置于应用程序的登录或事务过程中，并利用应用程序的现有用户数据库。[了解详细信息][2]

### 适用于 Office 365 的多因素身份验证

适用于 Office 365 的多因素身份验证采用 Azure 多因素身份验证技术，专用于 Office 365 应用程序，并可通过 Office 365 门户进行管理。因此，管理员现在可以使用多因素身份验证来保护其 Office 365 资源。[了解详细信息][3]

### 面向 Azure 管理员的多因素身份验证

适用于 Office 365 的多因素身份验证功能的相同子集将免费提供给所有 Azure 管理员使用。Azure 订阅的每个管理员帐户现在可以通过启用这项核心多因素身份验证功能来获得更多的保护。因此，如果某个管理员想要访问 Azure 门户以创建 VM 和网站以及管理存储、移动服务或任何其他 Azure 服务，则可在其管理员帐户中添加多因素身份验证。[了解详细信息][4]

### 多因素身份验证功能比较

下面显示了可用的多因素身份验证版本，以及各版本所提供的功能的摘要。你可以参考这些信息来确定适合你自己的多因素身份验证版本。[了解详细信息][4]

![Azure 多因素身份验证功能比较][Azure 多因素身份验证功能比较]

**其他资源**

-   [作为组织注册 Azure][作为组织注册 Azure]
-   [Azure 标识][Azure 标识]
-   [Azure 多因素身份验证库][Azure 多因素身份验证库]

  [Azure 多因素身份验证]: ./media/multi-factor-authentication/WAMFA1.png
  [了解详细信息]: http://msdn.microsoft.com/en-us/library/dn249466.aspx
  [1]: http://msdn.microsoft.com/en-us/library/dn249467.aspx
  [2]: http://msdn.microsoft.com/en-us/library/dn249464.aspx
  [3]: http://msdn.microsoft.com/en-us/library/dn383636.aspx
  [4]: http://msdn.microsoft.com/en-us/library/dn249471.aspx
  [Azure 多因素身份验证功能比较]: ./media/multi-factor-authentication/mfacomparison1.png
  [作为组织注册 Azure]: /en-us/manage/services/identity/organizational-account/
  [Azure 标识]: /en-us/manage/windows/fundamentals/identity/
  [Azure 多因素身份验证库]: http://technet.microsoft.com/en-us/library/dn249471.aspx
