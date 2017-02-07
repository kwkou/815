<properties
    pageTitle="Azure AD SAML 协议参考 | Azure"
    description="本文概述 Azure Active Directory 中的单一登录和单一注销 SAML 配置文件。"
    services="active-directory"
    documentationcenter=".net"
    author="priyamohanram"
    manager="mbaldwin"
    editor="" />
<tags
    ms.assetid="88125cfc-45c1-448b-9903-a629d8f31b01"
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/07/2017"
    wacn.date="02/07/2017"
    ms.author="priyamo" />  


# Azure Active Directory 如何使用 SAML 协议
Azure Active Directory (Azure AD) 使用 SAML 2.0 协议，使应用程序能够为其用户提供单一登录体验。Azure AD 的[单一登录](/documentation/articles/active-directory-single-sign-on-protocol-reference/)和[单一注销](/documentation/articles/active-directory-single-sign-out-protocol-reference/) SAML 配置文件说明了标识提供者服务中如何使用 SAML 断言、协议和绑定。

SAML 协议要求标识提供者 (Azure AD) 与服务提供者（应用程序）交换有关自身的信息。

将应用程序注册到 Azure AD 时，应用开发人员需将与联合身份验证相关的信息注册到 Azure AD。这包括应用程序的**重定向 URI** 和**元数据 URI**。

Azure AD 使用云服务的**元数据 URI** 来检索云服务的签名密钥和注销 URI。如果应用程序不支持元数据 URI，开发人员必须联系 Microsoft 支持人员，让其提供注销 URI 和签名密钥。

Azure Active Directory 会公开特定于租户的和公用的（独立于租户的）单一登录和单一注销终结点。这些 URL 表示可寻址位置（不只是标识符），方便你转到终结点读取元数据。

- 特定于租户的终结点位于 `https://login.microsoftonline.com/<TenantDomainName>/FederationMetadata/2007-06/FederationMetadata.xml`。<TenantDomainName> 占位符表示 Azure AD 租户的已注册域名或 TenantID GUID。例如，contoso.com 租户的联合元数据位于：https://login.microsoftonline.com/contoso.com/FederationMetadata/2007-06/FederationMetadata.xml

- 独立于租户的终结点位于 `https://login.microsoftonline.com/common/FederationMetadata/2007-06/FederationMetadata.xml`。此终结点地址中显示**公用终结点**，而不是租户域名或 ID。

有关 Azure AD 发布的联合元数据文档的信息，请参阅 [Federation Metadata（联合元数据）](/documentation/articles/active-directory-federation-metadata/)。

<!---HONumber=Mooncake_0120_2017-->
<!---Update_Description: wording update -->