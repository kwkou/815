<properties
    pageTitle="有关 Azure Active Directory 中应用程序管理的文章索引 | Azure"
    description="了解如何自定义联合证书的过期日期，以及如何续订即将过期的证书。"
    services="active-directory"
    documentationcenter=""
    author="MarkusVi"
    manager="femila" />
<tags
    ms.assetid="5321b8e4-2afa-4dfe-8d53-4add7abb5ec8"
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/10/2017"
    wacn.date="02/07/2017"
    ms.author="markvi" />

# 有关 Azure Active Directory 中应用程序管理的文章索引
本页提供了一份完整列表，列出编写的 Azure Active Directory (Azure AD) 中各种应用程序相关功能的所有文章。

每个主要功能区都有简介，同时也根据你要查找的信息指导需要阅读的文章。

## 概述文章
对于只需要 Azure AD 应用程序管理功能的简要说明的用户，以下文章是很好的起点。

| 文章指南 | |
|:---:| --- |
| 介绍 Azure AD 解决的应用程序管理问题 |[使用 Azure Active Directory (AD) 管理应用程序](/documentation/articles/active-directory-enable-sso-scenario/) |
| Azure AD 中与启用单一登录、定义有权访问应用的人员，以及用户如何启动应用相关的各种功能概述 |[Azure Active Directory 中的应用程序访问和单一登录](/documentation/articles/active-directory-appssoaccess-whatis/) |
| 探讨将应用集成到 Azure AD 时所涉及的不同步骤 |[将 Azure Active Directory 与应用程序集成](/documentation/articles/active-directory-integrating-applications-getting-started/)<br /><br />[启用 SaaS 应用的单一登录](/documentation/articles/active-directory-sso-integrate-saas-apps/)<br /><br />[管理对应用的访问](/documentation/articles/active-directory-managing-access-to-apps/) |
| 如何在 Azure AD 中表示应用的技术说明 |[如何及为何将应用程序添加到 Azure AD](/documentation/articles/active-directory-how-applications-are-added/) |

## 疑难解答文章
本部分提供相关故障排除指南的快速访问链接。可以在本页的余下部分找到有关每个功能区的详细信息。

| 功能区 | |
|:---:| --- |
| 联合单一登录 |[基于 SAML 的单一登录疑难解答](/documentation/articles/active-directory-saml-debugging/) |
| 基于密码的单一登录 | Internet Explorer 访问面板扩展疑难解答 |
| 本地 AD 与 Azure AD 之间的单一登录 |[密码同步疑难解答](/documentation/articles/active-directory-aadconnectsync-implement-password-synchronization/#troubleshooting-password-synchronization/)<br /><br />[密码写回疑难解答](/documentation/articles/active-directory-passwords-troubleshoot/#troubleshoot-password-writeback/) |

## 单一登录 (SSO)
### 联合单一登录：使用一个标识登录多个应用程序
单一登录可让用户只使用一组凭据，访问各种不同的应用程序和服务。可以通过联合方法启用单一登录。当用户尝试登录联合应用时，将被重定向到其组织的官方登录页面（由 Azure Active Directory 呈现），一旦身份验证成功，即重定向回到应用。

| 文章指南 | |
|:---:| --- |
| 联合身份验证和其他登录类型简介 |[使用 Azure AD 进行单一登录](/documentation/articles/active-directory-appssoaccess-whatis/) |
| 通过简化的单一登录配置步骤与 Azure AD 预先集成的数千个 SaaS 应用 |[Azure AD 应用程序库入门](/documentation/articles/active-directory-appssoaccess-whatis/#get-started-with-the-azure-ad-application-gallery/)<br /><br />[支持联合身份验证的预先集成应用完整列表](http://aka.ms/aadfederatedapps)<br /><br />[如何将应用添加到 Azure AD 应用库](/documentation/articles/active-directory-app-gallery-listing/) |
| 150 个以上的应用教程，介绍如何为应用配置单一登录等内容 | |
| 如何手动设置和自定义单一登录配置 |如何为不在 Azure Active Directory 应用程序库中的应用配置联合单一登录<br /><br />[如何针对预先集成的应用自定义在 SAML 令牌中颁发的声明](/documentation/articles/active-directory-saml-claims-customization/) |
| 使用 SAML 协议的联合应用的故障排除指南 |[排查基于 SAML 的单一登录问题](/documentation/articles/active-directory-saml-debugging/) |
| 如何设置应用的证书过期日期，以及如何续订证书 |[在 Azure Active Directory 中管理用于联合单一登录的证书](/documentation/articles/active-directory-sso-certs/) |

联合单一登录适用于所有版本的 Azure AD，每个用户最多十个应用。[Azure AD 高级版](/pricing/details/identity/)支持无限数目的应用程序。如果组织拥有 [Azure AD 基本版](/pricing/details/identity/)或 [Azure AD 高级版](/pricing/details/identity/)，则可以使用组来分配对联合应用程序的访问权限。

### 基于密码的单一登录：非联合应用的帐户共享和 SSO
为了实现单一登录到不支持联合身份验证的应用程序，Azure AD 提供了密码管理功能，可安全地将密码存储到 SaaS 应用并自动将用户登录到这些应用。你可以轻松分发新建帐户的凭据，并与多人共享团队帐户。用户无需知道他们有权访问的帐户凭据。

| 文章指南 | |
|:---:| --- |
| 基于密码的 SSO 工作原理简介以及简要的技术概述 |[使用 Azure AD 进行基于密码的单一登录](/documentation/articles/active-directory-appssoaccess-whatis/#password-based-single-sign-on/) |
| 与共享帐户相关的方案以及 Azure AD 如何解决这些问题的摘要 |[使用 Azure AD 共享帐户](/documentation/articles/active-directory-sharing-accounts/) |
| 自动定期更改特定应用的密码 |[自动密码滚动更新（预览版）](https://blogs.technet.microsoft.com/enterprisemobility/2015/02/20/azure-ad-automated-password-roll-over-for-facebook-twitter-and-linkedin-now-in-preview/) |
| Internet Explorer 版本的 Azure AD 密码管理扩展功能的部署和故障排除指南 |如何使用组策略部署 Internet Explorer 的访问面板扩展 <br /><br /> 排查 Internet Explorer 访问面板扩展问题 |

基于密码的单一登录适用于所有版本的 Azure AD，每个用户最多十个应用。[Azure AD 高级版](/pricing/details/identity/) 支持无限数目的应用程序。如果组织拥有 [Azure AD 基本版](/pricing/details/identity/)或 [Azure AD 高级版](/pricing/details/identity/)，则可以使用组来分配对应用程序的访问权限。自动密码滚动更新是一项 [Azure AD 高级版](/pricing/details/identity/)功能。


### 在 Azure AD 与本地 AD 之间启用单一登录
如果你的组织在本地维护 Windows Server Active Directory，并在云中维护 Azure Active Directory，则你可能想在这两个系统之间启用单一登录。Azure AD Connect（将这两个系统集成在一起的工具）提供多个设置单一登录的选项：建立与 ADFS 的联合或其他联合提供程序，或启用密码同步。

| 文章指南 | |
|:---:| --- |
| Azure AD Connect 中提供的单一登录选项概述，以及管理混合环境的相关信息 |[Azure AD Connect 中的用户登录选项](/documentation/articles/active-directory-aadconnect-user-signin/) |
| 同时使用本地 Active Directory 和 Azure Active Directory 管理环境的一般指导 | [将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect/) |
| 有关使用密码同步启用 SSO 的指导 |[使用 Azure AD Connect 实现密码同步](/documentation/articles/active-directory-aadconnectsync-implement-password-synchronization/)<br /><br />[密码同步疑难解答](https://support.microsoft.com/zh-cn/kb/2855271) |
| 有关使用密码写回启用 SSO 的指导 |[Azure AD 中的密码管理入门](/documentation/articles/active-directory-passwords-getting-started/)<br /><br />[密码写回疑难解答](/documentation/articles/active-directory-passwords-troubleshoot/#troubleshoot-password-writeback/) |
| 有关使用第三方标识提供程序启用 SSO 的指南 |[可用于启用单一登录的兼容第三方标识提供程序列表](https://aka.ms/ssoproviders) |

Azure AD Connect 适用于[所有版本的 Azure Active Directory](/pricing/details/identity/)。Azure AD 自助密码重置适用于 [Azure AD 基本版](/pricing/details/identity/)和 [Azure AD 高级版](/pricing/details/identity/)。对本地 AD 进行密码写回是一项 [Azure AD 高级版](/pricing/details/identity/)功能。

## 应用和 Azure AD

###构建与 Azure AD 集成的应用程序

如果你的组织正在开发或维护业务线 (LoB) 应用程序，或者如果你是 Azure Active Directory 客户的应用开发人员，以下教程可帮助你将应用程序与 Azure AD 集成。

| 文章指南 | |
|:---:| --- |
| 有关 IT 专业人员和应用程序开发人员集成应用程序与 Azure AD 的指南 |[适用于开发 Azure AD 应用程序的 IT 专业人员指南](/documentation/articles/active-directory-applications-guiding-developers-for-lob-applications/)<br /><br />[Azure Active Directory 的开发人员指南](/documentation/articles/active-directory-developers-guide/) |
| 应用程序供应商如何将其应用添加到 Azure AD 应用库 |[列出 Azure Active Directory 应用程序库中的应用程序](/documentation/articles/active-directory-app-gallery-listing/) |
| 如何使用 Azure Active Directory 管理对开发的应用程序的访问 |[如何对开发的应用程序启用用户分配](/documentation/articles/active-directory-applications-guiding-developers-requiring-user-assignment/)<br /><br />[将用户分配到应用](/documentation/articles/active-directory-applications-guiding-developers-assigning-users/)<br /> |


## 管理对应用程序的访问

###使用组和自助服务管理谁可以访问哪些应用

为了帮助管理谁有权访问哪些资源，Azure Active Directory 可让你使用组设置分配和权限级别。IT 可以选择启用自助功能，以便在需要时，用户可以直接请求权限。

| 文章指南 | |
| :---: | --- |
| Azure AD 访问管理功能的概述 | [有关管理对应用的访问的简介](/documentation/articles/active-directory-managing-access-to-apps/)<br /><br />|





###访问面板：用于访问应用和自助功能的门户

用户可以在 Azure AD 访问面板上启动应用程序和访问自助功能，而自助功能可让用户管理自己的应用程序和组成员资格。除了访问面板外，以下列表还包括其他用于访问启用 SSO 的应用程序的选项。

| 文章指南 | |
|:---:| --- |
| 用于将单一登录应用部署到用户的各种选项比较 |[为用户部署 Azure AD 集成的应用程序](/documentation/articles/active-directory-appssoaccess-whatis/#deploying-azure-ad-integrated-applications-to-users/) |
| 访问面板及其移动对应产品 MyApps 的概述 | 访问面板和 MyApps 简介 <br /> - [iOS](https://itunes.apple.com/us/app/my-apps-azure-active-directory/id824048653?mt=8)<br /> - [Android](https://play.google.com/store/apps/details?id=com.microsoft.myapps) |
| 如何从 Office 365 网站访问 Azure AD 应用 |[使用 Office 365 应用启动程序](https://support.office.com/zh-cn/article/Meet-the-Office-365-app-launcher-79f12104-6fed-442f-96a0-eb089a3f476a) |
| 如何从 Intune Managed Browser 移动应用访问 Azure AD 应用 |[Intune Managed Browser](https://technet.microsoft.com/zh-cn/library/dn878029.aspx)<br /> - [iOS](https://itunes.apple.com/us/app/microsoft-intune-managed-browser/id943264951?mt=8)<br /> - [Android](https://play.google.com/store/apps/details?id=com.microsoft.intune.mam.managedbrowser) |
| 如何使用深层链接访问 Azure AD 应用和启动单一登录 |[获取应用的直接登录链接](/documentation/articles/active-directory-appssoaccess-whatis/#direct-sign-on-links-for-federated-password-based-or-existing-apps/) |

访问面板适用于[所有版本的 Azure Active Directory](/pricing/details/identity/)。


##另请参阅

[什么是 Azure Active Directory？](/documentation/articles/active-directory-whatis/)

[Azure 多重身份验证](/home/features/multi-factor-authentication/)

<!---HONumber=Mooncake_0120_2017-->
<!---Update_Description: wording update -->