<properties
	pageTitle="保护 Azure AD 中的特权访问 | Azure"
	description="本主题介绍在 Azure、Azure Active Directory 和 Microsoft 在线服务中保护特许访问的方法。"
	services="active-directory"
	documentationCenter=""
	authors="kgremban"
	manager="femila"
	editor="mwahl"/>

<tags
	ms.service="active-directory"
	ms.date="07/14/2016"
	wacn.date="08/22/2016"/>


# 保护 Azure AD 中的特权访问

保护特权访问是帮助保护现代组织中的业务资产的首要步骤。组织中的大多数或所有业务资产的安全性取决于管理 IT 系统的特权帐户的完整性。网络攻击者往往以这些帐户为目标来获取组织数据和系统的访问权限。

若要保护管理访问权限以对抗顽固的对手，需要避免这些管理帐户和系统曝露于风险之中。越来越多的用户开始通过云服务获取特权访问。这包括 Office365 全局管理员、Azure 订阅管理员和拥有 VM 或 SaaS 应用管理访问权限的用户。

Microsoft 建议遵循 [Securing Privileged Access（保护特权访问）](https://technet.microsoft.com/library/mt631194.aspx)中的路线图。

对于使用 Azure Active Directory 来管理 Azure、Office 365 或其他 Microsoft 服务和应用程序访问权限的客户而言，是否可以应用这些原则将取决于用户帐户是由 Active Directory 进行管理和身份验证，还是在 Azure Active Directory 中进行管理和身份验证。以下部分提供了有关用于支持保护特权访问的 Azure AD 功能的详细信息。

## 多重身份验证

若要提高管理员身份验证的安全性，应在授予特权之前要求进行多重身份验证 (MFA)。MFA 是要求使用多种方式（而不仅仅是用户名和密码）对你的身份进行验证的一种方法。它为用户登录和事务提供了附加的安全层。

Azure Multi-Factor Authentication 可帮助保护对数据和应用程序的访问，同时可以满足用户对简单登录过程的需求。它通过各种简单的验证选项（包括电话呼叫、短信、移动应用通知或验证码）和第三方 OATH 令牌）来提供强式身份验证。


有关详细信息，请参阅 [MFA for Office 365 and MFA for Azure（对 Office 365 的 MFA 和对 Azure 的 MFA）](https://blogs.technet.microsoft.com/ad/2014/02/11/mfa-for-office-365-and-mfa-for-azure/)。

## 时间约束的特权

某些组织可能发现它们有太多拥有高特权角色的用户。用户可能因为某个特定活动（例如注册某个服务）而被添加到该角色，但之后却不经常使用这些特权。

若要减少特权的曝露时间并增大其使用的可见性，可将用户限制为只在需要执行某个任务时才适时使用 (JIT) 其特权。对于 Azure Active Directory 和 Microsoft 在线服务，可以使用 [Azure AD Privileged Identity Management (PIM)](http://aka.ms/AzurePIM)。


![PIM 仪表板][2]


## 攻击检测

Azure Active Directory Identity Protection 提供一个整合的视图来让你查看影响组织标识的风险事件和潜在漏洞。Identity Protection 根据风险事件计算每个用户的用户风险级别，让你配置基于风险的策略来自动保护组织的标识。将这些策略和 Azure Active Directory 与 EMS 提供的其他条件访问控制相结合，可以自动阻止用户，或者提供有关重置密码和强制实施多重身份验证等的建议。

![Azure AD Identity Protection][3]

## 条件性访问

借助条件性访问控制，Azure Active Directory 会在验证用户身份时先检查你选择的特定条件，然后才允许访问应用程序。一旦符合这些条件，用户就会通过身份验证并获权访问应用程序。


![设置使用 MFA 的条件性访问规则][4]


## 相关文章


- 启用 [Azure Multi-Factor Authentication](/documentation/articles/multi-factor-authentication-get-started-cloud/)



有关构建完整安全路线图的详细信息，请参阅 [Microsoft Cloud Security for Enterprise Architects（针对企业结构设计的 Microsoft 云安全性）](http://aka.ms/securecustomer)文档中的“Customer responsibilities and roadmap”（客户责任和路线图）部分。有关运用 Microsoft 服务来帮助实现其中任一主题所述功能的详细信息，请联系 Microsoft 代表或访问我们的[网络安全解决方案网页](https://www.microsoft.com/microsoftservices/campaigns/cybersecurity-protection.aspx)。

<!--Image references-->

[1]: ./media/active-directory-privileged-identity-management-configure/Search_PIM.png
[2]: ./media/active-directory-privileged-identity-management-configure/PIM_Dash.png
[3]: ./media/active-directory-identityprotection/29.png
[4]: ./media/active-directory-conditional-access/conditionalaccess-saas-apps.png

<!---HONumber=Mooncake_0815_2016-->