<properties
    pageTitle="Azure Active Directory 预览版中针对企业应用的单一登录管理 | Azure"
    description="了解如何通过 Azure Active Directory 管理企业应用的单一登录"
    services="active-directory"
    documentationCenter=""
    authors="asmalser"
    manager="stevenpo"
    editor=""/>

<tags
    ms.service="active-directory"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="identity"
    ms.date="09/12/2016"
    ms.author="asmalser"
    wacn.date="10/11/2016"/>

#预览：在新的 Azure 门户中管理企业应用的单一登录

本文介绍如何使用 [Azure 门户](https://portal.azure.cn)管理应用程序的单一登录设置，尤其是已从 [Azure Active Directory (Azure AD) 应用程序库](/documentation/articles/active-directory-appssoaccess-whatis/#get-started-with-the-azure-ad-application-gallery/)中添加的设置。单一登录的 Azure AD 管理体验目前为公共预览版。本文介绍仅在预览期间提供的新功能以及一些临时限制。[预览版内容](/documentation/articles/active-directory-preview-explainer/)

##在新门户中查找应用

从 2016 年 9 月开始，所有已在目录中被目录管理员使用 [Azure 经典管理门户](https://manage.windowsazure.cn)中的 [Azure Active Directory 应用程序库](/documentation/articles/active-directory-appssoaccess-whatis/#get-started-with-the-azure-ad-application-gallery/)进行过单一登录配置的应用程序均可在 Azure 门户中查看和管理。

这些应用程序可在 Azure 门户的"企业应用程序"部分查找，可在[门户](https://portal.azure.cn)的"更多服务"列表中查找相关链接。企业应用是已在组织中部署并且正由用户使用的应用。

!["企业应用程序"边栏选项卡][1]

选择"所有应用程序"即可查看包含所有已配置应用的列表，包括那些已从库中添加的应用。选择某个应用时，会加载该应用的资源边栏选项卡，可在其中查看该应用的报告，以及管理各种设置。

若要管理单一登录设置，请选择"单一登录"。

!["应用程序资源"边栏选项卡][2]


##单一登录模式

在"单一登录"边栏选项卡中，一开始是"模式"菜单，用于配置单一登录模式。可用选项包括：

* **基于 SAML 的登录** - 如果应用程序允许使用 SAML 2.0 协议通过 Azure Active Directory 进行完整的联合身份验证单一登录，则会提供此选项。此

* **基于密码的登录** - 如果 Azure AD 允许对此应用程序使用密码来填写表单，则会提供此选项。

* **联合登录** - 以前称为"现有单一登录"。此选项允许管理员在其用户的 Azure AD 访问面板或 Office 3645 应用程序启动程序中放置一个此应用程序的链接。

有关这些模式的详细信息，请参阅 [How does single sign-on with Azure Active Directory work](/documentation/articles/active-directory-appssoaccess-whatis/#how-does-single-sign-on-with-azure-active-directory-work/)（通过 Azure Active Directory 实现单一登录的原理）。


##基于 SAML 的登录

"基于 SAML 的登录"选项显示的边栏选项卡分四部分：

###域和 URL
可以在这里将应用程序的域和 URL 的所有详细信息添加到 Azure AD 目录。进行单一登录所需的所有输入直接显示在屏幕上，而所有可选输入则可通过选择"显示高级 URL 设置"复选框来查看。受支持输入的完整列表包括：

* **登录 URL** - 用户在这里登录到此应用程序。如果将应用程序配置为执行服务提供者发起的单一登录，则当用户导航到此 URL 时，服务提供者会进行必要的重定向，重定向到 Azure AD 完成用户身份验证和登录。如果填充了此字段，则 Azure AD 会使用此 URL 从 Office 365 和 Azure AD 访问面板启动应用程序。如果忽略了此字段，则从 Office 365、Azure AD 访问面板或 Azure AD 单一登录 URL 启动应用时，Azure AD 会执行标识提供者发起的登录。

* **标识符** - 可通过此 URI 以独特的方式标识正为其配置单一登录的应用程序。这是 Azure AD 发送回应用程序的值，作为 SAML 令牌的 Audience 参数，预计由应用程序对其进行验证。该值也在应用程序提供的任何 SAML 元数据中显示为实体 ID。

* **回复 URL** - 应用程序通常会在回复 URL 中接收 SAML 令牌。也称断言使用者服务 (ACS) URL。输入这些项目以后，即可单击"下一步"转到下一屏幕。该屏幕说明需在应用程序端进行什么配置才能接受来自 Azure AD 的 SAML 令牌。

* **中继状态** - 中继状态是一个可选参数，可用于告知应用程序：在完成身份验证以后，需将用户重定向到何处。该值通常是应用程序的有效 URL，但某些应用程序会将此字段用于其他用途（有关详细信息，请参阅应用的单一登录文档）。设置中继状态功能是新的 Azure 门户独有的新功能。

###用户属性
管理员可在此项中查看和编辑在 SAML 令牌中发送的属性，该令牌由 Azure AD 在每次用户登录时颁发给应用程序。

第一个预览版本支持的唯一可编辑属性是"用户标识符"属性。此属性的值是 Azure AD 中的字段，用于唯一标识应用程序中的每个用户。例如，如果使用"电子邮件地址"作为用户名和唯一标识符对应用进行了部署，则可将该值设置为 Azure AD 中的"user.mail"字段。

后续预览版将支持编辑其他属性。

###SAML 签名证书
此部分显示 Azure AD 用来签署 SAML 令牌的证书的详细信息，此类令牌在每次进行用户身份验证时颁发给应用程序。这样即可检查当前证书的属性，包括到期日期。

后续预览版将支持滚动更新证书以及管理其他证书选项的功能。请注意，仍可在 [Azure 经典管理门户](/documentation/articles/active-directory-sso-certs/)中执行对证书的完整管理。

###应用程序配置
最后一部分提供必需的文档和/或控件，用于对应用程序本身进行配置，使用 Azure Active Directory 作为标识提供者。

"配置应用程序"飞出式菜单提供进行应用程序配置所需的说明，这些说明既新又简洁，而且是嵌入式的。这是新的 Azure 门户独有的另一新功能。

> [AZURE.NOTE] 如需嵌入式文档的完整示例，请参阅 Salesforce.com 应用程序。预览期间将持续添加其他应用的文档。

![嵌入式文档][3]

##基于密码的登录
如果应用程序支持，可选择基于密码的 SSO 模式，然后选择"保存"，这样即可将其配置为进行基于密码的 SSO。如需详细了解如何部署基于密码的 SSO，请参阅 [How does single sign-on with Azure Active Directory work](/documentation/articles/active-directory-appssoaccess-whatis/#how-does-single-sign-on-with-azure-active-directory-work/)（通过 Azure Active Directory 实现单一登录的原理）。

![基于密码的登录][4]


##联合登录
如果应用程序支持，可选择联合 SSO 模式，然后即可输入一个 URL，该 URL 是用户单击此应用时，Azure AD 访问面板或 Office 365 的重定向目标。有关联合 SSO（以前称为"现有 SSO"）的详细信息，请参阅 [How does single sign-on with Azure Active Directory work](/documentation/articles/active-directory-appssoaccess-whatis/#how-does-single-sign-on-with-azure-active-directory-work/)（通过 Azure Active Directory 实现单一登录的原理）。

![联合登录][5]

[1]: ./media/active-directory-enterprise-apps-manage-sso/enterprise-apps-blade.PNG
[2]: ./media/active-directory-enterprise-apps-manage-sso/enterprise-apps-sso-blade.PNG
[3]: ./media/active-directory-enterprise-apps-manage-sso/enterprise-apps-blade-embedded-docs.PNG
[4]: ./media/active-directory-enterprise-apps-manage-sso/enterprise-apps-blade-password-sso.PNG
[5]: ./media/active-directory-enterprise-apps-manage-sso/enterprise-apps-blade-linked-sso.PNG

<!---HONumber=Mooncake_0926_2016-->
