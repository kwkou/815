<properties
   pageTitle="Azure 标识管理和访问控制安全最佳实践 | Azure"
   description="本文提供一系列有关使用内置 Azure 功能进行标识管理和访问控制的最佳实践。"
   services="virtual-machines, cloud-services, storage"
   documentationCenter="na"
   authors="YuriDio"
   manager="swadhwa"
   editor="TomSh"/>

<tags
   ms.service="azure-security"
   ms.date="05/06/2016"
   wacn.date="05/30/2016"/>

# Azure 标识管理和访问控制安全最佳实践

许多人将标识视为安全性的新边界层，从传统的以网络为中心的角度来看可以接管这种角色。安全关注与投资主轴的这种演变来源于这样一个事实：网络周边的漏洞越来越多，在 [BYOD](http://aka.ms/byodcg) 设备和云应用程序呈爆炸性增长之前，周边防御不如以前那么有效。

本文介绍一系列 Azure 标识管理和访问控制安全最佳实践。这些最佳实践衍生自我们的 [Azure AD](/documentation/articles/active-directory-whatis) 经验和客户经验。

对于每项最佳实践，我们将说明：

- 最佳实践是什么
- 为何要启用该最佳实践
- 如果你无法启用该最佳实践，可能的结果是什么
- 最佳实践的可能替代方案
- 如何学习启用最佳实践

这篇 Azure 标识管理和访问控制安全最佳实践以共识以及 Azure 平台功能和特性集（因为在编写本文时已存在）为基础。看法和技术将随着时间改变，本文将会定期更新以反映这些更改。

本文中介绍的 Azure 标识管理和访问控制安全最佳实践包括：

- 集中化标识管理
- 启用单一登录 (SSO)
- 部署密码管理
- 对用户实施多重身份验证 (MFA)
- 使用基于角色的访问控制 (RBAC) 
- 使用 Resource Manager 控制创建资源的位置
- 指导开发人员利用适用于 SaaS 应用的标识功能
- 主动监视可疑活动

## 集中化标识管理

保护标识安全性的一项重要措施是确保 IT 部门可以从创建帐户的单个位置管理帐户。大多数企业 IT 组织都将主要帐户目录部署在本地，而混合云部署正呈上升趋势，因此，必须了解如何集成本地目录和云目录，并向最终用户提供无缝的体验。

若要实现这种[混合标识](/documentation/articles/active-directory-hybrid-identity-design-considerations-overview)方案，我们建议采用两种做法：

- 使用 Azure AD Connect 将本地目录与云目录同步
- 使用 [Active Directory 联合身份验证服务](https://msdn.microsoft.com/library/bb897402.aspx) (AD FS) 将本地标识与云目录联合

无法将本地标识与云标识集成的组织在管理帐户时会遇到管理开销增大的情况，从而增加出错和安全违规的可能性。

有关 Azure AD 同步的详细信息，请阅读 [Integrating your on-premises identities with Azure Active Directory（将本地标识与 Azure Active Directory 集成）](/documentation/articles/active-directory-aadconnect)一文。

## 启用单一登录 (SSO)

如果你要管理多个目录，则不仅会给 IT 人员造成管理问题，而且最终用户还必须记住多个密码。通过使用 [SSO](https://azure.microsoft.com/documentation/videos/overview-of-single-sign-on/)，可以让用户使用相同的凭据集登录并访问他们所需的资源，而不管此资源位于本地还是云中。

用户可以使用 SSO 基于他们在 Azure AD 中的组织帐户访问 [SaaS 应用程序](/documentation/articles/active-directory-appssoaccess-whatis)。这是不仅适用于 Microsoft SaaS 应用，而且适用于其他应用，例如 [Google Apps](/documentation/articles/active-directory-saas-google-apps-tutorial) 和 [Salesforce](/documentation/articles/active-directory-saas-salesforce-tutorial)。可将应用程序配置为使用 Azure AD 作为[基于 SAML 的标识](/documentation/articles/fundamentals-identity)提供者。作为安全控制机制，Azure AD 不会发出可让用户登录应用程序的令牌，除非使用 Azure AD 向他们授予了访问权限。你可以直接或者通过他们所属的组授予访问权限。

> [AZURE.NOTE] 是否使用 SSO 的决定会影响到如何将本地目录与云目录集成。如果你要使用 SSO，则需要使用联合，因为目录同步只提供[相同的登录体验](/documentation/articles/active-directory-aadconnect)。

不对其用户和应用程序实施 SSO 的组织更容易遇到这种情况：用户需要设置多个密码，这直接增大了重复使用密码或使用弱密码的可能性。

有关 Azure AD SSO 的详细信息，请阅读 [AD FS management and customizaton with Azure AD Connect（使用 Azure AD Connect 进行 AD FS 管理和自定义）](/documentation/articles/active-directory-aadconnect-federation-management)一文。

## 部署密码管理

如果你有多个租户或者想要允许用户[重置其自己的密码](/documentation/articles/active-directory-passwords-update-your-own-password)，则必须使用适当的安全策略来防止滥用。在 Azure 中，你可以利用自助密码重置功能并自定义安全选项来满足你的业务要求。

必须从这些用户获取反馈，并在用户尝试执行这些步骤时，从用户的体验中获得经验教训，这一点尤其重要。根据这些经验制定计划，以减少在部署大型组时可能出现的潜在问题。此外，建议你使用[密码重置注册活动报告](/documentation/articles/active-directory-passwords-get-insights)来监视正在注册的用户。

对于想要避免密码更改支持电话但又允许用户重置其自身密码的组织，服务台更容易收到大量由于密码问题而打来的电话。在具有多个租户的组织中，必须实施这种类型的功能，并允许用户在安全策略中规定的安全边界范围内执行密码重置。

有关密码重置的详细信息，请阅读 [Deploying Password Management and training users to use it（部署密码管理并向用户培训其用法）](/documentation/articles/active-directory-passwords-best-practices)一文。

## 对用户实施多重身份验证 (MFA)

对于需要遵守行业标准（如 [PCI DSS 版本 3.2](http://blog.pcisecuritystandards.org/preparing-for-pci-dss-32)）的组织，必须使用多重身份验证功能来对用户进行身份验证。除了能够遵守行业标准以外，实施 MFA 来对用户进行身份验证还可帮助组织减少凭据失窃类型的攻击，如[传递哈希 (PtH)](http://aka.ms/PtHPaper)。

如果针对用户启用 Azure MFA，则可为用户登录和事务增加第二层安全性。在此情况下，事务可能将访问位于文件服务器或 SharePoint Online 中的文档。Azure MFA 还可帮助 IT 部门减少使用透露的凭据访问企业数据的可能性。

例如：针对用户实施 Azure MFA，并将其配置为使用电话呼叫或短信作为验证方法。如果用户的凭据被透露，攻击者无法访问任何资源，因为攻击者无权访问用户的电话。未添加额外标识保护层的组织将更容易受到凭据窃取攻击，从而导致数据泄漏。

想要保留完整本地身份验证控制权的组织有一个替代方法，就是使用 [Azure Multi-Factor Authentication 服务器](/documentation/articles/multi-factor-authentication-get-started-server)（也称为本地 MFA）。使用此方法仍可实施多重身份身份验证，同时保留本地 MFA 服务器。

有关 Azure MFA 的详细信息，请参阅 [Getting started with Azure Multi-Factor Authentication in the cloud（云中的 Azure Multi-Factor Authentication 入门）](./multi-factor-authentication/multi-factor-authentication-get-started-cloud)。


## 使用 Resource Manager 控制创建资源的位置

非常重要的一点是，既要允许云操作员执行任务，同时又要防止他们违反管理组织资源所需的惯例。想要控制创建资源的位置的组织应该对这些位置进行硬编码。
 
若要实现此目的，组织可以创建安全策略，其中的定义描述了会明确遭到拒绝的操作或资源。你可以在所需范围（例如订阅、资源组或是单个资源）分配这些策略定义。

> [AZURE.NOTE] 这与 RBAC 不同，实际上它利用 RBAC 来对有权创建这些资源的用户进行身份验证。

对于组织只想在有关联的适当成本中心时允许操作，否则将拒绝请求的情况，可以利用 [Azure Resource Manager](/documentation/articles/resource-group-overview) 来创建自定义策略。
 
无法控制资源创建方式的组织更容易遇到用户创建的资源超过所需数目，因而滥用服务的情况。强化资源创建过程是保护多租户方案的重要步骤。

有关使用 Azure Resource Manager 创建策略的详细信息，请参阅 [Use Policy to manage resources and control access（使用策略来管理资源和控制访问）](/documentation/articles/resource-manager-policy)一文。

## 指导开发人员利用适用于 SaaS 应用的标识功能

在许多情况下，当用户访问可集成到本地或云目录的 [SaaS 应用](https://azure.microsoft.com/marketplace/active-directory/all/)时，将会利用用户标识。首先且最重要的是，我们建议开发人员使用安全方法（如 [Microsoft 安全开发生命周期 (SDL)](https://www.microsoft.com/sdl/default.aspx)）来开发这些应用。Azure AD 通过以下方式简化了对开发人员的身份验证：将标识提供为一项服务、支持行业标准协议（例如 [OAuth 2.0](http://oauth.net/2/) 和 [OpenID Connect](http://openid.net/connect/)），并提供用于不同平台的开源库。

请务必将外包身份验证的任何应用程序注册到 Azure AD，这是必须执行的过程。幕后的原因是，Azure AD 在处理登录 (SSO) 或交换令牌时，需要协调与应用程序之间的通信。当 Azure AD 颁发的令牌的生存期过期时，用户的会话便过期。始终评估应用程序是否应使用此时限，或者你是否可以减少此时限。缩短生存期可以充当一种安全措施，根据非活动时间段强制用户注销。

不对应用访问实施标识控制且并不指导其开发人员如何安全地将应用与标识管理系统集成的组织，可能更容易遇到凭据失窃类型的攻击，例如[“开放 Web 应用程序安全项目 (OWASP) 10 大欺骗”中所述的弱身份验证和会话管理](https://www.owasp.org/index.php/OWASP_Top_Ten_Cheat_Sheet)。

有关适用于 SaaS 应用的身份验证方案的详细信息，请阅读 [Authentication Scenarios for Azure AD（Azure AD 的身份验证方案）](/documentation/articles/active-directory-authentication-scenarios)。

## 主动监视可疑活动

根据 [Verizon 2016 年数据违规报告](http://www.verizonenterprise.com/verizon-insights-lab/dbir/2016/)，凭据透露仍呈上升趋势，即将成为网络罪犯的最有利可图业务之一。出于此原因，必须部署主动的标识监视系统，以便快速检测可疑行为活动，并触发警报以作进一步调查。Azure AD 中的两项主要功能可帮助组织监视其标识：Azure AD 高级版中的[异常报告](./active-directory/active-directory-view-access-usage-reports)和 Azure AD 中的[标识保护](./active-directory/active-directory-identityprotection)功能。

请务必使用异常报告来识别以下企图：[在未受跟踪的情况下](./active-directory/active-directory-reporting-sign-ins-from-unknown-sources)登录、针对特定帐户展开[暴力破解](./active-directory/active-directory-reporting-sign-ins-after-multiple-failures)攻击、尝试从多个位置登录、从[受感染的设备](./active-directory/active-directory-reporting-sign-ins-from-possibly-infected-devices)和可疑 IP 地址登录。请记住这只是一些报告。换而言之，必须制定相应的流程和过程，使 IT 管理员每天或按需（通常在事件响应方案中）运行这些报告。

相比之下，Azure AD Identity Protection 是一种主动监视系统，它会在自身的仪表板上标记当前风险。除此之外，你还会通过电子邮件收到每日摘要通知。我们建议你根据业务要求调整风险级别。某个风险事件的风险级别指示该风险事件的严重性（高、中或低）。风险级别可帮助标识保护用户确定他们为了减少组织遇到的风险，而必须采取的措施的优先级。

不主动监视其标识系统的组织将面临用户凭据透露的风险。如果不知道有人正在使用这些凭据实施可疑活动，组织就无法缓解这种类型的威胁。
有关 Azure 标识保护的详细信息，请参阅 [Azure Active Directory Identity Protection（Azure Active Directory 标识保护）](/documentation/articles/active-directory-identityprotection)。

<!---HONumber=Mooncake_0523_2016-->