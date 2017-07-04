<properties
    pageTitle="如何与 Azure Active Directory 集成 | Azure"
    description="介绍与 Azure Active Directory 集成的好处与相关资源的指南。"
    services="active-directory"
    documentationcenter="dev-center-name"
    author="bryanla"
    manager="mbaldwin"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="d13bba54-96bd-4b81-bee9-c8025ffa1648"
    ms.service="active-directory"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="identity"
    ms.date="01/07/2017"
    wacn.date="05/02/2017"
    ms.author="mbaldwin"
    ms.sourcegitcommit="78da854d58905bc82228bcbff1de0fcfbc12d5ac"
    ms.openlocfilehash="5cb663ae5da749691abaefe7268fc668a923c4c4"
    ms.lasthandoff="04/22/2017" />

# <a name="integrating-with-azure-active-directory"></a>与 Azure Active Directory 集成
[AZURE.INCLUDE [active-directory-devguide](../../includes/active-directory-devguide.md)]

Azure Active Directory 为组织的云应用程序提供企业级标识管理。  Azure AD 集成可以简化用户登录体验，并帮助应用程序符合 IT 策略。

## <a name="how-to-integrate"></a>集成方式
应用程序可通过多种方式与 Azure AD 集成。  请根据你的应用程序，利用其中的一个或多个方案。

### <a name="support-azure-ad-as-a-way-to-sign-in-to-your-application"></a>支持使用 Azure AD 作为登录应用程序的方式
**减少登录问题并降低支持成本。** 如果使用 Azure AD 登录应用程序，用户不需要记住更多的名称和密码。  作为开发人员，你可以减少要存储和保护的密码。  无需重置忘记的密码，单凭这一点就能节省不少的精力。  使用 Azure AD 可以登录世界上最热门的一些云应用程序，包括 Office 365 和 Azure。  Azure AD 包含了来自数百万家组织的几亿个用户，你的用户很可能已经登录到了 Azure AD。  深入了解如何[添加 Azure AD 登录支持](/documentation/articles/active-directory-authentication-scenarios/)。

**简化应用程序注册。**  在注册应用程序期间，Azure AD 可以发送有关用户的基本信息，以便你可以预先填写注册表单，或者完全清除表单。  用户可以使用其 Azure AD 帐户，通过社交媒体和移动应用程序中常见的许可体验注册应用程序。  任何用户在不需要 IT 人员的情况下都可以注册和登录与 Azure AD 集成的应用程序。  深入了解如何[注册应用程序进行 Azure AD 帐户登录](/documentation/articles/app-service-mobile-how-to-configure-active-directory-authentication/)。

### <a name="browse-for-users-manage-user-provisioning-and-control-access-to-your-application"></a>浏览用户，管理用户设置，以及控制对应用程序的访问
**浏览目录中的用户。**  在邀请其他人或授予访问权限时，可以使用图形 API 来帮助用户搜索和浏览其组织中的其他人员，而无需键入电子邮件地址。  用户可以使用熟悉的通讯簿样式界面进行浏览，包括查看组织层次结构的详细信息。  深入了解[图形 API](/documentation/articles/active-directory-graph-api/)。

**重复使用你的客户正在管理的 Active Directory 组和通讯组列表。**  Azure AD 包含你的客户已用于电子邮件分发和管理访问权限的组。  使用图形 API 时，可以重复使用这些组，而无需要求客户在应用程序中创建并管理一系列不同的组。  还可以在登录令牌中向应用程序发送组信息。  深入了解[图形 API](/documentation/articles/active-directory-graph-api/)。

**使用 Azure AD 控制谁有权访问你的应用程序。**  Azure AD 中的管理员和应用程序所有者可以将应用程序访问权限分配给特定的用户和组。  使用图形 API，可以读取此列表并使用它来控制资源的设置和取消设置，以及应用程序中的访问权限。

**使用 Azure AD 进行基于角色的访问控制。**  管理员和应用程序所有者可以向你在 Azure AD 中注册应用程序时定义的角色分配用户和组。  角色信息将在登录令牌中发送到应用程序，并可使用图形 API 来读取。  了解有关 [使用 Azure AD 进行授权](http://blogs.technet.com/b/ad/archive/2014/12/18/azure-active-directory-now-with-group-claims-and-application-roles.aspx)的详细信息。

### <a name="get-access-to-users-profile-calendar-email-contacts-files-and-more"></a>获取对用户配置文件、日历、电子邮件、联系人、文件等的访问权限
**Azure AD 是 Office 365 和其他 Microsoft 业务服务的授权服务器。**  如果你支持使用 Azure AD 登录到你的应用程序，或者支持将当前用户帐户链接到使用 OAuth 2.0 的 Azure AD 用户帐户，则可以请求对用户配置文件、日历、电子邮件、联系人、文件和其他信息的读取和写入访问权限。  你可以一气呵成地将事件写入用户日历，并在其 OneDrive 中读取或写入文件。  了解有关 [访问 Office 365 API](https://msdn.microsoft.com/office/office365/howto/platform-development-overview)的详细信息。

### <a name="promote-your-application-in-the-azure-and-office-365-marketplaces"></a>在 Azure 和 Office 365 应用商店中推广应用程序
**向数百万个已在使用 Azure AD 的组织推广应用程序。**  搜索和浏览这些应用商店的用户已在使用一个或多个云服务，这让他们成为合格的云服务客户。 

**用户注册应用程序后，该程序将在其 Azure AD 访问面板和 Office 365 应用启动器中显示。**  之后，用户能够快速轻松地返回应用程序，提高了用户参与度。  

### <a name="secure-device-to-service-and-service-to-service-communication"></a>保护设备与服务之间以及服务与服务之间的通信
**使用 Azure AD 进行服务和设备的标识管理可以减少编写代码以及让 IT 人员管理访问权限产生的成本。**  服务和设备可以使用 OAuth 从 Azure AD 获取令牌，并使用这些令牌来访问 Web API。  使用 Azure AD 可以避免编写复杂的身份验证代码。  由于服务和设备的标识存储在 Azure AD 中，IT 人员可以在一个位置管理密钥和吊销，而无需单独在应用程序中执行此操作。

## <a name="benefits-of-integration"></a>集成的好处
与 Azure AD 集成带来的好处是无需编写额外的代码。

### <a name="integration-with-enterprise-identity-management"></a>与企业标识管理集成
**帮助应用程序符合 IT 策略。**  组织可将其企业标识管理系统与 Azure AD 集成，这样，在员工离开组织后，他们将自动失去对应用程序的访问权限，而不需要 IT 人员采取额外的措施。  IT 人员可以控制谁可以访问你的应用程序，并确定需要哪些访问策略（例如多重身份验证），这就减少了为遵守复杂的企业策略而要编写的代码量。  Azure AD 为管理员提供详细的审核日志，其中记录了哪些人登录了你的应用程序，IT 人员可以跟踪使用情况。

**Azure AD 已将 Active Directory 扩展到云中，以便你的应用程序可与 AD 集成。**  世界各地的许多组织都在使用 Active Directory 作为首要登录和标识管理系统，并要求它们的应用程序与 AD 配合使用。  与 Azure AD 集成可将你的应用与 Active Directory 相集成。

### <a name="advanced-security-features"></a>高级安全功能
**多重身份验证。**  Azure AD 提供本机多重身份验证。  IT 管理员可以要求访问应用程序之前经过多重身份验证，因此你无需编写此项支持的代码。  了解有关 [多重身份验证](/documentation/services/multi-factor-authentication/)的详细信息。

### <a name="easy-development"></a>易于开发
**行业标准协议。**  Microsoft 承诺支持行业标准。  Azure AD 支持 SAML 2.0、OpenID Connect 1.0、OAuth 2.0 和 WS-Federation 1.2 身份验证协议。  图形 API 符合 OData 4.0 规范。  如果你的应用程序已支持使用 SAML 2.0 或 OpenID Connect 1.0 进行联合登录，可以直接添加对 Azure AD 的支持。  深入了解 [Azure AD 支持的身份验证协议](/documentation/articles/active-directory-authentication-protocols/)。

**开放源代码库。**  Microsoft 为主流语言和平台提供完全支持的开源代码库以加速开发。  这些源代码已获 Apache 2.0 的授权，可以在项目中任意衍生和改写。  深入了解 [Azure AD 身份验证库](/documentation/articles/active-directory-authentication-libraries/)。

### <a name="worldwide-presence-and-high-availability"></a>全球存在和高可用性
**Azure AD 已部署在世界各地的数据中心，并受到全天候的管理和监视。**  Azure AD 是 Azure 和 Office 365 的标识管理系统，已在世界各地的 28 个数据中心部署。  我们保证至少将目录数据复制到三个数据中心。  全局负载均衡器确保用户访问包含其数据的最近 Azure AD 副本，如果检测到问题，会自动将请求重新路由到其他数据中心。

## <a name="next-steps"></a>后续步骤
[开始编写代码](/documentation/articles/active-directory-developers-guide/#getting-started/)。

[使用 Azure AD 登录用户](/documentation/articles/active-directory-authentication-scenarios/)

<!--Update_Description: wording update-->