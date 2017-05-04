<properties
    pageTitle="Azure Active Directory PoC 演练手册的实现 | Azure"
    description="研究并快速实现标识和访问管理方案"
    services="active-directory"
    keywords="azure active directory, 演练手册, 概念证明, PoC"
    documentationcenter=""
    author="dstefanMSFT"
    manager="asuthar"
    translationtype="Human Translation" />
<tags
    ms.assetid=""
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="4/12/2017"
    wacn.date="05/02/2017"
    ms.author="dstefan"
    ms.sourcegitcommit="78da854d58905bc82228bcbff1de0fcfbc12d5ac"
    ms.openlocfilehash="d142b0cd89f7ea328aa5028b6a1b1b207f19fa4d"
    ms.lasthandoff="04/22/2017" />

# <a name="azure-active-directory-proof-of-concept-playbook-implementation"></a>Azure Active Directory 概念证明演练手册：实现

## <a name="foundation---syncing-ad-to-azure-ad"></a>基础 - 将 AD 同步到 Azure AD 

混合标识是已部署本地目录的大部分企业客户的基础。 混合标识的目标是有意地尽量花费最少的时间来展示实际标识和访问方案的价值。 

| 方案 | 构建基块| 
| --- | --- |  
| [将本地标识扩展到云](#extending-your-on-premises-identity-to-the-cloud) | [目录同步 - 密码哈希同步](/documentation/articles/active-directory-playbook-building-blocks/#directory-synchronization---password-hash-sync-phs---new-installation/) <br/>**注意**：如果已安装 DirSync/ADSync 或旧版的 Azure AD Connect，则此步骤是可选的。 本指南中的某些方案可能需要更新版本的 Azure AD Connect。

### <a name="extending-your-on-premises-identity-to-the-cloud"></a>将本地标识扩展到云 

1. Bob 是 Contoso 公司的 Active Directory 管理员。 公司要求他为一组用户启用“标识即服务”。 执行 Azure AD Connect 向导后，云中已提供目标用户的标识。 
2. Bob 请求 Susie（一个目标用户）访问 Azure Active Directory 访问面板并确认她是否可以进行身份验证。 Susie 看到了带品牌的登录页以及一个空白的访问面板，以后随后可以使用该面板实现应用程序访问。

## <a name="theme---lots-of-apps-one-identity"></a>主题 - 多个应用，一个标识

| 方案 | 构建基块| 
| --- | --- |  
| [SSO 和标识生命周期事件](#sso-and-identity-lifecycle-events) |  |
| [将 LDAP 标识同步到 Azure AD](#synchronize-ldap-identities-to-azure-ad) |  [通用 LDAP 连接器配置](/documentation/articles/active-directory-playbook-building-blocks/#generic-ldap-connector-configuration/) |

### <a name="sso-and-identity-lifecycle-events"></a>SSO 和标识生命周期事件

1. Susie 请了假，根据公司策略，她的本地 AD 帐户将被暂时禁用。 Susie 现在无法登录到 Azure AD，因此无法访问 ServiceNow。 
2. Susie 从营销部平级调动了到销售部。 Kevin 从 ServiceNow 中删除了其访问权限。 Susie 登录到 Azure AD myapps，但再也看不到 ServiceNow 磁贴。 10 分钟后，Kevin 确认已通过 ServiceNow 管理控制台禁用 Susie 的帐户。


### <a name="synchronize-ldap-identities-to-azure-ad"></a>将 LDAP 标识同步到 Azure AD

1. Bob 所在的公司采用复杂的标识基础结构。 大多数用户在 Windows Server Active Directory 域服务 (ADDS) 内部进行维护。 其中一些用户由 Active Directory 轻型目录服务 (ADLDS) 中的 HR 系统管理。
2. Bob 的任务是为所有用户（包括 ADDS 中不存在的用户）启用对 SaaS 应用的访问。
3. Bob 配置通用 LDAP 连接器用于从 Azure AD Connect 中的 ADLDS 提取数据。
4. Bob 创建同步规则，以便将 LDAP 用户填充到 Metaverse 和 Azure AD 中
5. Susie（LDAP 用户）使用同步的标识访问其 SaaS 应用



> [AZURE.IMPORTANT] 
> 这是一项高级配置，需要对 FIM/MIM 有一定的了解。 如果在生产环境中使用，我们建议浏览[顶级支持](https://support.microsoft.com/zh-cn/premier)了解有关此配置的问题。



## <a name="theme---increase-your-security"></a>主题 - 增强安全性 

| 方案 | 构建基块| 
| --- | --- |  
| [确保管理员帐户访问安全](#secure-administrator-account-access) | [使用电话呼叫执行 Azure MFA](/documentation/articles/active-directory-playbook-building-blocks/#azure-multi-factor-authentication-with-phone-calls/) |
| [使用基于证书的身份验证，在没有密码的情况下进行身份验证](#authenticate-without-passwords-using-certificate-based-authentication) | [配置基于证书的身份验证](/documentation/articles/active-directory-playbook-building-blocks/#configuring-certificate-based-authentication/)

### <a name="secure-administrator-account-access"></a>确保对管理员帐户的访问

1. Bob 是 Azure AD 全局管理员。 他将 Stuart 指定为服务的共同管理员。 
2. 为了提高安全性，Bob 将 Stuart 的帐户配置为始终需要执行 MFA
3. 登录到 Azure 门户预览时，Stuart 发现需要注册电话号码才能继续登录
4. Stuart 的后续登录受到多重身份验证的保护，现在他需要拨打电话来验证自己的身份。

### <a name="authenticate-without-passwords-using-certificate-based-authentication"></a>使用基于证书的身份验证，在没有密码的情况下进行身份验证

1. Bob 是某家金融机构的全局管理员，该机构禁止使用密码作为应用程序的身份验证因素。
2. Bob 在 ADFS 和 Azure AD 中启用并实施证书身份验证
3. Susie 访问应用程序时，系统提示她使用证书进行身份验证

[AZURE.INCLUDE [active-directory-playbook-toc](../../includes/active-directory-playbook-steps.md)]

