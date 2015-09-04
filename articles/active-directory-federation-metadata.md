<properties
   pageTitle="联合元数据"
   description="介绍元数据终结点，并说明了元数据文档中接受 Azure AD 令牌的服务必须使用的内容。"
   services="active-directory"
   documentationCenter="dev-center-name"
   authors="msmbaldwin"
   manager="mbaldwin"
   editor=""/>

<tags
   ms.service="active-directory"
   ms.date="05/30/2015"
   wacn.date="08/29/2015"/>

# 联合元数据
Azure Active Directory (Azure AD) 发布联合元数据文档，用于为接受 Azure Active Directory 发出的安全令牌所配置的服务。联合元数据文档格式在 [Web Services 联合身份验证语言（WS 联合身份验证）版本 1.2](http://docs.oasis-open.org/wsfed/federation/v1.2/os/ws-federation-1.2-spec-os.html) 中进行介绍，此版本对 [OASIS 安全声明标记语言 (SAML) V2.0 的元数据](http://docs.oasis-open.org/security/saml/v2.0/saml-metadata-2.0-os.pdf)进行了扩展。

本主题列出了元数据终结点，并介绍了元数据文档中接受 Azure AD 令牌的服务需使用的内容。

##特定于租户且独立于租户的元数据终结点

Azure AD 发布特定于租户且独立于租户的终结’点。特定于租户的终结点是针对特定租户设计的。特定于租户的联合元数据包括有关租户（包括特定于租户的颁发者）的信息和终结点信息。限制对单个租户的访问的应用程序使用特定于租户的终结点。

独立于租户的终结点提供对所有 Azure AD 租户通用的信息。此信息适用于托管在 *login.chinacloudapi.cn* 的租户，并跨租户共享。建议多租户应用程序使用独立于租户的终结点，因为它们与任何特定租户均不相关。

## 联合元数据终结点

Azure AD 将联合元数据发布在 *https://login.chinacloudapi.cn/<TenantDomainName>/FederationMetadata/2007-06/FederationMetadata.xml*，在此处，<TenantDomainName> 的值可以是“通用”或特定于租户的值。终结点是可寻址的，因此你可转到地址站点，查看租户的联合元数据。

对于**特定于租户的终结点**，<TenantDomainName> 可以是以下类型之一：

- Azure AD 租户的注册域名，例如：contoso.partner.onmschina.cn。

- 域的不可变租户 ID，例如 72f988bf-86f1-41af-91ab-2d7cd011db45。

对于**独立于租户的终结点**，<TenantDomainName> 为**通用**。此名称指示只能使用托管在 login.chinacloudapi.cn 上的所有 Azure AD 租户通用的联合元数据元素。

例如，特定于租户的终结点可以是 **https://login.chinacloudapi.cn/contoso.partner.onmschina.cnFederationMetadata/2007-06/FederationMetadata.xml*。独立于租户的终结点是 **https://login.chinacloudapi.cn/common/FederationMetadata/2007-06/FederationMetadata.xml*。

## 联合元数据的内容

以下部分提供使用 Azure AD 颁发的令牌的服务所需的信息。

### EntityID

**EntityDescriptor** 元素包含 **EntityID** 属性。**EntityID** 属性的值表示颁发者，即颁发令牌的安全令牌服务 (STS)。必须验证颁发者，以确认令牌是由哪个租户颁发的。

以下元数据显示了包含 **EntityID** 元素的特定于租户的 **EntityDescriptor** 元素示例。

    <EntityDescriptor 
    xmlns="urn:oasis:names:tc:SAML:2.0:metadata" 
    ID="_b827a749-cfcb-46b3-ab8b-9f6d14a1294b" 
    entityID="https://sts.chinacloudapi.cn/72f988bf-86f1-41af-91ab-2d7cd011db45/">

独立于租户的终结点中的 **EntityID** 提供了一个模板，可用来生成特定于租户的 **EntityID** 值。请将独立于租户的终结点中的“{tenant}”文本替换为令牌中 **TenantID** 声明的值。生成的值将与令牌颁发者的值相同。此策略允许多租户应用程序验证给定租户的颁发者。

以下元数据显示了独立于租户的 **EntityID** 元素示例。在此元素中，{tenant} 是一个文本，而不是占位符。

    <EntityDescriptor 
    xmlns="urn:oasis:names:tc:SAML:2.0:metadata" 
    ID="="_0e5bd9d0-49ef-4258-bc15-21ce143b61bd" 
    entityID="https://sts.chinacloudapi.cn/{tenant}/">

### 令牌签名证书
当服务收到 Azure AD 租户颁发的令牌时，必须使用联合元数据文档中发布的签名密钥来验证该令牌的签名。

联合元数据包含租户用来进行令牌签名的证书的公共部分。证书原始字节显示在 **KeyDescriptor** 元素中。仅当 **use** 属性值为 **signing** 时，才可以使用令牌签名证书进行签名。

Azure AD 发布的联合元数据文档可以包含多个签名密钥，例如，当 Azure AD 准备更新签名证书时。如果联合元数据文档包含多个证书，验证令牌的服务应该支持文档中的所有证书。

以下元数据显示了一个包含签名密钥的示例 **KeyDescriptor** 元素。

    <KeyDescriptor use="signing">
    <KeyInfo xmlns="http://www.w3.org/2000/09/xmldsig#">
    <X509Data>
    <X509Certificate>
    MIIDPjCCAiqgAwIBAgIQVWmXY/+9RqFA/OG9kFulHDAJBgUrDgMCHQUAMC0xKzApBgNVBAMTImFjY291bnRzLmFjY2Vzc2NvbnRyb2wud2luZG93cy5uZXQwHhcNMTIwNjA3MDcwMDAwWhcNMTQwNjA3MDcwMDAwWjAtMSswKQYDVQQDEyJhY2NvdW50cy5hY2Nlc3Njb250cm9sLndpbmRvd3MubmV0MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEArCz8Sn3GGXmikH2MdTeGY1D711EORX/lVXpr+ecGgqfUWF8MPB07XkYuJ54DAuYT318+2XrzMjOtqkT94VkXmxv6dFGhG8YZ8vNMPd4tdj9c0lpvWQdqXtL1TlFRpD/P6UMEigfN0c9oWDg9U7Ilymgei0UXtf1gtcQbc5sSQU0S4vr9YJp2gLFIGK11Iqg4XSGdcI0QWLLkkC6cBukhVnd6BCYbLjTYy3fNs4DzNdemJlxGl8sLexFytBF6YApvSdus3nFXaMCtBGx16HzkK9ne3lobAwL2o79bP4imEGqg+ibvyNmbrwFGnQrBc1jTF9LyQX9q+louxVfHs6ZiVwIDAQABo2IwYDBeBgNVHQEEVzBVgBCxDDsLd8xkfOLKm4Q/SzjtoS8wLTErMCkGA1UEAxMiYWNjb3VudHMuYWNjZXNzY29udHJvbC53aW5kb3dzLm5ldIIQVWmXY/+9RqFA/OG9kFulHDAJBgUrDgMCHQUAA4IBAQAkJtxxm/ErgySlNk69+1odTMP8Oy6L0H17z7XGG3w4TqvTUSWaxD4hSFJ0e7mHLQLQD7oV/erACXwSZn2pMoZ89MBDjOMQA+e6QzGB7jmSzPTNmQgMLA8fWCfqPrz6zgH+1F1gNp8hJY57kfeVPBiyjuBmlTEBsBlzolY9dd/55qqfQk6cgSeCbHCy/RU/iep0+UsRMlSgPNNmqhj5gmN2AFVCN96zF694LwuPae5CeR2ZcVknexOWHYjFM0MgUSw0ubnGl0h9AJgGyhvNGcjQqu9vd1xkupFgaN+f7P3p3EVN5csBg5H94jEcQZT7EKeTiZ6bTrpDAnrr8tDCy8ng
    </X509Certificate>
    </X509Data>
    </KeyInfo>
    </KeyDescriptor>

**KeyDescriptor** 元素出现在联合元数据文档中的两个位置：特定于 WS 联合身份验证的部分中，和特定于 SAML 的部分中。在这两个部分中发布的证书将是相同的。

在特定于 WS 联合身份验证的部分中，WS 联合身份验证元数据读取器将读取 **SecurityTokenServiceType** 类型的 **RoleDescriptor** 元素中的证书。

以下元数据显示了一个示例 **RoleDescriptor** 元素。

    <RoleDescriptor xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:fed="http://docs.oasis-open.org/wsfed/federation/200706"
    xsi:type="fed:SecurityTokenServiceType"protocolSupportEnumeration="http://docs.oasis-open.org/wsfed/federation/200706">`

在特定于 SAML 的部分中，WS 联合身份验证元数据读取器将读取 **IDPSSODescriptor** 元素中的证书。

以下元数据显示了一个示例 **IDPSSODescriptor** 元素。

    <IDPSSODescriptor protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol">

特定于租户和独立于租户的证书采用相同格式。

### WS 联合身份验证终结点 URL
联合元数据包括 Azure AD 用于在 WS 联合身份验证协议中进行单一登录和单一注销的 URL。此终结点显示在 **PassiveRequestorEndpoint** 元素中。

以下元数据显示了特定于租户的终结点的示例 **PassiveRequestorEndpoint** 元素。

    <fed:PassiveRequestorEndpoint>
    <EndpointReference xmlns="http://www.w3.org/2005/08/addressing">
    <Address>
    https://login.chinacloudapi.cn/72f988bf-86f1-41af-91ab-2d7cd011db45/wsfed
    </Address>
    </EndpointReference>
    </fed:PassiveRequestorEndpoint>

对于独立于租户的终结点，WS 联合身份验证 URL 显示在 WS 联合身份验证终结点中，如以下示例中所示。

    <fed:PassiveRequestorEndpoint>
    <EndpointReference xmlns="http://www.w3.org/2005/08/addressing">
    <Address>
    https://login.chinacloudapi.cn/common/wsfed
    </Address>
    </EndpointReference>
    </fed:PassiveRequestorEndpoint>

### SAML 协议终结点 URL

联合元数据包括 Azure AD 用于在 SAML 2.0 协议中进行单一登录和单一注销的 URL。这些终结点显示在 **IDPSSODescriptor** 元素中。

登录和注销 URL 显示在 **SingleSignOnService** 和 **SingleLogoutService** 元素中。以下元数据显示了特定于租户的终结点的示例 **PassiveResistorEndpoint**。

    <IDPSSODescriptor protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol">
    …
    <SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect" Location="https://login.chinacloudapi.cn/contoso.partner.onmschina.cn/saml2" />
    <SingleSignOnService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect" Location="https:// login.chinacloudapi.cn/contoso.partner.onmschina.cn /saml2" />
    </IDPSSODescriptor>

同样，在独立于租户的联合元数据中发布通用 SAML 2.0 协议终结点的终结点，如以下示例中所示。

    <IDPSSODescriptor protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol">
    …
    <SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect" Location="https://login.chinacloudapi.cn/common/saml2" />
    <SingleSignOnService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect" Location="https://login.chinacloudapi.cn/common/saml2" />
    </IDPSSODescriptor>

## 另请参阅
[Azure Active Directory 身份验证协议](/documentation/articles/active-directory-authentication-protocols)

[Azure Active Directory 开发人员指南](/documentation/articles/active-directory-developers-guide)

<!---HONumber=67-->