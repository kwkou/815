<properties 
	pageTitle="Azure AD 术语 | Microsoft Azure"
	description="与 Azure AD 相关的术语和定义。" 
	services="active-directory" 
	documentationCenter="" 
	authors="curtand"
	manager="stevenpo"
	editor=""/>

<tags 
	ms.service="active-directory" 
	ms.date="04/26/2016"
	wacn.date="06/27/2016"/>

# Azure AD 术语

Microsoft Azure Active Directory (Azure AD) 具有一套独特的有关云方案、混合方案和本地方案的术语。下表定义了这些术语，让你对于这些术语将在本指南的各个主题中如何使用有个基本的了解。

 术语 | 定义
------------- | -------------
其他安全性验证 | 全局管理员可对组织目录中的用户帐户设置的一种安全设置，该设置要求用户使用密码并在电话中做出答复来通过 Azure Active Directory 身份验证系统进行身份验证。
Azure Active Directory | Azure 中的标识服务，它通过基于 REST 的 API 提供标识管理和访问控制功能。
Azure Active Directory 访问控制 | Azure 服务，用于为 REST Web 服务提供联合身份验证和规则驱动的基于声明的授权。
Azure Active Directory 身份验证系统 | Microsoft 在云中的标识服务，用于对工作或学校帐户进行身份验证和授权。
Azure Active Directory Graph | Azure Active Directory 的一项功能，用于访问社会企业图中的用户、组和角色对象，可轻松找到并显示用户信息和关系。
用于 Windows PowerShell 的 Azure Active Directory 模块 | 一组用于管理 Azure Active Directory 的 cmdlet。你可以使用这些 cmdlet 来管理用户、组、域、云服务订阅、许可证、目录同步、单一登录等。
Azure Active Directory Connect | Azure Active Directory Connect 向导是用于将本地目录与 Azure Active Directory 进行连接的单一工具，它提供了引导式体验。该向导将部署并配置建立和运行目录集成所需的所有组件，包括同步服务、密码同步或 Active Directory 联合身份验证服务 (AD FS)，以及 Azure AD PowerShell 模块等必备组件。
Azure Active Directory 同步工具 | 用于将公司本地 Active Directory 服务中的目录对象单向同步到 Azure Active Directory 的应用程序。
目录集成 | Azure Active Directory 的一项功能，你可以设置该功能以改进与本地目录和云目录中的身份维护相关联的管理体验。目录集成方案包括目录同步，以及通过单一登录进行的目录同步。
目录同步 | 用于将本地目录对象（用户、组、联系人）同步到云中，以帮助减少管理开销。目录同步在英文版 Azure AD 门户预览和 Azure 经典管理门户中也简写为 Directory Sync。设置了目录同步后，管理员可以将本地 Active Directory 中的目录对象设置为同步到 Azure AD 实例。
Microsoft Online Services 登录助手 | 此登录助手是安装在客户端计算机上的一个应用程序，它使用户只需在该计算机上登录一次即可在登录会话期间访问服务任意多次。如果没有此登录助手，最终用户必须在每次尝试访问服务时都提供用户名和密码。不应将此登录助手与单一登录混淆，后者是 Azure Active Directory 的目录集成功能，可以在部署后利用用户的现有本地企业凭据无缝访问 Microsoft 云服务。
多重身份验证（也称为双因素身份验证或 2FA） | 多重身份验证为用户登录和进行事务处理添加了关键的第二层安全性。当你为 Azure AD 中的用户帐户启用多重身份验证后，该用户在每次需要登录并使用你组织订阅的任何 Microsoft 云服务时，除了标准密码凭据外，还必须使用自己的手机作为附加安全验证方法。
单一登录 | 用于为用户提供更加无缝的身份验证体验，以便在登录到企业网络的情况下访问 Microsoft 云服务。为了设置单一登录，组织需要在本地部署安全令牌服务。设置单一登录后，用户可以使用其 Active Directory 企业凭据（用户名和密码），访问云中的服务及其现有的本地资源。
用户 ID | 用户 ID 是用户在“登录”页上提供的唯一标识符，用于访问组织已订阅的 Microsoft 云服务。
工作或学校帐户 | 组织（工作、学校、非营利单位）为其成员（员工、学生、客户）分配的用户帐户，该帐户提供到组织的一个或多个 Microsoft 云服务订阅（如 Office 365 或 Azure）的登录访问权限。这些帐户存储在组织的云目录（也称为 Azure Active Directory）中，通常会在用户离开组织时删除。工作或学校帐户不同于 Microsoft 帐户，因为它们由组织中的管理员（而非用户）创建和管理。 

## 后续步骤
- [以组织身份注册 Azure](/documentation/articles/sign-up-organization/)
- [Azure 订阅与 Azure AD 的关联方式](/documentation/articles/active-directory-how-subscriptions-associated-directory/)
- [Azure AD 服务限制和局限性](/documentation/articles/active-directory-service-limits-restrictions/)



<!---HONumber=Mooncake_0620_2016-->