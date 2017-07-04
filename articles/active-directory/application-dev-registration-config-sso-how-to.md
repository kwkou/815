<properties
    pageTitle="如何配置新的多租户应用程序 | Azure"
    description="如何为通过 Azure AD 进行开发和注册的自定义应用程序配置单一登录。"
    services="active-directory"
    documentationcenter=""
    author="ajamess"
    manager="femila"
    translationtype="Human Translation" />
<tags
    ms.assetid=""
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="04/04/2017"
    wacn.date="05/02/2017"
    ms.author="asteen"
    ms.sourcegitcommit="78da854d58905bc82228bcbff1de0fcfbc12d5ac"
    ms.openlocfilehash="8161a8606118cebc126bf41f8927d07b94d9b343"
    ms.lasthandoff="04/22/2017" />

# <a name="how-to-configure-a-new-multi-tenant-application"></a>如何配置新的多租户应用程序

通过用于 OpenID Connect、SAML 2.0 或 WS-Fed 的 Azure AD 进行联合身份验证时，可以在应用中自动启用联合单一登录 (SSO)。 假如最终用户已经通过 Azure AD 建立了一个会话，但仍需登录，则可能是因为应用配置错误。

- 如果使用 ADAL/MSAL，请确保将 **PromptBehavior** 设置为“Auto”而非“Always”。

- 若要生成移动应用，可能需要进行其他配置才能启用中转或非中转 SSO。

对于 Android，请参阅[在 Android 中启用跨应用 SSO](/documentation/articles/active-directory-sso-android/)。<br>

对于 iOS，请参阅[在 iOS 中启用跨应用 SSO](/documentation/articles/active-directory-sso-ios/)。

## <a name="next-steps"></a>后续步骤

[Azure AD SSO](/documentation/articles/active-directory-appssoaccess-whatis/)<br>

[在 Android 中启用跨应用 SSO](/documentation/articles/active-directory-sso-android/)<br>

[在 iOS 中启用跨应用 SSO](/documentation/articles/active-directory-sso-ios/)<br>

[AzureAD v2.0 聚合应用的许可和权限](/documentation/articles/active-directory-v2-scopes/)<br>

[AzureAD StackOverflow](http://stackoverflow.com/questions/tagged/azure-active-directory)

