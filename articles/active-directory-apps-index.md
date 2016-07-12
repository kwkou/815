<properties
	pageTitle="有关 Azure Active Directory 中应用程序管理的文章索引 | Azure"
	description="了解如何自定义联合证书的过期日期，以及如何续订即将过期的证书。"
	services="active-directory"
	documentationCenter=""
	authors="MarkusVi"
	manager="stevenpo"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="05/03/2016"
	wacn.date="06/24/2016"/>
	
	

#有关 Azure Active Directory 中应用程序管理的文章索引

本页提供了一份完整列表，列出编写的 Azure Active Directory (Azure AD) 中各种应用程序相关功能的所有文章。

每个主要功能区都有简介，同时也根据你要查找的信息指导需要阅读的文章。

##概述文章

对于只需要 Azure AD 应用程序管理功能的简要说明的用户，以下文章是很好的起点。

| 文章指南 | |
| :---: | --- |
| 介绍 Azure AD 解决的应用程序管理问题 | [使用 Azure Active Directory (AD) 管理应用程序](/documentation/articles/active-directory-enable-sso-scenario/) |
| Azure AD 中与启用单一登录、定义有权访问应用程序的人员，以及用户如何启动应用程序相关的各种功能概述 | [Azure Active Directory 中的应用程序访问和单一登录](/documentation/articles/active-directory-appssoaccess-whatis/) |
| 探讨将应用程序集成到 Azure AD 时所涉及的不同步骤 | [将 Azure Active Directory 与应用程序集成](/documentation/articles/active-directory-integrating-applications-getting-started/)<br /><br />[启用 SaaS 应用的单一登录](/documentation/articles/active-directory-sso-integrate-saas-apps/)<br /><br />[管理对应用的访问](/documentation/articles/active-directory-managing-access-to-apps/) |
| 如何在 Azure AD 中表示应用程序的技术说明 | [如何以及为何将应用程序添加到 Azure AD](/documentation/articles/active-directory-how-applications-are-added/) |

##疑难解答文章

本部分提供相关疑难解答指南的快速访问链接。可以在本页的余下部分找到有关每个功能区的详细信息。

| 功能区 | |
| :---: | --- |
| 联合单一登录 | [排查基于 SAML 的单一登录问题](/documentation/articles/active-directory-saml-debugging/) |
| 基于密码的单一登录 | [Internet Explorer 访问面板扩展故障排除](/documentation/articles/active-directory-saas-ie-troubleshooting/) |
| 应用程序代理 | [应用代理故障排除指南](/documentation/articles/active-directory-application-proxy-troubleshoot/) |
| 本地 AD 与 Azure AD 之间的单一登录 | [密码同步疑难解答](/documentation/articles/active-directory-aadconnectsync-implement-password-synchronization/#managing-password-synchronization)<br /><br />[密码写回疑难解答](/documentation/articles/active-directory-passwords-troubleshoot/#troubleshoot-password-writeback) | 
| 动态组成员身份 | [动态组成员身份疑难解答](/documentation/articles/active-directory-accessmanagement-troubleshooting/) |

##单一登录 (SSO)

###联合单一登录：使用一个标识登录多个应用程序

单一登录可让用户只使用一组凭据，访问各种不同的应用程序和服务。可以通过联合方法启用单一登录。当用户尝试登录联合应用时，将被重定向到其组织的官方登录页面（由 Azure Active Directory 呈现），一旦身份验证成功，即重定向回到应用。

| 文章指南 | |
| :---: | --- |
| 联合身份验证和其他登录类型简介 | [使用 Azure AD 进行单一登录](/documentation/articles/active-directory-appssoaccess-whatis/) |
| 通过已简化的单一登录配置步骤与 Azure AD 预先集成的数千个 SaaS 应用程序 | [Azure AD 应用程序库入门](/documentation/articles/active-directory-appssoaccess-whatis/#get-started-with-the-azure-ad-application-gallery)<br /><br />[支持联合身份验证的预先集成应用完整列表](http://aka.ms/aadfederatedapps)<br /><br />[如何将应用添加到 Azure AD 应用库](/documentation/articles/active-directory-app-gallery-listing/) |
| 超过 150 个应用教程，讲解如何为 [Salesforce](/documentation/articles/active-directory-saas-salesforce-tutorial/)、[ServiceNow](/documentation/articles/active-directory-saas-servicenow-tutorial/)、[Google Apps](/documentation/articles/active-directory-saas-google-apps-tutorial/)、[Workday](/documentation/articles/active-directory-saas-workday-tutorial/) 等应用配置单一登录 | [有关如何将 SaaS 应用与 Azure Active Directory 集成的教程列表](/documentation/articles/active-directory-saas-tutorial-list/) |
| 如何手动设置和自定义单一登录配置 | [如何针对不在 Azure Active Directory 应用程序库中的应用配置联合单一登录](/documentation/articles/active-directory-saas-custom-apps/)<br /><br />[如何针对预先集成的应用自定义在 SAML 令牌中颁发的声明](/documentation/articles/active-directory-saml-claims-customization/) |
| 使用 SAML 协议的联合应用的故障排除指南 | [排查基于 SAML 的单一登录问题](/documentation/articles/active-directory-saml-debugging/) |
| 如何设置应用的证书过期日期，以及如何续订证书 | [在 Azure Active Directory 中管理用于联合单一登录的证书](/documentation/articles/active-directory-sso-certs/) |

联合单一登录适用于所有版本的 Azure AD，每个用户最多十个应用。[Azure AD 高级版](https://azure.microsoft.com/pricing/details/active-directory/)支持无限数目的应用程序。如果你的组织拥有 [Azure AD 基本版](https://azure.microsoft.com/pricing/details/active-directory/)或 [Azure AD 高级版](https://azure.microsoft.com/pricing/details/active-directory/)，则你可以[使用组来分配对联合应用程序的访问权限](#how-to-manage-who-has-access-to-which-apps)。

###基于密码的单一登录：非联合应用的帐户共享和 SSO

若要启用单一登录到应用程序不支持联合身份验证，Azure AD 提供的密码管理功能，可安全地存储密码的 SaaS 应用并自动将用户登录到这些应用。你可以轻松分发新建帐户的凭据，并与多人共享团队帐户。用户无需要知道他们有权访问的帐户凭据。

| 文章指南 | |
| :---: | --- |
| 基于密码的 SSO 工作原理简介以及简要的技术概述 | [使用 Azure AD 进行基于密码的单一登录](/documentation/articles/active-directory-appssoaccess-whatis/#password-based-single-sign-on) |
| 与共享帐户相关的方案以及 Azure AD 如何解决这些问题的摘要 | [使用 Azure AD 共享帐户](/documentation/articles/active-directory-sharing-accounts/) |
| 自动定期更改特定应用的密码 | [自动密码滚动更新（预览版）](http://blogs.technet.com/b/ad/archive/2015/02/20/azure-ad-automated-password-roll-over-for-facebook-twitter-and-linkedin-now-in-preview.aspx0) |
| Internet Explorer 版本的 Azure AD 密码管理扩展功能的部署和故障排除指南 | [如何使用组策略部署 Internet Explorer 的访问面板扩展](/documentation/articles/active-directory-saas-ie-group-policy/)<br /><br />[排查 Internet Explorer 访问面板扩展问题](/documentation/articles/active-directory-saas-ie-troubleshooting/) |

基于密码的单一登录适用于所有版本的 Azure AD，每个用户最多十个应用。[Azure AD Premium](/pricing/details/active-directory/) 支持无限数目的应用程序。如果你的组织拥有 [Azure AD 基本版](https://azure.microsoft.com/pricing/details/active-directory/)或 [Azure AD 高级版](https://azure.microsoft.com/pricing/details/active-directory/)，则你可以[使用组来分配对应用程序的访问权限](#how-to-manage-who-has-access-to-which-apps)。自动密码滚动更新是一项 [Azure AD 高级版](https://azure.microsoft.com/pricing/details/active-directory/)功能。

###应用程序代理：单一登录和远程访问本地应用程序

如果专用网络中具有需要由网络外部的用户和设备访问的应用程序，可以使用 Azure AD 应用程序代理对这些应用程序启用安全远程访问。

| 文章指南 | |
| :---: | --- |
| Azure AD 应用程序代理及其工作原理的概述 | [提供安全的本地应用程序远程访问](/documentation/articles/active-directory-application-proxy-get-started/)<br /><br />[Azure AD 应用代理的工作原理](/documentation/articles/active-directory-appssoaccess-enable-hybrid-access/) |
| 有关如何配置应用程序代理以及如何发布第一个应用的教程 | [如何设置 Azure AD 应用代理](/documentation/articles/active-directory-application-proxy-enable/)<br /><br />[如何以无提示方式安装应用代理连接器](/documentation/articles/active-directory-application-proxy-silent-installation/)<br /><br />[如何使用应用代理发布应用程序](/documentation/articles/active-directory-application-proxy-publish/)<br /><br />[如何使用自己的域名](/documentation/articles/active-directory-application-proxy-custom-domains/) |
| 如何对使用应用代理发布的应用程序启用单一登录和条件性访问 | [使用应用程序代理进行单一登录](/documentation/articles/active-directory-application-proxy-sso-using-kcd/)<br /><br />[条件性访问和应用程序代理](/documentation/articles/active-directory-application-proxy-conditional-access/) |
| 有关如何对以下方案使用应用程序代理的指导 | [如何支持本机客户端应用程序](/documentation/articles/active-directory-application-proxy-native-client/)<br /><br />[如何支持声明感知应用程序](/documentation/articles/active-directory-application-proxy-claims-aware-apps/)<br /><br />[如何支持在不同网络和位置上发布的应用程序](/documentation/articles/active-directory-application-proxy-connectors/) |
| 应用程序代理故障排除指南 | [应用代理故障排除指南](/documentation/articles/active-directory-application-proxy-troubleshoot/) |

应用程序代理适用于所有版本的 Azure AD，每个用户最多十个应用。[Azure AD 高级版](https://azure.microsoft.com/pricing/details/active-directory/)支持无限数目的应用程序。如果你的组织拥有 [Azure AD 基本版](https://azure.microsoft.com/pricing/details/active-directory/)或 [Azure AD 高级版](https://azure.microsoft.com/pricing/details/active-directory/)，则你可以[使用组来分配对应用程序的访问权限](#how-to-manage-who-has-access-to-which-apps)。

你也可能对 [Azure AD 域服务](../active-directory-domain-services/active-directory-ds-overview)感兴趣，这些服务可让你将本地应用程序迁移到 Azure，同时又满足这些应用程序的标识需求。

###在 Azure AD 与本地 AD 之间启用单一登录

如果你的组织在本地维护 Windows Server Active Directory，并在云中维护 Azure Active Directory，则你可能想在这两个系统之间启用单一登录。Azure AD Connect（将这两个系统集成在一起的工具）提供多个设置单一登录的选项：建立与 ADFS 的联合或其他联合提供程序，或启用密码同步。

| 文章指南 | |
| :---: | --- |
| Azure AD Connect 中提供的单一登录选项概述，以及管理混合环境的相关信息 | [Azure AD Connect 中的用户登录选项](/documentation/articles/active-directory-aadconnect-user-signin/) |
| 同时使用本地 Active Directory 和 Azure Active Directory 管理环境的一般指导 | [Azure AD 混合标识设计注意事项](/documentation/articles/active-directory-hybrid-identity-design-considerations-overview/)<br /><br />[将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect/) |
| 有关使用密码同步启用 SSO 的指导 | [使用 Azure AD Connect 实现密码同步](/documentation/articles/active-directory-aadconnectsync-implement-password-synchronization/)<br /><br />[密码同步疑难解答](https://support.microsoft.com/zh-cn/kb/2855271) |
| 有关使用密码写回启用 SSO 的指导 | [Azure AD 中的密码管理入门](/documentation/articles/active-directory-passwords-getting-started/)<br /><br />[密码写回疑难解答](/documentation/articles/active-directory-passwords-troubleshoot/#troubleshoot-password-writeback) |
| 有关使用第三方标识提供者启用 SSO 的指导 | [可用于启用单一登录的兼容第三方标识提供者列表](https://aka.ms/ssoproviders) | 
| Windows 10 用户如何通过 Azure AD Join 享受单一登录的优点 | [通过 Azure Active Directory Join 将云功能扩展到 Windows 10 设备](/documentation/articles/active-directory-azureadjoin-overview/) |

Azure AD Connect 适用于[所有版本的 Azure Active Directory](https://azure.microsoft.com/pricing/details/active-directory/)。Azure AD 自助密码重置适用于 [Azure AD 基本版](https://azure.microsoft.com/pricing/details/active-directory/)和 [Azure AD 高级版](https://azure.microsoft.com/pricing/details/active-directory/)。对本地 AD 进行密码写回是一项 [Azure AD 高级版](https://azure.microsoft.com/pricing/details/active-directory/)功能。

###条件性访问：强制高风险应用符合额外的安全要求

对应用程序和资源设置单一登录后，可以在每次登录该应用程序时，强制符合特定的安全要求，进一步保护机密应用的安全。例如，你可以使用 Azure AD 来要求每次访问特定的应用都需要执行多重身份验证，而不管该应用原本是否支持该功能。条件性访问的另一个常见示例是请求用户连接到组织信任的网络，以便能够访问特别机密的应用程序。

| 文章指南 | |
| :---: | --- |
| 跨 Azure AD、Office365 和 Intune 提供的条件性访问功能简介 | [使用条件性访问管理风险](/documentation/articles/active-directory-conditional-access/) |
| 如何为以下类型的资源启用条件性访问 | [SaaS 应用的条件性访问](/documentation/articles/active-directory-conditional-access-azuread-connected-apps/)<br /><br />[Office 365 服务的条件性访问](/documentation/articles/active-directory-conditional-access-device-policies/)<br /><br />[本地应用程序的条件性访问](/documentation/articles/active-directory-conditional-access-on-premises-setup/)<br /><br />[通过 Azure AD 应用代理发布的本地应用程序的条件性访问](/documentation/articles/active-directory-application-proxy-conditional-access/) |
| 如何向 Azure Active Directory 注册设备以启用基于设备的条件性访问策略 | [Azure Active Directory 设备注册概述](/documentation/articles/active-directory-conditional-access-device-registration-overview/)<br /><br />[如何为加入域的 Windows 设备启用自动设备注册](/documentation/articles/active-directory-conditional-access-automatic-device-registration/)<br /> — [适用于 Windows 8.1 设备的步骤](/documentation/articles/active-directory-conditional-access-automatic-device-registration-windows8_1/)<br /> — [适用于 Windows 7 设备的步骤](/documentation/articles/active-directory-conditional-access-automatic-device-registration-windows7/) |
| 如何对涉及多重身份验证的策略使用 Android 版 Azure 验证器应用 | [适用于 Android 的 Azure 验证器](/documentation/articles/active-directory-conditional-access-azure-authenticator-app/) |

条件性访问是一项 [Azure AD 高级版](https://azure.microsoft.com/pricing/details/active-directory/)功能。


##应用和 Azure AD

###Cloud App Discovery：查找组织正在使用哪些 SaaS 应用

Cloud App Discovery 可帮助 IT 部门了解整个组织正在使用哪些 SaaS 应用。它可以测量应用程序使用情况，让 IT 可以判断哪些购买的应用程序可在 IT 控制下获得最大益处，且可与 Azure AD 集成。

| 文章指南 | |
| :---: | --- |
| 工作原理的一般概述 | [使用 Cloud App Discovery 查找未经认可的云应用程序](/documentation/articles/active-directory-cloudappdiscovery-whatis/) |
| 深入探讨其工作原理，并回答有关隐私的问题 | [安全和隐私注意事项](/documentation/articles/active-directory-cloudappdiscovery-security-and-privacy-considerations/) |
| 常见问题 | [Cloud App Discovery 常见问题](http://social.technet.microsoft.com/wiki/contents/articles/24037.cloud-app-discovery-frequently-asked-questions.aspx) |
| 有关部署 Cloud App Discovery 的教程 | [组策略部署指南](http://social.technet.microsoft.com/wiki/contents/articles/30965.cloud-app-discovery-group-policy-deployment-guide.aspx)<br /><br />[System Center 部署指南](http://social.technet.microsoft.com/wiki/contents/articles/30968.cloud-app-discovery-system-center-deployment-guide.aspx)<br /><br />[在具有自定义端口的代理服务器上安装](/documentation/articles/active-directory-cloudappdiscovery-registry-settings-for-proxy-services/) |
| Cloud App Discovery 代理更新的更改日志 | [更改日志](http://social.technet.microsoft.com/wiki/contents/articles/24616.cloud-app-discovery-agent-changelog.aspx) |

Cloud App Discovery 是一项 [Azure AD Premium](/pricing/details/active-directory/) 功能。

###自动预配和取消预配 SaaS 应用中的用户帐户

自动执行创建、维护和删除 SaaS 应用程序，如 Dropbox、Salesforce、ServiceNow 等中的用户标识。在 Azure AD 与 SaaS 应用程序之间匹配和同步现有标识，并在用户离开组织时自动禁用帐户以控制访问。

| 文章指南 | |
| :---: | --- |
| 了解其工作原理，并查找常见问题的解答 | [在 SaaS 应用中自动预配和取消预配用户](/documentation/articles/active-directory-saas-app-provisioning/) |
| 配置如何在 Azure AD 与 SaaS 应用程序之间映射信息 | [自定义属性映射](/documentation/articles/active-directory-saas-customizing-attribute-mappings/)<br><br>[编写属性映射表达式](/documentation/articles/active-directory-saas-writing-expressions-for-attribute-mappings/) |
| 如何对支持 SCIM 协议的任何应用启用自动化预配 | [对启用 SCIM 的任何应用设置自动化用户预配](/documentation/articles/active-directory-scim-provisioning/) |
| 获取有关预配失败的通知 | [预配通知](/documentation/articles/active-directory-saas-account-provisioning-notifications/) |
| 根据应用程序的属性值限制可以预配应用程序的人员 | [范围筛选器](/documentation/articles/active-directory-saas-scoping-filters/) |

自动化用户预配适用于所有版本的 Azure AD，每个用户最多十个应用程序。[Azure AD 高级版](https://azure.microsoft.com/pricing/details/active-directory/)支持无限数目的应用程序。如果你的组织拥有 [Azure AD Basic](/pricing/details/active-directory/) 或 [Azure AD Premium](https://azure.microsoft.com/pricing/details/active-directory/)，则你可以[使用组来管理要预配哪些用户](#how-to-manage-who-has-access-to-which-apps)。

###构建与 Azure AD 集成的应用程序

如果你的组织正在开发或维护业务线 (LoB) 应用程序，或者如果你是 Azure Active Directory 客户的应用开发人员，以下教程可帮助你将应用程序与 Azure AD 集成。

| 文章指南 | |
| :---: | --- |
| 有关 IT 专业人员和应用程序开发人员集成应用程序与 Azure AD 的指南 | [适用于开发 Azure AD 应用程序的 IT 专业人员指南](/documentation/articles/active-directory-applications-guiding-developers-for-lob-applications/)<br /><br />[Azure Active Directory 的开发人员指南](/documentation/articles/active-directory-developers-guide/) |
| 应用程序供应商如何将其应用添加到 Azure AD 应用库 | [列出 Azure Active Directory 应用程序库中的应用程序](/documentation/articles/active-directory-app-gallery-listing/) |
| 如何使用 Azure Active Directory 管理对开发的应用程序的访问 | [如何对开发的应用程序启用用户分配](/documentation/articles/active-directory-applications-guiding-developers-requiring-user-assignment/)<br /><br />[将用户分配到应用](/documentation/articles/active-directory-applications-guiding-developers-assigning-users/)<br /><br />[将组分配到应用](/documentation/articles/active-directory-applications-guiding-developers-assigning-groups/) |

如果你正在开发面向消费者的应用程序，可能对使用 [Azure Active Directory B2C](https://azure.microsoft.com/services/active-directory-b2c/) 感兴趣，这可以让你不必开发自己的标识系统即可管理用户。[了解详细信息](/documentation/articles/active-directory-b2c-overview/)。


##管理对应用程序的访问

###使用组和自助服务管理谁可以访问哪些应用

为了帮助管理谁有权访问哪些资源，Azure Active Directory 可让你使用组设置分配和权限级别。IT 可以选择启用自助功能，以便在需要时，用户可以直接请求权限。

| 文章指南 | |
| :---: | --- |
| Azure AD 访问管理功能的概述 | [有关管理对应用的访问的简介](/documentation/articles/active-directory-managing-access-to-apps/)<br /><br />[Azure AD 中访问管理的工作原理](/documentation/articles/active-directory-manage-groups/)<br /><br />[如何使用组来管理对 SaaS 应用程序的访问](/documentation/articles/active-directory-accessmanagement-group-saasapps/) |
| 启用应用程序和组的自助管理 | [自助应用程序管理](/documentation/articles/active-directory-self-service-application-access/)<br /><br />[自助组管理](/documentation/articles/active-directory-accessmanagement-self-service-group-management/) |
| 有关在 Azure AD 中设置组的说明 | [如何创建安全组](/documentation/articles/active-directory-accessmanagement-manage-groups/)<br /><br />[如何指定组的所有者](/documentation/articles/active-directory-accessmanagement-managing-group-owners/)<br /><br />[如何使用“所有用户”组](/documentation/articles/active-directory-accessmanagement-dedicated-groups/) |
| 使用动态组通过基于属性的成员资格规则自动填充组成员资格 | [动态组成员资格：简单规则](/documentation/articles/active-directory-accessmanagement-simplerulegroup/)<br /><br />[动态组成员资格：高级规则](/documentation/articles/active-directory-accessmanagement-groups-with-advanced-rules/)<br /><br />[动态组成员资格疑难解答](/documentation/articles/active-directory-accessmanagement-troubleshooting/) |

基于组的应用程序访问管理适用于 [Azure AD Basic](/pricing/details/active-directory/) 和 [Azure AD Premium](/pricing/details/active-directory/)。自助组管理、自助应用程序管理以及动态组是 [Azure AD Premium](/pricing/details/active-directory/) 功能。

###B2B 协作：启用应用程序的合作访问

如果你的公司与其他公司合作，可能需要管理公司应用程序的合作访问。Azure Active Directory B2B 协作提供简单且安全的方式来与合作伙伴共享应用程序。此功能目前处于预览状态。

| 文章指南 | |
| :---: | --- |
| 不同 Azure AD 功能的概述，帮助管理外部用户，例如合作伙伴、客户等。 | [比较 Azure AD 中用于管理外部标识的功能](/documentation/articles/active-directory-b2b-compare-external-identities/) |
| B2B 协作预览版简介及其入门方法 | [以简单安全的方式将云合作伙伴与 Azure AD 集成](/documentation/articles/active-directory-b2b-what-is-azure-ad-b2b/)<br /><br />[Azure Active Directory B2B 协作](/documentation/articles/active-directory-b2b-collaboration-overview/) |
| 深入探讨 Azure AD B2B 协作及其用法 | [B2B 协作：工作原理](/documentation/articles/active-directory-b2b-how-it-works/)<br /><br />[Azure AD B2B 协作预览版的当前限制](/documentation/articles/active-directory-b2b-current-preview-limitations/)<br /><br />[Azure AD B2B 协作预览版的详细用法演练](/documentation/articles/active-directory-b2b-detailed-walkthrough/) |
| 有关 Azure AD B2B 协作工作原理的技术详细信息参考文章 | [用于添加合作伙伴用户的 CSV 文件格式](/documentation/articles/active-directory-b2b-references-csv-file-format/)<br /><br />[受 Azure AD B2B 协作影响的用户属性](/documentation/articles/active-directory-b2b-references-external-user-object-attribute-changes/)<br /><br />[合作伙伴用户的用户令牌格式](/documentation/articles/active-directory-b2b-references-external-user-token-format/) |

B2B 协作预览版目前适用于[所有版本的 Azure Active Directory](https://azure.microsoft.com/pricing/details/active-directory/)。

###访问面板：用于访问应用和自助功能的门户

用户可以在 Azure AD 访问面板上启动应用程序和访问自助功能，而自助功能可让用户管理自己的应用程序和组成员资格。除了访问面板外，以下列表还包括其他用于访问启用 SSO 的应用程序的选项。

| 文章指南 | |
| :---: | --- |
| 用于将单一登录应用部署到用户的各种选项比较 | [为用户部署 Azure AD 集成的应用程序](/documentation/articles/active-directory-appssoaccess-whatis/#deploying-azure-ad-integrated-applications-to-users) |
| 访问面板及其移动对应产品 MyApps 的概述 | [访问面板和 MyApps 简介](/documentation/articles/active-directory-saas-access-panel-introduction/)<br /> — [iOS](https://itunes.apple.com/us/app/my-apps-azure-active-directory/id824048653?mt=8)<br /> — [Android](https://play.google.com/store/apps/details?id=com.microsoft.myapps) |
| 如何从 Office 365 网站访问 Azure AD 应用 | [使用 Office 365 应用启动程序](https://support.office.com/zh-cn/article/Meet-the-Office-365-app-launcher-79f12104-6fed-442f-96a0-eb089a3f476a) |
| 如何从 Intune Managed Browser 移动应用访问 Azure AD 应用 | [Intune Managed Browser](https://technet.microsoft.com/zh-cn/library/dn878029.aspx)<br /> - [iOS](https://itunes.apple.com/us/app/microsoft-intune-managed-browser/id943264951?mt=8)<br /> - [Android](https://play.google.com/store/apps/details?id=com.microsoft.intune.mam.managedbrowser) |
| 如何使用深层链接访问 Azure AD 应用和启动单一登录 | [获取应用的直接登录链接](/documentation/articles/active-directory-appssoaccess-whatis/#direct-sign-on-links-for-federated-password-based-or-existing-apps) |

访问面板适用于[所有版本的 Azure Active Directory](/pricing/details/active-directory/)。

###报告：轻松审核应用访问更改及监视应用登录

Azure Active Directory 提供多个报告和警报，以帮助你监视组织对应用程序的访问。你可以在应用发生异常登录时收到警报，还可以跟踪用户访问应用程序的时间及更改原因。

| 文章指南 | |
| :---: | --- |
| Azure Active Directory 中报告功能的概述 | [Azure AD 报告入门](/documentation/articles/active-directory-reporting-getting-started/) |
| 如何监视用户的登录和应用使用情况 | [查看访问和使用情况报告](/documentation/articles/active-directory-view-access-usage-reports/)<br /><br />[Azure Active Directory 所有用户登录活动报告](/documentation/articles/active-directory-reporting-all-user-sign-in-activity-report/) |
| 跟踪对可以访问特定应用程序所做更改的人员 | [Azure Active Directory 审核报告事件](/documentation/articles/active-directory-reporting-audit-events/) |
| 使用报告 API 将这些报告的数据导出到首选的工具 | [Azure AD Reporting API 入门](/documentation/articles/active-directory-reporting-api-getting-started/) |

若要查看不同版本的 Azure Active Directory 随附哪些报告，请[单击此处](/documentation/articles/active-directory-view-access-usage-reports/#report-editions)。

##另请参阅

[什么是 Azure Active Directory？](/documentation/articles/active-directory-whatis/)

[Azure Active Directory B2C](/services/active-directory-b2c/)

[Azure Active Directory 域服务](/services/active-directory-ds/)

[Azure Multi-Factor Authentication](/services/multi-factor-authentication/)

<!---HONumber=Mooncake_0613_2016-->