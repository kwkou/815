<properties 
	pageTitle="动态加密：使用 .NET 配置内容密钥授权策略" 
	description="了解如何配置内容密钥的授权策略。" 
	services="media-services" 
	documentationCenter="" 
	authors="juliako" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="media-services" 
	ms.date="09/16/2015"
	wacn.date="10/22/2015"/>



#动态加密：配置内容密钥授权策略 
[AZURE.INCLUDE [media-services-selector-content-key-auth-policy](../includes/media-services-selector-content-key-auth-policy)]


##概述

借助 Windows Azure 媒体服务，你可以传送使用高级加密标准 (AES)（使用 128 位加密密钥）和/或 PlayReady DRM 动态加密的内容。媒体服务还提供了用于向已授权客户端传送密钥和 PlayReady 许可证的服务。

当前你可以加密以下流格式：HLS、MPEG DASH 和平滑流。无法加密 HDS 流格式或渐进式下载。

如果你需要媒体服务来加密资产，则需要将加密密钥（**CommonEncryption** 或 **EnvelopeEncryption**）与资产相关联（如[此处](/documentation/articles/media-services-dotnet-create-contentkey)所述），并且配置密钥的授权策略（如本文所述）。

当播放器请求流时，媒体服务将使用指定的密钥通过 AES 或 PlayReady 加密来动态加密你的内容。为了解密流，播放器将从密钥传送服务请求密钥。为了确定用户是否被授权获取密钥，服务将评估你为密钥指定的授权策略。

媒体服务支持通过多种方式对发出密钥请求的用户进行身份验证。内容密钥授权策略可能受到一种或多种授权限制：**开放**、**令牌**限制或 **IP** 限制。令牌限制策略必须附带由安全令牌服务 (STS) 颁发的令牌。媒体服务支持采用**简单 Web 令牌** ([SWT](https://msdn.microsoft.com/zh-cn/library/gg185950.aspx#BKMK_2)) 格式和 **JSON Web 令牌**(JWT) 格式的令牌。

媒体服务不提供安全令牌服务。你可以创建自定义 STS 或利用 Windows Azure ACS 来颁发令牌。必须将 STS 配置为创建令牌，该令牌使用指定密钥以及你在令牌限制配置中指定的颁发声明进行签名（如本文所述）。如果令牌有效，而且令牌中的声明与为内容密钥配置的声明相匹配，则媒体服务密钥传送服务会将加密密钥返回到客户端。

有关详细信息，请参阅

[JWT 令牌身份验证](http://www.gtrifonov.com/2015/01/03/jwt-token-authentication-in-azure-media-services-and-dynamic-encryption/)

[将基于 Azure 媒体服务 OWIN MVC 的应用与 Azure Active Directory 相集成，并基于 JWT 声明限制内容密钥传送](http://www.gtrifonov.com/2015/01/24/mvc-owin-azure-media-services-ad-integration/)。

[使用 Azure ACS 颁发令牌](http://mingfeiy.com/acs-with-key-services)。

###请注意以下事项：

- 为了能够使用动态打包和动态加密，必须确保至少有一个流式处理保留单位。有关详细信息，请参阅[如何缩放媒体服务](/documentation/articles/media-services-manage-origins#scale_streaming_endpoints)。 
- 你的资产必须包含一组自适应比特率 MP4 或自适应比特率平滑流文件。有关详细信息，请参阅[对资产进行编码](/documentation/articles/media-services-encode-asset)。  
- 使用 **AssetCreationOptions.StorageEncrypted** 选项上载资产并对其进行编码。
- 如果你打算创建需要相同策略配置的多个内容密钥，我们强烈建议你创建单个授权策略，并将其重复用于多个内容密钥。
- 密钥传送服务将 ContentKeyAuthorizationPolicy 及其相关对象（策略选项和限制）缓存 15 分钟。如果你创建 ContentKeyAuthorizationPolicy 并指定使用“令牌”限制，然后对其进行测试，再将策略更新为“开放”限制，则现有策略切换到“开放”版本的策略需要大约 15 分钟。
- 如果你添加或更新资产的传送策略，则必须删除现有定位符（如果有）并创建新定位符。


##AES-128 动态加密 

###开放限制

开放限制意味着系统会将密钥传送到发出密钥请求的任何用户。此限制可能适用于测试用途。

以下示例创建开放授权策略，并将其添加到内容密钥。
	
	static public void AddOpenAuthorizationPolicy(IContentKey contentKey)
	{
	    // Create ContentKeyAuthorizationPolicy with Open restrictions 
	    // and create authorization policy             
	    IContentKeyAuthorizationPolicy policy = _context.
	                            ContentKeyAuthorizationPolicies.
	                            CreateAsync("Open Authorization Policy").Result;
	
	    List<ContentKeyAuthorizationPolicyRestriction> restrictions =
	        new List<ContentKeyAuthorizationPolicyRestriction>();
	
	    ContentKeyAuthorizationPolicyRestriction restriction =
	        new ContentKeyAuthorizationPolicyRestriction
	        {
	            Name = "HLS Open Authorization Policy",
	            KeyRestrictionType = (int)ContentKeyRestrictionType.Open,
	            Requirements = null // no requirements needed for HLS
	        };
	
	    restrictions.Add(restriction);
	
	    IContentKeyAuthorizationPolicyOption policyOption =
	        _context.ContentKeyAuthorizationPolicyOptions.Create(
	        "policy", 
	        ContentKeyDeliveryType.BaselineHttp, 
	        restrictions, 
	        "");
	
	    policy.Options.Add(policyOption);
	
	    // Add ContentKeyAutorizationPolicy to ContentKey
	    contentKey.AuthorizationPolicyId = policy.Id;
	    IContentKey updatedKey = contentKey.UpdateAsync().Result;
	    Console.WriteLine("Adding Key to Asset: Key ID is " + updatedKey.Id);
	}


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

使用 **媒体服务 SDK for .NET** 时，可以使用 **TokenRestrictionTemplate** 类来生成限制令牌。以下示例创建包含令牌限制的授权策略。在此示例中，客户端必须出示令牌，其中包含：签名密钥 (VerificationKey)、令牌颁发者和必需的声明。
	
	public static string AddTokenRestrictedAuthorizationPolicy(IContentKey contentKey)
	{
	    string tokenTemplateString = GenerateTokenRequirements();
	
	    IContentKeyAuthorizationPolicy policy = _context.
	                            ContentKeyAuthorizationPolicies.
	                            CreateAsync("HLS token restricted authorization policy").Result;
	
	    List<ContentKeyAuthorizationPolicyRestriction> restrictions =
	            new List<ContentKeyAuthorizationPolicyRestriction>();
	
	    ContentKeyAuthorizationPolicyRestriction restriction =
	            new ContentKeyAuthorizationPolicyRestriction
	            {
	                Name = "Token Authorization Policy",
	                KeyRestrictionType = (int)ContentKeyRestrictionType.TokenRestricted,
	                Requirements = tokenTemplateString
	            };
	
	    restrictions.Add(restriction);
	
	    //You could have multiple options 
	    IContentKeyAuthorizationPolicyOption policyOption =
	        _context.ContentKeyAuthorizationPolicyOptions.Create(
	            "Token option for HLS",
	            ContentKeyDeliveryType.BaselineHttp,
	            restrictions,
	            null  // no key delivery data is needed for HLS
	            );
	
	    policy.Options.Add(policyOption);
	
	    // Add ContentKeyAutorizationPolicy to ContentKey
	    contentKey.AuthorizationPolicyId = policy.Id;
	    IContentKey updatedKey = contentKey.UpdateAsync().Result;
	    Console.WriteLine("Adding Key to Asset: Key ID is " + updatedKey.Id);
	
	    return tokenTemplateString;
	}
	
	static private string GenerateTokenRequirements()
	{
	    TokenRestrictionTemplate template = new TokenRestrictionTemplate(TokenType.SWT);
	
	    template.PrimaryVerificationKey = new SymmetricVerificationKey();
	    template.AlternateVerificationKeys.Add(new SymmetricVerificationKey());
            template.Audience = _sampleAudience.ToString();
            template.Issuer = _sampleIssuer.ToString();
	
	    template.RequiredClaims.Add(TokenClaim.ContentKeyIdentifierClaim);
	
	    return TokenRestrictionTemplateSerializer.Serialize(template);
	}

####<a id="test"></a>测试令牌

若要获取用于密钥授权策略的基于令牌限制的测试令牌，请执行以下操作。
	
	// Deserializes a string containing an Xml representation of a TokenRestrictionTemplate
	// back into a TokenRestrictionTemplate class instance.
	TokenRestrictionTemplate tokenTemplate =
	    TokenRestrictionTemplateSerializer.Deserialize(tokenTemplateString);
	
	// Generate a test token based on the the data in the given TokenRestrictionTemplate.
	// Note, you need to pass the key id Guid because we specified 
	// TokenClaim.ContentKeyIdentifierClaim in during the creation of TokenRestrictionTemplate.
	Guid rawkey = EncryptionUtils.GetKeyIdAsGuid(key.Id);
	
	//The GenerateTestToken method returns the token without the word “Bearer” in front
	//so you have to add it in front of the token string. 
	string testToken = TokenRestrictionTemplateSerializer.GenerateTestToken(tokenTemplate, null, rawkey);
	Console.WriteLine("The authorization token is:\nBearer {0}", testToken);
	Console.WriteLine();


##PlayReady 动态加密 

媒体服务允许你配置相应的权限和限制，以便在用户尝试播放受保护的内容时，PlayReady DRM 运行时会强制实施这些权限和限制。

使用 PlayReady 保护你的内容时，需要在授权策略中指定的项目之一是用于定义 [PlayReady 许可证模板](/documentation/articles/media-services-playready-license-template-overview)的 XML 字符串。在 媒体服务 SDK for .NET 中，**PlayReadyLicenseResponseTemplate** 和 **PlayReadyLicenseTemplate** 类将帮助你定义 PlayReady 许可证模板。

###开放限制
	
开放限制意味着系统会将密钥传送到发出密钥请求的任何用户。此限制可能适用于测试用途。

以下示例创建开放授权策略，并将其添加到内容密钥。

	static public void AddOpenAuthorizationPolicy(IContentKey contentKey)
	{
	
	    // Create ContentKeyAuthorizationPolicy with Open restrictions 
	    // and create authorization policy          
	
	    List<ContentKeyAuthorizationPolicyRestriction> restrictions = new List<ContentKeyAuthorizationPolicyRestriction>
	    {
	        new ContentKeyAuthorizationPolicyRestriction 
	        { 
	            Name = "Open", 
	            KeyRestrictionType = (int)ContentKeyRestrictionType.Open, 
	            Requirements = null
	        }
	    };
	
	    // Configure PlayReady license template.
	    string newLicenseTemplate = ConfigurePlayReadyLicenseTemplate();
	
	    IContentKeyAuthorizationPolicyOption policyOption =
	        _context.ContentKeyAuthorizationPolicyOptions.Create("",
	            ContentKeyDeliveryType.PlayReadyLicense,
	                restrictions, newLicenseTemplate);
	
	    IContentKeyAuthorizationPolicy contentKeyAuthorizationPolicy = _context.
	                ContentKeyAuthorizationPolicies.
	                CreateAsync("Deliver Common Content Key with no restrictions").
	                Result;
	
	
	    contentKeyAuthorizationPolicy.Options.Add(policyOption);
	
	    // Associate the content key authorization policy with the content key.
	    contentKey.AuthorizationPolicyId = contentKeyAuthorizationPolicy.Id;
	    contentKey = contentKey.UpdateAsync().Result;
	}

###令牌限制

若要配置令牌限制选项，你需要使用 XML 来描述令牌的授权要求。令牌限制配置 XML 必须符合[此](#schema)部分中所示的 XML 架构。
	
	public static string AddTokenRestrictedAuthorizationPolicy(IContentKey contentKey)
	{
	    string tokenTemplateString = GenerateTokenRequirements();
	
	    IContentKeyAuthorizationPolicy policy = _context.
	                            ContentKeyAuthorizationPolicies.
	                            CreateAsync("HLS token restricted authorization policy").Result;
	
	    List<ContentKeyAuthorizationPolicyRestriction> restrictions = new List<ContentKeyAuthorizationPolicyRestriction>
	    {
	        new ContentKeyAuthorizationPolicyRestriction 
	        { 
	            Name = "Token Authorization Policy", 
	            KeyRestrictionType = (int)ContentKeyRestrictionType.TokenRestricted,
	            Requirements = tokenTemplateString, 
	        }
	    };
	
	    // Configure PlayReady license template.
	    string newLicenseTemplate = ConfigurePlayReadyLicenseTemplate();
	
	    IContentKeyAuthorizationPolicyOption policyOption =
	        _context.ContentKeyAuthorizationPolicyOptions.Create("Token option",
	            ContentKeyDeliveryType.PlayReadyLicense,
	                restrictions, newLicenseTemplate);
	
	    IContentKeyAuthorizationPolicy contentKeyAuthorizationPolicy = _context.
	                ContentKeyAuthorizationPolicies.
	                CreateAsync("Deliver Common Content Key with no restrictions").
	                Result;
	            
	    policy.Options.Add(policyOption);
	
	    // Add ContentKeyAutorizationPolicy to ContentKey
	    contentKeyAuthorizationPolicy.Options.Add(policyOption);
	
	    // Associate the content key authorization policy with the content key
	    contentKey.AuthorizationPolicyId = contentKeyAuthorizationPolicy.Id;
	    contentKey = contentKey.UpdateAsync().Result;
	
	    return tokenTemplateString;
	}
	
	static private string GenerateTokenRequirements()
	{
	
	    TokenRestrictionTemplate template = new TokenRestrictionTemplate(TokenType.SWT);
	
	    template.PrimaryVerificationKey = new SymmetricVerificationKey();
	    template.AlternateVerificationKeys.Add(new SymmetricVerificationKey());
            template.Audience = _sampleAudience.ToString();
            template.Issuer = _sampleIssuer.ToString();
	
	
	    template.RequiredClaims.Add(TokenClaim.ContentKeyIdentifierClaim);
	
	    return TokenRestrictionTemplateSerializer.Serialize(template);
	} 
	
	static private string ConfigurePlayReadyLicenseTemplate()
	{
	    // The following code configures PlayReady License Template using .NET classes
	    // and returns the XML string.
	             
	    PlayReadyLicenseResponseTemplate responseTemplate = new PlayReadyLicenseResponseTemplate();
	    PlayReadyLicenseTemplate licenseTemplate = new PlayReadyLicenseTemplate();
	
	    responseTemplate.LicenseTemplates.Add(licenseTemplate);
	
	    return MediaServicesLicenseTemplateSerializer.Serialize(responseTemplate);
	}

若要获取用于密钥授权策略的基于令牌限制的测试令牌，请参阅[此](#test)部分。

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
    }

###<a id="TokenType"></a>TokenType

    public enum TokenType
    {
        Undefined = 0,
        SWT = 1,
        JWT = 2,
    }



##后续步骤
在配置内容密钥的授权策略后，请转到[如何配置资产传送策略](/documentation/articles/media-services-dotnet-configure-asset-delivery-policy)主题。

<!---HONumber=74-->