<properties 
   pageTitle="支持的令牌和声明类型"
   description="本指南帮助你了解和评估 Azure Active Directory (AAD) 颁发的 SAML 2.0 令牌和 JSON Web 令牌 (JWT) 令牌中的声明。"
   documentationCenter="dev-center-name"
   authors="msmbaldwin"
   services="active-directory" 
   manager="mbaldwin"
   editor=""/>

<tags
   ms.service="active-directory"
   ms.date="09/17/2015"
   wacn.date="01/29/2016"/>

# 支持的令牌和声明类型

本主题旨在帮助你了解和评估 Azure Active Directory (Azure AD) 颁发的 SAML 2.0 令牌和 JSON Web 令牌 (JWT) 令牌中的声明。

本主题以每种令牌声明的说明开头，并根据需要举例说明了 SAML 令牌和 JWT 令牌中的声明。处于预览状态的声明将单独列出。本主题以令牌示例结束以便你可以查看上下文中的声明。

Azure 会不断向令牌中添加声明，以便启用新方案。通常，我们会介绍这些处于预览状态的声明，然后在测试期间过后将它们转换为完全支持。为了给声明更改做准备，从 Azure AD 接受令牌的应用程序应忽略不熟悉的令牌声明，以使新声明不会中断应用程序。使用处于预览状态的声明的应用程序不应依赖这些声明，并且在声明未出现在令牌中时不应引发异常。如果你的应用程序需要 Azure AD 颁发的 SAML 或 JWT 令牌中未提供的声明，请使用此页底部的“社区加码”部分建议和讨论新声明类型。

## 令牌声明参考

此部分列出并说明 Azure AD 返回的令牌中的声明。它包括声明的 SAML 版本和 JWT 版本、声明的说明及其用法。这些声明按字母顺序列出。

### 应用程序 ID

此应用程序 ID 声明标识使用令牌访问资源的应用程序。该应用程序可以自身名义或者代表用户进行操作。应用程序 ID 通常表示应用程序对象，但它还可以表示 Azure AD 中的服务主体对象。

Azure AD 不支持 SAML 令牌中的应用程序 ID 声明。

在 JWT 令牌中，应用程序 ID 出现在appid 声明中。

    "appid":"15CB020F-3984-482A-864D-1D92265E8268"

### 目标受众
令牌的受众是令牌的预期接收者。接收令牌的应用程序必须验证受众值是否正确，并拒绝任何针对其他受众的令牌。

受众值是一个字符串，通常为所访问资源的基址，如“https://contoso.com”。在 Azure AD 令牌中，受众是请求该令牌的应用程序的应用程序 ID URI 。当应用程序（即受众）具有多个应用程序 ID URI 时，该令牌的 Audience 声明中的应用程序 ID URI 将与令牌请求中的应用程序 ID URI 相匹配。
在 SAML 令牌中，Audience 声明在 AudienceRestriction 元素的 Audience 元素中定义。

    <AudienceRestriction>
    <Audience>https://contoso.com</Audience>
    </AudienceRestriction>

在 JWT 令牌中，受众出现在aud 声明中。

    "aud":"https://contoso.com"

### 应用程序身份验证上下文类引用

应用程序身份验证上下文类引用声明指示如何对客户端进行身份验证。对于公共客户端，该值为 0。如果使用客户端 ID 和客户端机密，则该值为 1。

在 JWT 令牌中，身份验证上下文类引用值出现在appidacr（特定于应用程序的 ACR 值）声明中。

    "appidacr": "0"

### 身份验证上下文类引用
身份验证上下文类引用声明指示如何对使用者进行身份验证，这一点与应用程序身份验证上下文类引用 声明中的客户端不同。值为“0”指示最终用户身份验证不符合 ISO/IEC 29115 要求。

- 在 JWT 令牌中，身份验证上下文类引用声明出现在 acr（特定于用户的 ACR 值）声明中。

    "acr": "0"

### 即时身份验证

即时身份验证声明记录进行身份验证的日期和时间。

在 SAML 令牌中，即时身份验证出现在 AuthnStatement 元素的 AuthnInstant 属性中。它表示 UTC (Z) 时间格式的日期时间。

    <AuthnStatement AuthnInstant="2011-12-29T05:35:22.000Z">

Azure AD 未在 JWT 令牌中提供等效声明。

### 身份验证方法

身份验证方法声明告知如何对令牌的使用者进行身份验证。在此示例中，标识提供程序使用密码对用户进行身份验证。http://schemas.microsoft.com/ws/2008/06/identity/claims/authenticationmethod/password

在 SAML 令牌中，身份验证方法值出现在 AuthnContextClassRef 元素中。

    <AuthnContextClassRef>http://schemas.microsoft.com/ws/2008/06/identity/claims/authenticationmethod/password</AuthnContextClassRef>

在 JWT 令牌中，身份验证方法值出现在 amr 声明中。

    “amr”: ["pwd"]

###名字

名字或“给定名称”声明提供 Azure AD 用户对象上设置的用户的名字或“给定”名称。

在 SAML 令牌中，名字（或“给定名称”）出现在 givenname SAML 属性元素的声明中。

    <Attribute Name=” http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname”>
    <AttributeValue>Frank<AttributeValue>

在 JWT 令牌中，名字出现在 given_name 声明中。

    "given_name": "Frank"

### 组

组 声明提供表示使用者的组成员身份的对象 ID。这些值是唯一的（请参阅对象 ID），可安全地用于管理访问，例如强制要求授权才能访问资源。组声明中包含的组通过应用程序清单的“groupMembershipClaims”属性，基于每个应用程序进行配置。值为 null 将排除所有组；值为“SecurityGroup”将只包括“Active Directory 安全组”成员身份；值为“All”将包括安全组和 Office 365 通讯组列表。在任何配置中，组声明均表示使用者的可传递组成员身份。

在 SAML 令牌中，组声明出现在 groups 属性中。

    <Attribute Name="http://schemas.microsoft.com/ws/2008/06/identity/claims/groups">
    <AttributeValue>07dd8a60-bf6d-4e17-8844-230b77145381</AttributeValue>

在 JWT 令牌中，组声明出现在 groups 声明中。

    “groups”: ["0e129f5b-6b0a-4944-982d-f776045632af", … ]

### 标识提供程序

标识提供程序声明记录对令牌的使用者进行身份验证的标识提供程序。除非用户帐户与颁发者不在同一租户中，否则此值与颁发者声明的值相同。

在 SAML 令牌中，标识提供程序出现在 identityprovider SAML 属性元素的声明中。

    <Attribute Name=” http://schemas.microsoft.com/identity/claims/identityprovider”>
    <AttributeValue>https://sts.chinacloudapi.cn/cbb1a5ac-f33b-45fa-9bf5-f37db0fed422/<AttributeValue>

在 JWT 令牌中，标识提供程序出现在 idp 声明中。

    "idp":”https://sts.chinacloudapi.cn/cbb1a5ac-f33b-45fa-9bf5-f37db0fed422/”

### IssuedAt

IssuedAt 声明存储颁发令牌的时间。它通常用于度量令牌新鲜度。在 SAML 令牌中，IssuedAt 值出现在 IssueInstant 断言中。

    <Assertion ID="_d5ec7a9b-8d8f-4b44-8c94-9812612142be" IssueInstant="2014-01-06T20:20:23.085Z" Version="2.0" xmlns="urn:oasis:names:tc:SAML:2.0:assertion">

在 JWT 令牌中，IssuedAt 值出现在 iat 声明中。该值用自协调世界时 (UTC) 的 1970-01-010:0:0Z 以来经过的秒数来表示。

    "iat": 1390234181

### 颁发者

颁发者声明标识构造并返回令牌的安全令牌服务 (STS) 和 Azure AD 目录租户。在 Azure AD 返回的令牌中，颁发者是 sts.chinacloudapi.cn。颁发者声明值中的 GUID 是 Azure AD 目录的租户 ID。租户 ID 是目录的固定不变且可靠的标识符。

在 SAML 令牌中，颁发者声明出现在 Issuer 元素中。

    <Issuer>https://sts.chinacloudapi.cn/cbb1a5ac-f33b-45fa-9bf5-f37db0fed422/</Issuer>

在 JWT 令牌中，颁发者出现在 iss 声明中。

    "iss":”https://sts.chinacloudapi.cn/cbb1a5ac-f33b-45fa-9bf5-f37db0fed422/”

### 姓氏

姓氏声明提供 Azure AD 用户对象中定义的用户的姓、姓氏或家族名称。在 SAML 令牌中，姓氏出现在 surname SAML 属性元素的声明中。

    <Attribute Name=” http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname”>
    <AttributeValue>Miller<AttributeValue>

在 JWT 令牌中，姓氏出现在 family_name 声明中。

    "family_name": "Miller"

### Name

此名称声明提供了标识令牌使用者的人工可读值。此值不一定在租户中唯一，它旨在仅用于显示目的。在 SAML 令牌中，名称出现在 Name 属性中。

    <Attribute Name=”http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name”>
    <AttributeValue>frankm@contoso.com<AttributeValue>

在 JWT 声明中，名称出现在 unique_name 声明中。

    "unique_name": "frankm@contoso.com"

### 对象 ID

对象 ID 声明包含 Azure AD 中的对象的唯一标识符。此值不可变且不能重新分配或重复使用，因此你可以使用它来安全地执行授权检查，例如，当使用令牌访问资源时。在对 Azure AD 进行的查询中，可使用对象 ID 来标识对象。在 SAML 令牌中，对象 ID 出现在 objectidentifier 属性中。

    <Attribute Name="http://schemas.microsoft.com/identity/claims/objectidentifier">
    <AttributeValue>528b2ac2-aa9c-45e1-88d4-959b53bc7dd0<AttributeValue>

在 JWT 令牌中，对象 ID 出现在 oid 声明中。

    "oid":"528b2ac2-aa9c-45e1-88d4-959b53bc7dd0"

### 角色

角色声明提供表示 Azure AD 中使用者的应用程序角色分配的友好字符串，可用于强制实施基于角色的访问控制。可通过应用程序清单的“appRoles”属性基于每个应用程序定义应用程序角色。每个应用程序角色的“value”属性是角色声明中显示的值。角色声明中包含的角色表示直接和间接通过组成员身份授予使用者的所有应用程序角色。在 SAML 令牌中，角色声明出现在 roles 属性中。

    <Attribute Name="http://schemas.microsoft.com/ws/2008/06/identity/claims/role">
    <AttributeValue>Admin</AttributeValue>

在 JWT 令牌中，角色声明出现在 roles 声明中。

    “roles”: ["Admin", … ]

### 范围

令牌的作用域指示授予客户端应用程序的模拟权限。默认权限是 user_impersonation。受保护资源的所有者可在 Azure AD 中注册其他值。

在 JWT 令牌中，令牌的作用域在 scp 声明中指定。

    "scp": "user_impersonation"

### 使用者

令牌的使用者是令牌针对其验证信息的主体，如应用程序的用户。此值不可变且不能重新分配或重复使用，因此可以使用它来安全地执行授权检查，例如，当使用令牌访问资源时。因为使用者始终会在 Azure AD 颁发的令牌中存在，我们建议在通用授权系统中使用此值。

在 SAML 令牌中，令牌的使用者在 Subject 元素的 NameID 元素中指定。NameID 是使用者（可以是用户、应用程序或服务）的唯一且不可重复使用的标识符。

SubjectConfirmation 不是声明。它描述如何对令牌的使用者进行验证。“Bearer”指示通过其拥有的令牌确认的使用者。

    <Subject>
    <NameID>S40rgb3XjhFTv6EQTETkEzcgVmToHKRkZUIsJlmLdVc</NameID>
    <SubjectConfirmation Method="urn:oasis:names:tc:SAML:2.0:cm:bearer" />
    </Subject>

在 JWT 令牌中，使用者出现在 sub 声明中。

    "sub":"92d0312b-26b9-4887-a338-7b00fb3c5eab"

### 租户 ID
租户 ID 是一个不可变且不能重复使用的标识符，用于标识颁发令牌的目录租户。可以使用此值访问多租户应用程序中特定于租户的目录资源。例如，你可以在调用 Graph API 时使用此值标识租户。

在 SAML 令牌中，租户 ID 出现在 tenantid SAML 属性元素的声明中。

    <Attribute Name=”http://schemas.microsoft.com/identity/claims/tenantid”>
    <AttributeValue>cbb1a5ac-f33b-45fa-9bf5-f37db0fed422<AttributeValue>

在 JWT 令牌中，租户 ID 出现在 tid 声明中。

    "tid":"cbb1a5ac-f33b-45fa-9bf5-f37db0fed422"

### 令牌生存期
令牌生存期声明定义令牌有效的时间间隔。验证令牌的服务应验证当前日期是否在令牌生存期内。如果当前日期不在令牌生存期内，它应拒绝该令牌。考虑到 Azure AD 与该服务之间可能存在时钟时间差异（“时间偏差”），该服务可能会在令牌生存期期限之外提供最多五分钟的宽限期。

在 SAML 令牌中，令牌生存期声明在 Conditions 元素中使用 NotBefore 和NotOnOrAfter 属性定义。

    <Conditions
    NotBefore="2013-03-18T21:32:51.261Z"
    NotOnOrAfter="2013-03-18T22:32:51.261Z"
    >

在 JWT 令牌中，令牌生存期由 nbf（不在此之前）和 exp（到期时间）声明定义。这些声明的值用自协调世界时 (UTC) 的 1970-01-010:0:0Z 以来经过的秒数来表示。有关详细信息，请参阅 RFC 3339。

    "nbf":1363289634,
    "exp":1363293234

### 用户主体名称
用户主体名称声明存储用户主体的用户名。

在 JWT 令牌中，用户主体名称出现在 upn 声明中。

    "upn": frankm@contoso.com

### 版本
版本声明存储令牌的版本号。在 JWT 令牌中，用户主体名称出现在 ver 声明中。

    "ver": "1.0"

## 示例令牌

本部分显示 Azure AD 返回的 SAML 和 JWT 令牌的示例。这些示例让你查看上下文中的声明。SAML 令牌

这是典型 SAML 令牌的一个示例。

	<?xml version="1.0" encoding="UTF-8"?>
	<t:RequestSecurityTokenResponse xmlns:t="http://schemas.xmlsoap.org/ws/2005/02/trust">
	  <t:Lifetime>
		<wsu:Created xmlns:wsu="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-utility-1.0.xsd">2014-12-24T05:15:47.060Z</wsu:Created>
		<wsu:Expires xmlns:wsu="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-utility-1.0.xsd">2014-12-24T06:15:47.060Z</wsu:Expires>
	  </t:Lifetime>
	  <wsp:AppliesTo xmlns:wsp="http://schemas.xmlsoap.org/ws/2004/09/policy">
		<EndpointReference xmlns="http://www.w3.org/2005/08/addressing">
		  <Address>https://contoso.partner.onmschina.cn/MyWebApp</Address>
		</EndpointReference>
	  </wsp:AppliesTo>
	  <t:RequestedSecurityToken>
		<Assertion xmlns="urn:oasis:names:tc:SAML:2.0:assertion" ID="_3ef08993-846b-41de-99df-b7f3ff77671b" IssueInstant="2014-12-24T05:20:47.060Z" Version="2.0">
		  <Issuer>https://sts.chinacloudapi.cn/b9411234-09af-49c2-b0c3-653adc1f376e/</Issuer>
		  <ds:Signature xmlns:ds="http://www.w3.org/2000/09/xmldsig#">
			<ds:SignedInfo>
			  <ds:CanonicalizationMethod Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#" />
			  <ds:SignatureMethod Algorithm="http://www.w3.org/2001/04/xmldsig-more#rsa-sha256" />
			  <ds:Reference URI="#_3ef08993-846b-41de-99df-b7f3ff77671b">
				<ds:Transforms>
				  <ds:Transform Algorithm="http://www.w3.org/2000/09/xmldsig#enveloped-signature" />
				  <ds:Transform Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#" />
				</ds:Transforms>
				<ds:DigestMethod Algorithm="http://www.w3.org/2001/04/xmlenc#sha256" />
				<ds:DigestValue>cV1J580U1pD24hEyGuAxrbtgROVyghCqI32UkER/nDY=</ds:DigestValue>
			  </ds:Reference>
			</ds:SignedInfo>
			<ds:SignatureValue>j+zPf6mti8Rq4Kyw2NU2nnu0pbJU1z5bR/zDaKaO7FCTdmjUzAvIVfF8pspVR6CbzcYM3HOAmLhuWmBkAAk6qQUBmKsw+XlmF/pB/ivJFdgZSLrtlBs1P/WBV3t04x6fRW4FcIDzh8KhctzJZfS5wGCfYw95er7WJxJi0nU41d7j5HRDidBoXgP755jQu2ZER7wOYZr6ff+ha+/Aj3UMw+8ZtC+WCJC3yyENHDAnp2RfgdElJal68enn668fk8pBDjKDGzbNBO6qBgFPaBT65YvE/tkEmrUxdWkmUKv3y7JWzUYNMD9oUlut93UTyTAIGOs5fvP9ZfK2vNeMVJW7Xg==</ds:SignatureValue>
			<KeyInfo xmlns="http://www.w3.org/2000/09/xmldsig#">
			  <X509Data>
				<X509Certificate>MIIDPjCCAabcAwIBAgIQsRiM0jheFZhKk49YD0SK1TAJBgUrDgMCHQUAMC0xKzApBgNVBAMTImFjY291bnRzLmFjY2Vzc2NvbnRyb2wud2luZG93cy5uZXQwHhcNMTQwMTAxMDcwMDAwWhcNMTYwMTAxMDcwMDAwWjAtMSswKQYDVQQDEyJhY2NvdW50cy5hY2Nlc3Njb250cm9sLndpbmRvd3MubmV0MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAkSCWg6q9iYxvJE2NIhSyOiKvqoWCO2GFipgH0sTSAs5FalHQosk9ZNTztX0ywS/AHsBeQPqYygfYVJL6/EgzVuwRk5txr9e3n1uml94fLyq/AXbwo9yAduf4dCHTP8CWR1dnDR+Qnz/4PYlWVEuuHHONOw/blbfdMjhY+C/BYM2E3pRxbohBb3x//CfueV7ddz2LYiH3wjz0QS/7kjPiNCsXcNyKQEOTkbHFi3mu0u13SQwNddhcynd/GTgWN8A+6SN1r4hzpjFKFLbZnBt77ACSiYx+IHK4Mp+NaVEi5wQtSsjQtI++XsokxRDqYLwus1I1SihgbV/STTg5enufuwIDAQABo2IwYDBeBgNVHQEEVzBVgBDLebM6bK3BjWGqIBrBNFeNoS8wLTErMCkGA1UEAxMiYWNjb3VudHMuYWNjZXNzY29udHJvbC53aW5kb3dzLm5ldIIQsRiM0jheFZhKk49YD0SK1TAJBgUrDgMCHQUAA4IBAQCJ4JApryF77EKC4zF5bUaBLQHQ1PNtA1uMDbdNVGKCmSp8M65b8h0NwlIjGGGy/unK8P6jWFdm5IlZ0YPTOgzcRZguXDPj7ajyvlVEQ2K2ICvTYiRQqrOhEhZMSSZsTKXFVwNfW6ADDkN3bvVOVbtpty+nBY5UqnI7xbcoHLZ4wYD251uj5+lo13YLnsVrmQ16NCBYq2nQFNPuNJw6t3XUbwBHXpF46aLT1/eGf/7Xx6iy8yPJX4DyrpFTutDz882RWofGEO5t4Cw+zZg70dJ/hH/ODYRMorfXEW+8uKmXMKmX2wyxMKvfiPbTy5LmAU8Jvjs2tLg4rOBcXWLAIarZ</X509Certificate>
			  </X509Data>
			</KeyInfo>
		  </ds:Signature>
		  <Subject>
			<NameID Format="urn:oasis:names:tc:SAML:2.0:nameid-format:persistent">m_H3naDei2LNxUmEcWd0BZlNi_jVET1pMLR6iQSuYmo</NameID>
			<SubjectConfirmation Method="urn:oasis:names:tc:SAML:2.0:cm:bearer" />
		  </Subject>
		  <Conditions NotBefore="2014-12-24T05:15:47.060Z" NotOnOrAfter="2014-12-24T06:15:47.060Z">
			<AudienceRestriction>
			  <Audience>https://contoso.partner.onmschina.cn/MyWebApp</Audience>
			</AudienceRestriction>
		  </Conditions>
		  <AttributeStatement>
			<Attribute Name="http://schemas.microsoft.com/identity/claims/objectidentifier">
			  <AttributeValue>a1addde8-e4f9-4571-ad93-3059e3750d23</AttributeValue>
			</Attribute>
			<Attribute Name="http://schemas.microsoft.com/identity/claims/tenantid">
			  <AttributeValue>b9411234-09af-49c2-b0c3-653adc1f376e</AttributeValue>
			</Attribute>
			<Attribute Name="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name">
			  <AttributeValue>sample.admin@contoso.partner.onmschina.cn</AttributeValue>
			</Attribute>
			<Attribute Name="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname">
			  <AttributeValue>Admin</AttributeValue>
			</Attribute>
			<Attribute Name="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname">
			  <AttributeValue>Sample</AttributeValue>
			</Attribute>
			<Attribute Name="http://schemas.microsoft.com/ws/2008/06/identity/claims/groups">
			  <AttributeValue>5581e43f-6096-41d4-8ffa-04e560bab39d</AttributeValue>
			  <AttributeValue>07dd8a89-bf6d-4e81-8844-230b77145381</AttributeValue>
			  <AttributeValue>0e129f4g-6b0a-4944-982d-f776000632af</AttributeValue>
			  <AttributeValue>3ee07328-52ef-4739-a89b-109708c22fb5</AttributeValue>
			  <AttributeValue>329k14b3-1851-4b94-947f-9a4dacb595f4</AttributeValue>
			  <AttributeValue>6e32c650-9b0a-4491-b429-6c60d2ca9a42</AttributeValue>
			  <AttributeValue>f3a169a7-9a58-4e8f-9d47-b70029v07424</AttributeValue>
			  <AttributeValue>8e2c86b2-b1ad-476d-9574-544d155aa6ff</AttributeValue>
			  <AttributeValue>1bf80264-ff24-4866-b22c-6212e5b9a847</AttributeValue>
			  <AttributeValue>4075f9c3-072d-4c32-b542-03e6bc678f3e</AttributeValue>
			  <AttributeValue>76f80527-f2cd-46f4-8c52-8jvd8bc749b1</AttributeValue>
			  <AttributeValue>0ba31460-44d0-42b5-b90c-47b3fcc48e35</AttributeValue>
			  <AttributeValue>edd41703-8652-4948-94a7-2d917bba7667</AttributeValue>
			</Attribute>
			<Attribute Name="http://schemas.microsoft.com/identity/claims/identityprovider">
			  <AttributeValue>https://sts.chinacloudapi.cn/b9411234-09af-49c2-b0c3-653adc1f376e/</AttributeValue>
			</Attribute>
		  </AttributeStatement>
		  <AuthnStatement AuthnInstant="2014-12-23T18:51:11.000Z">
			<AuthnContext>
			  <AuthnContextClassRef>urn:oasis:names:tc:SAML:2.0:ac:classes:Password</AuthnContextClassRef>
			</AuthnContext>
		  </AuthnStatement>
		</Assertion>
	  </t:RequestedSecurityToken>
	  <t:RequestedAttachedReference>
		<SecurityTokenReference xmlns="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-secext-1.0.xsd" xmlns:d3p1="http://docs.oasis-open.org/wss/oasis-wss-wssecurity-secext-1.1.xsd" d3p1:TokenType="http://docs.oasis-open.org/wss/oasis-wss-saml-token-profile-1.1#SAMLV2.0">
		  <KeyIdentifier ValueType="http://docs.oasis-open.org/wss/oasis-wss-saml-token-profile-1.1#SAMLID">_3ef08993-846b-41de-99df-b7f3ff77671b</KeyIdentifier>
		</SecurityTokenReference>
	  </t:RequestedAttachedReference>
	  <t:RequestedUnattachedReference>
		<SecurityTokenReference xmlns="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-secext-1.0.xsd" xmlns:d3p1="http://docs.oasis-open.org/wss/oasis-wss-wssecurity-secext-1.1.xsd" d3p1:TokenType="http://docs.oasis-open.org/wss/oasis-wss-saml-token-profile-1.1#SAMLV2.0">
		  <KeyIdentifier ValueType="http://docs.oasis-open.org/wss/oasis-wss-saml-token-profile-1.1#SAMLID">_3ef08993-846b-41de-99df-b7f3ff77671b</KeyIdentifier>
		</SecurityTokenReference>
	  </t:RequestedUnattachedReference>
	  <t:TokenType>http://docs.oasis-open.org/wss/oasis-wss-saml-token-profile-1.1#SAMLV2.0</t:TokenType>
	  <t:RequestType>http://schemas.xmlsoap.org/ws/2005/02/trust/Issue</t:RequestType>
	  <t:KeyType>http://schemas.xmlsoap.org/ws/2005/05/identity/NoProofKey</t:KeyType>
	</t:RequestSecurityTokenResponse>

### JWT 令牌 - 用户模拟

这是用户模拟 Web 流程中使用的典型 JSON Web 令牌 (JWT) 的一个示例。除了声明外，该令牌还在 **ver** 中提供了版本号，并在 **appidacr** 中提供了身份验证上下文类引用，该项指示如何对客户端进行身份验证。对于公共客户端，该值为 0。如果使用客户端 ID 或客户端机密，则该值为 1。

    {
     typ: "JWT",
     alg: "RS256",
     x5t: "kriMPdmBvx68skT8-mPAB3BseeA"
    }.
    {
     aud: "https://contoso.partner.onmschina.cn/scratchservice",
     iss: "https://sts.chinacloudapi.cn/b9411234-09af-49c2-b0c3-653adc1f376e/",
     iat: 1416968588,
     nbf: 1416968588,
     exp: 1416972488,
     ver: "1.0",
     tid: "b9411234-09af-49c2-b0c3-653adc1f376e",
     amr: [
      "pwd"
     ],
     roles: [
      "Admin"
     ],
     oid: "6526e123-0ff9-4fec-ae64-a8d5a77cf287",
     upn: "sample.user@contoso.partner.onmschina.cn",
     unique_name: "sample.user@contoso.partner.onmschina.cn",
     sub: "yf8C5e_VRkR1egGxJSDt5_olDFay6L5ilBA81hZhQEI",
     family_name: "User",
     given_name: "Sample",
     groups: [
      "0e129f6b-6b0a-4944-982d-f776000632af",
      "323b13b3-1851-4b94-947f-9a4dacb595f4",
      "6e32c250-9b0a-4491-b429-6c60d2ca9a42",
      "f3a161a7-9a58-4e8f-9d47-b70022a07424",
      "8d4c81b2-b1ad-476d-9574-544d155aa6ff",
      "1bf80164-ff24-4866-b19c-6212e5b9a847",
      "76f80127-f2cd-46f4-8c52-8edd8bc749b1",
      "0ba27160-44d0-42b5-b90c-47b3fcc48e35"
     ],
     appid: "b075ddef-0efa-123b-997b-de1337c29185",
     appidacr: "1",
     scp: "user_impersonation",
     acr: "1"
    }.

##另请参阅

[Azure Active Directory 身份验证协议](/documentation/articles/active-directory-authentication-protocols/)

<!---HONumber=Mooncake_1221_2015-->