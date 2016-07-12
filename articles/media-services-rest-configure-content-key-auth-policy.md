<properties 
	pageTitle="使用媒体服务 REST API 配置内容密钥授权策略" 
	description="了解如何使用媒体服务 REST API 配置内容密钥的授权策略。" 
	services="media-services" 
	documentationCenter="" 
	authors="Juliako" 
	manager="dwrede" 
	editor=""/>

<tags
	ms.service="media-services"
 	ms.date="04/07/2016"  
	wacn.date="05/16/2016"/>


#动态加密：配置内容密钥授权策略 
[AZURE.INCLUDE [media-services-selector-content-key-auth-policy](../includes/media-services-selector-content-key-auth-policy.md)]
 

##概述

借助 Azure 媒体服务，你可以传送使用高级加密标准 (AES)（使用 128 位加密密钥）和 PlayReady 或 Widevine DRM 加密的内容。媒体服务还提供了用于向已授权客户端传送密钥和 PlayReady/Widevine 许可证的服务。

如果你需要媒体服务来加密资产，则需要将加密密钥（**CommonEncryption** 或 **EnvelopeEncryption**）与资产相关联（如[此处](/documentation/articles/media-services-rest-create-contentkey/)所述），并且配置密钥的授权策略（如本文所述）。


当播放器请求流时，媒体服务将使用指定的密钥通过 AES 或 PlayReady 加密来动态加密你的内容。为了解密流，播放器将从密钥传送服务请求密钥。为了确定用户是否被授权获取密钥，服务将评估你为密钥指定的授权策略。

媒体服务支持通过多种方式对发出密钥请求的用户进行身份验证。内容密钥授权策略可能受到一种或多种授权限制：**开放**或**令牌**限制。令牌限制策略必须附带由安全令牌服务 (STS) 颁发的令牌。媒体服务支持采用**简单 Web 令牌** ([SWT](https://msdn.microsoft.com/zh-cn/library/gg185950.aspx#BKMK_2)) 格式和 **JSON Web 令牌**(JWT) 格式的令牌。

媒体服务不提供安全令牌服务。你可以创建自定义 STS 或利用 Azure ACS 来颁发令牌。必须将 STS 配置为创建令牌，该令牌使用指定密钥以及你在令牌限制配置中指定的颁发声明进行签名（如本文所述）。如果令牌有效，而且令牌中的声明与为内容密钥配置的声明相匹配，则媒体服务密钥传送服务会将加密密钥返回到客户端。

有关详细信息，请参阅

[JWT 令牌身份验证](http://www.gtrifonov.com/2015/01/03/jwt-token-authentication-in-azure-media-services-and-dynamic-encryption/)

[将基于 Azure 媒体服务 OWIN MVC 的应用与 Azure Active Directory 相集成，并基于 JWT 声明限制内容密钥传送](http://www.gtrifonov.com/2015/01/24/mvc-owin-azure-media-services-ad-integration/)。

[使用 Azure ACS 颁发令牌](http://mingfeiy.com/acs-with-key-services)。

###请注意以下事项：

- 为了能够使用动态打包和动态加密，必须确保至少有一个流式处理保留单位。有关详细信息，请参阅[如何缩放媒体服务](/documentation/articles/media-services-manage-origins/#scale_streaming_endpoints)。 
- 你的资产必须包含一组自适应比特率 MP4 或自适应比特率平滑流文件。有关详细信息，请参阅[对资产进行编码](/documentation/articles/media-services-encode-asset/)。  
- 使用 **AssetCreationOptions.StorageEncrypted** 选项上载资产并对其进行编码。
- 如果你打算创建需要相同策略配置的多个内容密钥，我们强烈建议你创建单个授权策略，并将其重复用于多个内容密钥。
- 密钥传送服务将 ContentKeyAuthorizationPolicy 及其相关对象（策略选项和限制）缓存 15 分钟。如果你创建 ContentKeyAuthorizationPolicy 并指定使用“令牌”限制，然后对其进行测试，再将策略更新为“开放”限制，则现有策略切换到“开放”版本的策略需要大约 15 分钟。
- 如果你添加或更新资产的传送策略，则必须删除现有定位符（如果有）并创建新定位符。
- 目前，无法加密 HDS 流格式或渐进式下载。


##AES-128 动态加密 

>[AZURE.NOTE] 使用媒体服务 REST API 时，需注意以下事项：
>
>访问媒体服务中的实体时，必须在 HTTP 请求中设置特定标头字段和值。有关详细信息，请参阅[媒体服务 REST API 开发的设置](/documentation/articles/media-services-rest-how-to-use/)。

>在成功连接到 https://media.chinacloudapi.cn 之后，你将接收到指定另一个媒体服务 URI 的 301 重定向。必须按[使用 REST API 连接到媒体服务](/documentation/articles/media-services-rest-connect_programmatically/)中所述对新的 URI 执行后续调用。


###开放限制

开放限制意味着系统会将密钥传送到发出密钥请求的任何用户。此限制可能适用于测试用途。

以下示例创建开放授权策略，并将其添加到内容密钥。

####<a id="ContentKeyAuthorizationPolicies"></a>创建 ContentKeyAuthorizationPolicies

请求：
		
	POST https://wamsbayclus001rest-hs.chinacloudapp.cn/api/ContentKeyAuthorizationPolicies HTTP/1.1
	Content-Type: application/json
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=juliakoams1&urn%3aSubscriptionId=bbbef702-e769-477b-9f16-bc4d3aa97387&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1423578086&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=lZlyQ2%2bvH73qtJsb42%2fH3xF7r7EvQFR3UXyezuDENFU%3d
	x-ms-version: 2.11
	x-ms-client-request-id: d732dbfa-54fc-474c-99d6-9b46a006f389
	Host: wamsbayclus001rest-hs.chinacloudapp.cn
	Content-Length: 36
	
	{"Name":"Open Authorization Policy"}
	
响应：
	
	HTTP/1.1 201 Created
	Cache-Control: no-cache
	Content-Length: 211
	Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
	Location: https://wamsbayclus001rest-hs.chinacloudapp.cn/api/ContentKeyAuthorizationPolicies('nb%3Ackpid%3AUUID%3Adb4593da-f4d1-4cc5-a92a-d20eacbabee4')
	Server: Microsoft-IIS/8.5
	x-ms-client-request-id: d732dbfa-54fc-474c-99d6-9b46a006f389
	request-id: aabfa731-e884-4bf3-8314-492b04747ac4
	x-ms-request-id: aabfa731-e884-4bf3-8314-492b04747ac4
	X-Content-Type-Options: nosniff
	DataServiceVersion: 3.0;
	X-Powered-By: ASP.NET
	Strict-Transport-Security: max-age=31536000; includeSubDomains
	Date: Tue, 10 Feb 2015 08:25:56 GMT
	
	{"odata.metadata":"https://wamsbayclus001rest-hs.chinacloudapp.cn/api/$metadata#ContentKeyAuthorizationPolicies/@Element","Id":"nb:ckpid:UUID:db4593da-f4d1-4cc5-a92a-d20eacbabee4","Name":"Open Authorization Policy"}

####<a id="ContentKeyAuthorizationPolicyOptions"></a>创建 ContentKeyAuthorizationPolicyOptions
	
请求：

	POST https://wamsbayclus001rest-hs.chinacloudapp.cn/api/ContentKeyAuthorizationPolicyOptions HTTP/1.1
	Content-Type: application/json
	DataServiceVersion: 3.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=juliakoams1&urn%3aSubscriptionId=bbbef702-e769-477b-9f16-bc4d3aa97387&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1423580006&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=Ref3EsonGF7fUKCwGwGgiMnZitzIzsDOvvMTeVrVVPg%3d
	x-ms-version: 2.11
	x-ms-client-request-id: d225e357-e60e-4f42-add8-9d93aba1409a
	Host: wamsbayclus001rest-hs.chinacloudapp.cn
	Content-Length: 168
	
	{"Name":"policy","KeyDeliveryType":2,"KeyDeliveryConfiguration":"","Restrictions":[{"Name":"HLS Open Authorization Policy","KeyRestrictionType":0,"Requirements":null}]}

响应：
	
	HTTP/1.1 201 Created
	Cache-Control: no-cache
	Content-Length: 349
	Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
	Location: https://wamsbayclus001rest-hs.chinacloudapp.cn/api/ContentKeyAuthorizationPolicyOptions('nb%3Ackpoid%3AUUID%3A57829b17-1101-4797-919b-f816f4a007b7')
	Server: Microsoft-IIS/8.5
	x-ms-client-request-id: d225e357-e60e-4f42-add8-9d93aba1409a
	request-id: 81bcad37-295b-431f-972f-b23f2e4172c9
	x-ms-request-id: 81bcad37-295b-431f-972f-b23f2e4172c9
	X-Content-Type-Options: nosniff
	DataServiceVersion: 3.0;
	X-Powered-By: ASP.NET
	Strict-Transport-Security: max-age=31536000; includeSubDomains
	Date: Tue, 10 Feb 2015 08:56:40 GMT
	
	{"odata.metadata":"https://wamsbayclus001rest-hs.chinacloudapp.cn/api/$metadata#ContentKeyAuthorizationPolicyOptions/@Element","Id":"nb:ckpoid:UUID:57829b17-1101-4797-919b-f816f4a007b7","Name":"policy","KeyDeliveryType":2,"KeyDeliveryConfiguration":"","Restrictions":[{"Name":"HLS Open Authorization Policy","KeyRestrictionType":0,"Requirements":null}]}

####<a id="LinkContentKeyAuthorizationPoliciesWithOptions"></a>将 ContentKeyAuthorizationPolicies 与 Options 相链接

请求：
	
	POST https://wamsbayclus001rest-hs.chinacloudapp.cn/api/ContentKeyAuthorizationPolicies('nb%3Ackpid%3AUUID%3A0baa438b-8ac2-4c40-a53c-4d4722b78715')/$links/Options HTTP/1.1
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Content-Type: application/json
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=juliakoams1&urn%3aSubscriptionId=zbbef702-2233-477b-9f16-bc4d3aa97387&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1423580006&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=Ref3EsonGF7fUKCwGwGgiMnZitzIzsDOvvMTeVrVVPg%3d
	x-ms-version: 2.11
	x-ms-client-request-id: 9847f705-f2ca-4e95-a478-8f823dbbaa29
	Host: wamsbayclus001rest-hs.chinacloudapp.cn
	Content-Length: 154
	
	{"uri":"https://wamsbayclus001rest-hs.chinacloudapp.cn/api/ContentKeyAuthorizationPolicyOptions('nb%3Ackpoid%3AUUID%3A57829b17-1101-4797-919b-f816f4a007b7')"}

响应：

	HTTP/1.1 204 No Content

####<a id="AddAuthorizationPolicyToKey"></a>将授权策略添加到内容密钥

请求：

	PUT https://wamsbayclus001rest-hs.chinacloudapp.cn/api/ContentKeys('nb%3Akid%3AUUID%3A2e6d36a7-a17c-4e9a-830d-eca23ad1a6f9') HTTP/1.1
	Content-Type: application/json
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=juliakoams1&urn%3aSubscriptionId=zbbef702-2233-477b-9f16-bc4d3aa97387&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1423581565&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=JiNSG3w6r2C0nIyfKvTZj1uPJGjuitD%2b0sbfZ%2b2JDZI%3d
	x-ms-version: 2.11
	x-ms-client-request-id: e613efff-cb6a-41b4-984a-f4f8fb6e76a4
	Host: wamsbayclus001rest-hs.chinacloudapp.cn
	Content-Length: 78
	
	{"AuthorizationPolicyId":"nb:ckpid:UUID:c06cebb8-c4f0-4d1a-ba00-3273fb2bc3ad"}

响应：

	HTTP/1.1 204 No Content

###令牌限制

本部分介绍如何创建内容密钥授权策略，以及如何将其与内容密钥相关联。授权策略描述了必须达到什么授权要求才能确定用户是否有权接收密钥（例如，“验证密钥”列表是否包含令牌签名时使用的密钥）。

若要配置令牌限制选项，你需要使用 XML 来描述令牌的授权要求。令牌限制配置 XML 必须符合以下 XML 架构。

####<a id="schema"></a>令牌限制架构
	
	<?xml version="1.0" encoding="utf-8"?>
	<xs:schema xmlns:tns="http://schemas.microsoft.com/Azure/MediaServices/KeyDelivery/TokenRestrictionTemplate/v1" elementFormDefault="qualified" targetNamespace="http://schemas.microsoft.com/Azure/MediaServices/KeyDelivery/TokenRestrictionTemplate/v1" xmlns:xs="http://www.w3.org/2001/XMLSchema">
	  <xs:complexType name="TokenClaim">
	    <xs:sequence>
	      <xs:element name="ClaimType" nillable="true" type="xs:string" />
	      <xs:element minOccurs="0" name="ClaimValue" nillable="true" type="xs:string" />
	    </xs:sequence>
	  </xs:complexType>
	  <xs:element name="TokenClaim" nillable="true" type="tns:TokenClaim" />
	  <xs:complexType name="TokenRestrictionTemplate">
	    <xs:sequence>
	      <xs:element minOccurs="0" name="AlternateVerificationKeys" nillable="true" type="tns:ArrayOfTokenVerificationKey" />
	      <xs:element name="Audience" nillable="true" type="xs:anyURI" />
	      <xs:element name="Issuer" nillable="true" type="xs:anyURI" />
	      <xs:element name="PrimaryVerificationKey" nillable="true" type="tns:TokenVerificationKey" />
	      <xs:element minOccurs="0" name="RequiredClaims" nillable="true" type="tns:ArrayOfTokenClaim" />
	    </xs:sequence>
	  </xs:complexType>
	  <xs:element name="TokenRestrictionTemplate" nillable="true" type="tns:TokenRestrictionTemplate" />
	  <xs:complexType name="ArrayOfTokenVerificationKey">
	    <xs:sequence>
	      <xs:element minOccurs="0" maxOccurs="unbounded" name="TokenVerificationKey" nillable="true" type="tns:TokenVerificationKey" />
	    </xs:sequence>
	  </xs:complexType>
	  <xs:element name="ArrayOfTokenVerificationKey" nillable="true" type="tns:ArrayOfTokenVerificationKey" />
	  <xs:complexType name="TokenVerificationKey">
	    <xs:sequence />
	  </xs:complexType>
	  <xs:element name="TokenVerificationKey" nillable="true" type="tns:TokenVerificationKey" />
	  <xs:complexType name="ArrayOfTokenClaim">
	    <xs:sequence>
	      <xs:element minOccurs="0" maxOccurs="unbounded" name="TokenClaim" nillable="true" type="tns:TokenClaim" />
	    </xs:sequence>
	  </xs:complexType>
	  <xs:element name="ArrayOfTokenClaim" nillable="true" type="tns:ArrayOfTokenClaim" />
	  <xs:complexType name="SymmetricVerificationKey">
	    <xs:complexContent mixed="false">
	      <xs:extension base="tns:TokenVerificationKey">
	        <xs:sequence>
	          <xs:element name="KeyValue" nillable="true" type="xs:base64Binary" />
	        </xs:sequence>
	      </xs:extension>
	    </xs:complexContent>
	  </xs:complexType>
	  <xs:element name="SymmetricVerificationKey" nillable="true" type="tns:SymmetricVerificationKey" />
	</xs:schema>

在配置**令牌**限制策略时，必须指定主**验证密钥**、**颁发者**和**受众**参数。**主验证密钥**包含用来为令牌签名的的密钥，**颁发者**是颁发令牌的安全令牌服务。**受众**（有时称为**范围**）描述该令牌的意图，或者令牌授权访问的资源。媒体服务密钥交付服务将验证令牌中的这些值是否与模板中的值匹配。

以下示例创建包含令牌限制的授权策略。在此示例中，客户端必须出示令牌，其中包含：签名密钥 (VerificationKey)、令牌颁发者和必需的声明。
	
###创建 ContentKeyAuthorizationPolicies

根据[此处](#ContentKeyAuthorizationPolicies)所示创建“令牌限制策略”。


###创建 ContentKeyAuthorizationPolicyOptions

请求：
	
	POST https://wamsbayclus001rest-hs.chinacloudapp.cn/api/ContentKeyAuthorizationPolicyOptions HTTP/1.1
	Content-Type: application/json
	DataServiceVersion: 3.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=juliakoams1&urn%3aSubscriptionId=bbbef702-e769-477b-9f16-bc4d3aa97387&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1423580720&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=5LsNu%2b0D4eD3UOP3BviTLDkUjaErdUx0ekJ8402xidQ%3d
	x-ms-version: 2.11
	x-ms-client-request-id: 2643d836-bfe7-438e-9ba2-bc6ff28e4a53
	Host: wamsbayclus001rest-hs.chinacloudapp.cn
	Content-Length: 1079
	
	{"Name":"Token option for HLS","KeyDeliveryType":2,"KeyDeliveryConfiguration":null,"Restrictions":[{"Name":"Token Authorization Policy","KeyRestrictionType":1,"Requirements":"<TokenRestrictionTemplate xmlns:i="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/Azure/MediaServices/KeyDelivery/TokenRestrictionTemplate/v1"><AlternateVerificationKeys><TokenVerificationKey i:type="SymmetricVerificationKey"><KeyValue>BklyAFiPTQsuJNKriQJBZHYaKM2CkCTDQX2bw9sMYuvEC9sjW0W7GUIBygQL/+POEeUqCYPnmEU2g0o1GW2Oqg==</KeyValue></TokenVerificationKey></AlternateVerificationKeys><Audience>urn:test</Audience><Issuer>http://testacs.com/</Issuer><PrimaryVerificationKey i:type="SymmetricVerificationKey"><KeyValue>E5BUHiN4vBdzUzdP0IWaHFMMU3D1uRZgF16TOhSfwwHGSw+Kbf0XqsHzEIYk11M372viB9vbiacsdcQksA0ftw==</KeyValue></PrimaryVerificationKey><RequiredClaims><TokenClaim><ClaimType>urn:microsoft:azure:mediaservices:contentkeyidentifier</ClaimType><ClaimValue i:nil="true" /></TokenClaim></RequiredClaims><TokenType>SWT</TokenType></TokenRestrictionTemplate>"}]}

响应：
	
	HTTP/1.1 201 Created
	Cache-Control: no-cache
	Content-Length: 1260
	Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
	Location: https://wamsbayclus001rest-hs.chinacloudapp.cn/api/ContentKeyAuthorizationPolicyOptions('nb%3Ackpoid%3AUUID%3Ae1ef6145-46e8-4ee6-9756-b1cf96328c23')
	Server: Microsoft-IIS/8.5
	x-ms-client-request-id: 2643d836-bfe7-438e-9ba2-bc6ff28e4a53
	request-id: 2310b716-aeaa-421e-913e-3ce2f6f685ca
	x-ms-request-id: 2310b716-aeaa-421e-913e-3ce2f6f685ca
	X-Content-Type-Options: nosniff
	DataServiceVersion: 3.0;
	X-Powered-By: ASP.NET
	Strict-Transport-Security: max-age=31536000; includeSubDomains
	Date: Tue, 10 Feb 2015 09:10:37 GMT
	
	{"odata.metadata":"https://wamsbayclus001rest-hs.chinacloudapp.cn/api/$metadata#ContentKeyAuthorizationPolicyOptions/@Element","Id":"nb:ckpoid:UUID:e1ef6145-46e8-4ee6-9756-b1cf96328c23","Name":"Token option for HLS","KeyDeliveryType":2,"KeyDeliveryConfiguration":null,"Restrictions":[{"Name":"Token Authorization Policy","KeyRestrictionType":1,"Requirements":"<TokenRestrictionTemplate xmlns:i="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/Azure/MediaServices/KeyDelivery/TokenRestrictionTemplate/v1"><AlternateVerificationKeys><TokenVerificationKey i:type="SymmetricVerificationKey"><KeyValue>BklyAFiPTQsuJNKriQJBZHYaKM2CkCTDQX2bw9sMYuvEC9sjW0W7GUIBygQL/+POEeUqCYPnmEU2g0o1GW2Oqg==</KeyValue></TokenVerificationKey></AlternateVerificationKeys><Audience>urn:test</Audience><Issuer>http://testacs.com/</Issuer><PrimaryVerificationKey i:type="SymmetricVerificationKey"><KeyValue>E5BUHiN4vBdzUzdP0IWaHFMMU3D1uRZgF16TOhSfwwHGSw+Kbf0XqsHzEIYk11M372viB9vbiacsdcQksA0ftw==</KeyValue></PrimaryVerificationKey><RequiredClaims><TokenClaim><ClaimType>urn:microsoft:azure:mediaservices:contentkeyidentifier</ClaimType><ClaimValue i:nil="true" /></TokenClaim></RequiredClaims><TokenType>SWT</TokenType></TokenRestrictionTemplate>"}]}
	
####将 ContentKeyAuthorizationPolicies 与 Options 相链接

根据[此处](#ContentKeyAuthorizationPolicies)所示将 ContentKeyAuthorizationPolicies 与 Options 相链接。

####将授权策略添加到内容密钥

根据[此处](#AddAuthorizationPolicyToKey)所示将 AuthorizationPolicy 添加到 ContentKey。


##PlayReady 动态加密 

媒体服务允许你配置相应的权限和限制，以便在用户尝试播放受保护的内容时，PlayReady DRM 运行时会强制实施这些权限和限制。

使用 PlayReady 保护你的内容时，需要在授权策略中指定的项目之一是用于定义 [PlayReady 许可证模板](/documentation/articles/media-services-playready-license-template-overview/)的 XML 字符串。

###开放限制
	
开放限制意味着系统会将密钥传送到发出密钥请求的任何用户。此限制可能适用于测试用途。

以下示例创建开放授权策略，并将其添加到内容密钥。
	
####<a id="ContentKeyAuthorizationPolicies2"></a>创建 ContentKeyAuthorizationPolicies

请求：

	POST https://wamsbayclus001rest-hs.chinacloudapp.cn/api/ContentKeyAuthorizationPolicies HTTP/1.1
	Content-Type: application/json
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=juliakoams1&urn%3aSubscriptionId=bbef702-2233-477b-9f16-bc4d3aa97387&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1423581565&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=JiNSG3w6r2C0nIyfKvTZj1uPJGjuitD%2b0sbfZ%2b2JDZI%3d
	x-ms-version: 2.11
	x-ms-client-request-id: 9e7fa407-f84e-43aa-8f05-9790b46e279b
	Host: wamsbayclus001rest-hs.chinacloudapp.cn
	Content-Length: 58
	
	{"Name":"Deliver Common Content Key"}
	
响应：
	
	HTTP/1.1 201 Created
	Cache-Control: no-cache
	Content-Length: 233
	Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
	Location: https://wamsbayclus001rest-hs.chinacloudapp.cn/api/ContentKeyAuthorizationPolicies('nb%3Ackpid%3AUUID%3Acc3c64a8-e2fc-4e09-bf60-ac954251a387')
	Server: Microsoft-IIS/8.5
	x-ms-client-request-id: 9e7fa407-f84e-43aa-8f05-9790b46e279b
	request-id: b3d33c1b-a9cb-4120-ac0c-18f64846c147
	x-ms-request-id: b3d33c1b-a9cb-4120-ac0c-18f64846c147
	X-Content-Type-Options: nosniff
	DataServiceVersion: 3.0;
	X-Powered-By: ASP.NET
	Strict-Transport-Security: max-age=31536000; includeSubDomains
	Date: Tue, 10 Feb 2015 09:26:00 GMT
	
	{"odata.metadata":"https://wamsbayclus001rest-hs.chinacloudapp.cn/api/$metadata#ContentKeyAuthorizationPolicies/@Element","Id":"nb:ckpid:UUID:cc3c64a8-e2fc-4e09-bf60-ac954251a387","Name":"Deliver Common Content Key"}
	

#### 创建 ContentKeyAuthorizationPolicyOptions

请求：
	
	POST https://wamsbayclus001rest-hs.chinacloudapp.cn/api/ContentKeyAuthorizationPolicyOptions HTTP/1.1
	Content-Type: application/json
	DataServiceVersion: 3.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=juliakoams1&urn%3aSubscriptionId=zbbef702-2233-477b-9f16-bc4d3aa97387&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1423581565&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=JiNSG3w6r2C0nIyfKvTZj1uPJGjuitD%2b0sbfZ%2b2JDZI%3d
	x-ms-version: 2.11
	x-ms-client-request-id: f160ad25-b457-4bc6-8197-315604c5e585
	Host: wamsbayclus001rest-hs.chinacloudapp.cn
	Content-Length: 593
	
	{"Name":"","KeyDeliveryType":1,"KeyDeliveryConfiguration":"<PlayReadyLicenseResponseTemplate xmlns:i="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/Azure/MediaServices/KeyDelivery/PlayReadyTemplate/v1"><LicenseTemplates><PlayReadyLicenseTemplate><AllowTestDevices>false</AllowTestDevices><ContentKey i:type="ContentEncryptionKeyFromHeader" /><LicenseType>Nonpersistent</LicenseType><PlayRight /></PlayReadyLicenseTemplate></LicenseTemplates></PlayReadyLicenseResponseTemplate>","Restrictions":[{"Name":"Open","KeyRestrictionType":0,"Requirements":null}]}
	
响应：
	
	HTTP/1.1 201 Created
	Cache-Control: no-cache
	Content-Length: 774
	Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
	Location: https://wamsbayclus001rest-hs.chinacloudapp.cn/api/ContentKeyAuthorizationPolicyOptions('nb%3Ackpoid%3AUUID%3A1052308c-4df7-4fdb-8d21-4d2141fc2be0')
	Server: Microsoft-IIS/8.5
	x-ms-client-request-id: f160ad25-b457-4bc6-8197-315604c5e585
	request-id: 563f5a42-50a4-4c4a-add8-a833f8364231
	x-ms-request-id: 563f5a42-50a4-4c4a-add8-a833f8364231
	X-Content-Type-Options: nosniff
	DataServiceVersion: 3.0;
	X-Powered-By: ASP.NET
	Strict-Transport-Security: max-age=31536000; includeSubDomains
	Date: Tue, 10 Feb 2015 09:23:24 GMT
	
	{"odata.metadata":"https://wamsbayclus001rest-hs.chinacloudapp.cn/api/$metadata#ContentKeyAuthorizationPolicyOptions/@Element","Id":"nb:ckpoid:UUID:1052308c-4df7-4fdb-8d21-4d2141fc2be0","Name":"","KeyDeliveryType":1,"KeyDeliveryConfiguration":"<PlayReadyLicenseResponseTemplate xmlns:i="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/Azure/MediaServices/KeyDelivery/PlayReadyTemplate/v1"><LicenseTemplates><PlayReadyLicenseTemplate><AllowTestDevices>false</AllowTestDevices><ContentKey i:type="ContentEncryptionKeyFromHeader" /><LicenseType>Nonpersistent</LicenseType><PlayRight /></PlayReadyLicenseTemplate></LicenseTemplates></PlayReadyLicenseResponseTemplate>","Restrictions":[{"Name":"Open","KeyRestrictionType":0,"Requirements":null}]}

####将 ContentKeyAuthorizationPolicies 与 Options 相链接

根据[此处](#ContentKeyAuthorizationPolicies)所示将 ContentKeyAuthorizationPolicies 与 Options 相链接。

####将授权策略添加到内容密钥

根据[此处](#AddAuthorizationPolicyToKey)所示将 AuthorizationPolicy 添加到 ContentKey。


###令牌限制

若要配置令牌限制选项，你需要使用 XML 来描述令牌的授权要求。令牌限制配置 XML 必须符合[此](#schema)部分中所示的 XML 架构。
	
####创建 ContentKeyAuthorizationPolicies
	
根据[此处](#ContentKeyAuthorizationPolicies2)所示创建 ContentKeyAuthorizationPolicies。

####创建 ContentKeyAuthorizationPolicyOptions
	
请求：

	POST https://wamsbayclus001rest-hs.chinacloudapp.cn/api/ContentKeyAuthorizationPolicyOptions HTTP/1.1
	Content-Type: application/json
	DataServiceVersion: 3.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=juliakoams1&urn%3aSubscriptionId=zbbef702-2233-477b-9f16-bc4d3aa97387&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1423583561&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=5eZnkOsSv%2fLLEKmS%2bWObBlsNYyee8BQlp%2bUYbjugcJg%3d
	x-ms-version: 2.11
	x-ms-client-request-id: ab079b0e-2ba9-4cf1-b549-a97bfa6cd2d3
	Host: wamsbayclus001rest-hs.chinacloudapp.cn
	Content-Length: 1525
	
	{"Name":"Token option","KeyDeliveryType":1,"KeyDeliveryConfiguration":"<PlayReadyLicenseResponseTemplate xmlns:i="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/Azure/MediaServices/KeyDelivery/PlayReadyTemplate/v1"><LicenseTemplates><PlayReadyLicenseTemplate><AllowTestDevices>false</AllowTestDevices><ContentKey i:type="ContentEncryptionKeyFromHeader" /><LicenseType>Nonpersistent</LicenseType><PlayRight /></PlayReadyLicenseTemplate></LicenseTemplates></PlayReadyLicenseResponseTemplate>","Restrictions":[{"Name":"Token Authorization Policy","KeyRestrictionType":1,"Requirements":"<TokenRestrictionTemplate xmlns:i="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/Azure/MediaServices/KeyDelivery/TokenRestrictionTemplate/v1"><AlternateVerificationKeys><TokenVerificationKey i:type="SymmetricVerificationKey"><KeyValue>w52OyHVqXT8aaupGxuJ3NGt8M6opHDOtx132p4r6q4hLI6ffnLusgEGie1kedUewVoIe1tqDkVE6xsIV7O91KA==</KeyValue></TokenVerificationKey></AlternateVerificationKeys><Audience>urn:test</Audience><Issuer>http://testacs.com/</Issuer><PrimaryVerificationKey i:type="SymmetricVerificationKey"><KeyValue>dYwLKIEMBljLeY9VM7vWdlhps31Fbt0XXhqP5VyjQa33bJXleBtkzQ6dF5AtwI9gDcdM2dV2TvYNhCilBKjMCg==</KeyValue></PrimaryVerificationKey><RequiredClaims><TokenClaim><ClaimType>urn:microsoft:azure:mediaservices:contentkeyidentifier</ClaimType><ClaimValue i:nil="true" /></TokenClaim></RequiredClaims><TokenType>SWT</TokenType></TokenRestrictionTemplate>"}]}

响应：
	
	HTTP/1.1 201 Created
	Cache-Control: no-cache
	Content-Length: 1706
	Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
	Location: https://wamsbayclus001rest-hs.chinacloudapp.cn/api/ContentKeyAuthorizationPolicyOptions('nb%3Ackpoid%3AUUID%3Ae42bbeae-de42-4077-90e9-a844f297ef70')
	Server: Microsoft-IIS/8.5
	x-ms-client-request-id: ab079b0e-2ba9-4cf1-b549-a97bfa6cd2d3
	request-id: ccf8a4ba-731e-4124-8192-079592c251cc
	x-ms-request-id: ccf8a4ba-731e-4124-8192-079592c251cc
	X-Content-Type-Options: nosniff
	DataServiceVersion: 3.0;
	X-Powered-By: ASP.NET
	Strict-Transport-Security: max-age=31536000; includeSubDomains
	Date: Tue, 10 Feb 2015 09:58:47 GMT
	
	{"odata.metadata":"https://wamsbayclus001rest-hs.chinacloudapp.cn/api/$metadata#ContentKeyAuthorizationPolicyOptions/@Element","Id":"nb:ckpoid:UUID:e42bbeae-de42-4077-90e9-a844f297ef70","Name":"Token option","KeyDeliveryType":1,"KeyDeliveryConfiguration":"<PlayReadyLicenseResponseTemplate xmlns:i="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/Azure/MediaServices/KeyDelivery/PlayReadyTemplate/v1"><LicenseTemplates><PlayReadyLicenseTemplate><AllowTestDevices>false</AllowTestDevices><ContentKey i:type="ContentEncryptionKeyFromHeader" /><LicenseType>Nonpersistent</LicenseType><PlayRight /></PlayReadyLicenseTemplate></LicenseTemplates></PlayReadyLicenseResponseTemplate>","Restrictions":[{"Name":"Token Authorization Policy","KeyRestrictionType":1,"Requirements":"<TokenRestrictionTemplate xmlns:i="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/Azure/MediaServices/KeyDelivery/TokenRestrictionTemplate/v1"><AlternateVerificationKeys><TokenVerificationKey i:type="SymmetricVerificationKey"><KeyValue>w52OyHVqXT8aaupGxuJ3NGt8M6opHDOtx132p4r6q4hLI6ffnLusgEGie1kedUewVoIe1tqDkVE6xsIV7O91KA==</KeyValue></TokenVerificationKey></AlternateVerificationKeys><Audience>urn:test</Audience><Issuer>http://testacs.com/</Issuer><PrimaryVerificationKey i:type="SymmetricVerificationKey"><KeyValue>dYwLKIEMBljLeY9VM7vWdlhps31Fbt0XXhqP5VyjQa33bJXleBtkzQ6dF5AtwI9gDcdM2dV2TvYNhCilBKjMCg==</KeyValue></PrimaryVerificationKey><RequiredClaims><TokenClaim><ClaimType>urn:microsoft:azure:mediaservices:contentkeyidentifier</ClaimType><ClaimValue i:nil="true" /></TokenClaim></RequiredClaims><TokenType>SWT</TokenType></TokenRestrictionTemplate>"}]}

####将 ContentKeyAuthorizationPolicies 与 Options 相链接

根据[此处](#ContentKeyAuthorizationPolicies)所示将 ContentKeyAuthorizationPolicies 与 Options 相链接。

####将授权策略添加到内容密钥

根据[此处](#AddAuthorizationPolicyToKey)所示将 AuthorizationPolicy 添加到 ContentKey。


##<a id="types"></a>定义 ContentKeyAuthorizationPolicy 时使用的类型

###<a id="ContentKeyRestrictionType"></a>ContentKeyRestrictionType

    public enum ContentKeyRestrictionType
    {
        Open = 0,
        TokenRestricted = 1,
        IPRestricted = 2,
    }

###<a id="ContentKeyDeliveryType"></a>ContentKeyDeliveryType

    public enum ContentKeyDeliveryType
    {
        None = 0,
        PlayReadyLicense = 1,
        BaselineHttp = 2,
        Widevine = 3
    }





##后续步骤
在配置内容密钥的授权策略后，请转到[如何配置资产传送策略](/documentation/articles/media-services-rest-configure-asset-delivery-policy/)主题。

 
<!---HONumber=Mooncake_0509_2016-->