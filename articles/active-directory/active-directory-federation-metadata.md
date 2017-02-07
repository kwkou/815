<properties
    pageTitle="Azure AD 联合元数据 | Azure"
    description="本文介绍 Azure Active Directory 针对接受 Azure Active Directory 令牌的服务发布的联合元数据文档。"
    services="active-directory"
    documentationcenter=".net"
    author="priyamohanram"
    manager="mbaldwin"
    editor="" />
<tags
    ms.assetid="c2d5f80b-aa74-452c-955b-d8eb3ed62652"
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/07/2017"
    wacn.date="02/07/2017"
    ms.author="priyamo" />  


# 联合元数据

对于配置为接受 Azure Active Directory 颁发的安全令牌的服务，Azure AD 发布了一个联合元数据文档。在扩展了 [OASIS 安全断言标记语言 (SAML) v2.0 元数据](http://docs.oasis-open.org/security/saml/v2.0/saml-metadata-2.0-os.pdf)的 [Web 服务联合语言（WS 联合身份验证）版本 1.2](http://docs.oasis-open.org/wsfed/federation/v1.2/os/ws-federation-1.2-spec-os.html) 中描述了联合元数据文档格式。

## 特定于租户和独立于租户的元数据终结点

Azure AD 发布了特定于租户和独立于租户的终结点。

特定于租户的终结点面向特定的租户。特定于租户的联合元数据包含有关租户的信息，包括特定于租户的颁发者和终结点信息。限制访问单个租户的应用程序使用特定于租户的终结点。

独立于租户的终结点提供所有 Azure AD 租户通用的信息。此信息适用于托管在 *login.microsoftonline.com* 上的租户，并在租户间共享。对于多租户应用程序，建议使用独立于租户的终结点，因为它们不与任何特定租户相关联。

## 联合元数据终结点

Azure AD 会在 `https://login.microsoftonline.com/<TenantDomainName>/FederationMetadata/2007-06/FederationMetadata.xml` 上发布联合元数据。

对于**特定于租户的终结点**，`TenantDomainName` 可以是以下类型之一：

- Azure AD 租户的已注册域名，例如：`contoso.partner.onmschina.cn`
- 域的不可变租户 ID，例如 `72f988bf-86f1-41af-91ab-2d7cd011db45`。

对于**独立于租户的终结点**，`TenantDomainName` 为 `common`。此文档仅列出了托管在 login.microsoftonline.com 上的所有 Azure AD 租户通用的联合元数据元素。

例如，特定于租户的终结点可以是 `https:// login.microsoftonline.com/contoso.partner.onmschina.cn/FederationMetadata/2007-06/FederationMetadata.xml`。独立于租户的终结点为 [https://login.microsoftonline.com/common/FederationMetadata/2007-06/FederationMetadata.xml](https://login.microsoftonline.com/common/FederationMetadata/2007-06/FederationMetadata.xml)。你可以在浏览器中键入此 URL 以查看联合元数据文档。

## 联合元数据的内容

以下部分提供使用 Azure AD 颁发的令牌的服务所需的信息。

### 实体 ID

`EntityDescriptor` 元素包含 `EntityID` 属性。`EntityID` 属性的值表示颁发者，即颁发令牌的安全令牌服务 (STS)。请务必在收到令牌时验证颁发者。

以下元数据显示了包含 `EntityID` 元素的特定于租户的 `EntityDescriptor` 元素的示例。

    <EntityDescriptor 
    xmlns="urn:oasis:names:tc:SAML:2.0:metadata" 
    ID="_b827a749-cfcb-46b3-ab8b-9f6d14a1294b" 
    entityID="https://sts.chinacloudapi.cn/72f988bf-86f1-41af-91ab-2d7cd011db45/">

可以将独立于租户的终结点中的租户 ID 替换为你的租户 ID，以创建特定于租户的 `EntityID` 值。生成的值将与令牌颁发者的值相同。该策略允许多租户应用程序验证给定租户的颁发者。

以下元数据显示了独立于租户的 `EntityID` 元素的示例。请注意，`{tenant}` 是文本而不是占位符。

    <EntityDescriptor 
    xmlns="urn:oasis:names:tc:SAML:2.0:metadata" 
    ID="="_0e5bd9d0-49ef-4258-bc15-21ce143b61bd" 
    entityID="https://sts.chinacloudapi.cn/{tenant}/">

### 令牌签名证书
当服务收到 Azure AD 租户颁发的令牌时，必须使用联合元数据文档中发布的签名密钥来验证该令牌的签名。联合元数据包含租户用来进行令牌签名的证书的公共部分。证书原始字节显示在 `KeyDescriptor` 元素中。仅当 `use` 属性值为 `signing` 时，才可以使用令牌签名证书进行签名。

Azure AD 发布的联合元数据文档可以包含多个签名密钥，例如，当 Azure AD 准备更新签名证书时。如果联合元数据文档包含多个证书，验证令牌的服务应该支持文档中的所有证书。

以下元数据显示了一个包含签名密钥的 `KeyDescriptor` 元素的示例。

    <KeyDescriptor use="signing">
    <KeyInfo xmlns="http://www.w3.org/2000/09/xmldsig#">
    <X509Data>
    <X509Certificate>
    MIIDPjCCAiqgAwIBAgIQVWmXY/+9RqFA/OG9kFulHDAJBgUrDgMCHQUAMC0xKzApBgNVBAMTImFjY291bnRzLmFjY2Vzc2NvbnRyb2wud2luZG93cy5uZXQwHhcNMTIwNjA3MDcwMDAwWhcNMTQwNjA3MDcwMDAwWjAtMSswKQYDVQQDEyJhY2NvdW50cy5hY2Nlc3Njb250cm9sLndpbmRvd3MubmV0MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEArCz8Sn3GGXmikH2MdTeGY1D711EORX/lVXpr+ecGgqfUWF8MPB07XkYuJ54DAuYT318+2XrzMjOtqkT94VkXmxv6dFGhG8YZ8vNMPd4tdj9c0lpvWQdqXtL1TlFRpD/P6UMEigfN0c9oWDg9U7Ilymgei0UXtf1gtcQbc5sSQU0S4vr9YJp2gLFIGK11Iqg4XSGdcI0QWLLkkC6cBukhVnd6BCYbLjTYy3fNs4DzNdemJlxGl8sLexFytBF6YApvSdus3nFXaMCtBGx16HzkK9ne3lobAwL2o79bP4imEGqg+ibvyNmbrwFGnQrBc1jTF9LyQX9q+louxVfHs6ZiVwIDAQABo2IwYDBeBgNVHQEEVzBVgBCxDDsLd8xkfOLKm4Q/SzjtoS8wLTErMCkGA1UEAxMiYWNjb3VudHMuYWNjZXNzY29udHJvbC53aW5kb3dzLm5ldIIQVWmXY/+9RqFA/OG9kFulHDAJBgUrDgMCHQUAA4IBAQAkJtxxm/ErgySlNk69+1odTMP8Oy6L0H17z7XGG3w4TqvTUSWaxD4hSFJ0e7mHLQLQD7oV/erACXwSZn2pMoZ89MBDjOMQA+e6QzGB7jmSzPTNmQgMLA8fWCfqPrz6zgH+1F1gNp8hJY57kfeVPBiyjuBmlTEBsBlzolY9dd/55qqfQk6cgSeCbHCy/RU/iep0+UsRMlSgPNNmqhj5gmN2AFVCN96zF694LwuPae5CeR2ZcVknexOWHYjFM0MgUSw0ubnGl0h9AJgGyhvNGcjQqu9vd1xkupFgaN+f7P3p3EVN5csBg5H94jEcQZT7EKeTiZ6bTrpDAnrr8tDCy8ng
    </X509Certificate>
    </X509Data>
    </KeyInfo>
    </KeyDescriptor>

`KeyDescriptor` 元素出现在联合元数据文档中的两个位置：特定于 WS 联合身份验证的部分中，以及特定于 SAML 的部分中。在这两个部分中发布的证书将是相同的。

在特定于 WS 联合身份验证的部分中，WS 联合身份验证元数据读取器读取 `SecurityTokenServiceType` 类型的 `RoleDescriptor` 元素中的证书。

以下元数据显示了一个 `RoleDescriptor` 元素示例。


	<RoleDescriptor xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:fed="http://docs.oasis-open.org/wsfed/federation/200706" xsi:type="fed:SecurityTokenServiceType"protocolSupportEnumeration="http://docs.oasis-open.org/wsfed/federation/200706">


在特定于 SAML 的部分中，WS 联合身份验证元数据读取器读取 `IDPSSODescriptor` 元素中的证书。

以下元数据显示了一个 `IDPSSODescriptor` 元素示例。

    <IDPSSODescriptor protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol">

特定于租户和独立于租户的证书格式没有差别。

### WS 联合身份验证终结点 URL

联合元数据包括 Azure AD 用于在 WS 联合身份验证协议中进行单一登录和单一注销的 URL。此终结点显示在 `PassiveRequestorEndpoint` 元素中。

以下元数据显示了特定于租户的终结点的 `PassiveRequestorEndpoint` 元素示例。

    <fed:PassiveRequestorEndpoint>
    <EndpointReference xmlns="http://www.w3.org/2005/08/addressing">
    <Address>
        https://login.microsoftonline.com/72f988bf-86f1-41af-91ab-2d7cd011db45/wsfed
    </Address>
    </EndpointReference>
    </fed:PassiveRequestorEndpoint>

对于独立于租户的终结点，WS 联合身份验证 URL 显示在 WS 联合身份验证终结点中，如以下示例中所示。

    <fed:PassiveRequestorEndpoint>
    <EndpointReference xmlns="http://www.w3.org/2005/08/addressing">
    <Address>
    https://login.microsoftonline.com/common/wsfed
    </Address>
    </EndpointReference>
    </fed:PassiveRequestorEndpoint>

### SAML 协议终结点 URL

联合元数据包括 Azure AD 用于在 SAML 2.0 协议中进行单一登录和单一注销的 URL。这些终结点显示在 `IDPSSODescriptor` 元素中。

登录和注销 URL 分别显示在 `SingleSignOnService` 和 `SingleLogoutService` 元素中。

以下元数据显示了特定于租户的终结点的 `PassiveResistorEndpoint` 示例。

	<IDPSSODescriptor protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol">
	 …
	<SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect" Location="https://login.microsoftonline.com/contoso.partner.onmschina.cn/saml2" />
	<SingleSignOnService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect" Location="https://login.microsoftonline.com/contoso.partner.onmschina.cn/saml2" />
	</IDPSSODescriptor>

同样，通用 SAML 2.0 协议终结点的终结点发布在独立于租户的联合元数据中，如以下示例中所示。


	<IDPSSODescriptor protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol">
	…
	    <SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect" Location="https://login.microsoftonline.com/common/saml2" />
	    <SingleSignOnService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect" Location="https://login.microsoftonline.com/common/saml2" />
	  </IDPSSODescriptor>

<!---HONumber=Mooncake_0120_2017-->
<!---Update_Description: update meta properties -->