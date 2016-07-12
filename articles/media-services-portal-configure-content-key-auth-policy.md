<properties 
	pageTitle="使用门户配置内容密钥授权策略" 
	description="了解如何配置内容密钥的授权策略。" 
	services="media-services" 
	documentationCenter="" 
	authors="juliako" 
	manager="dwrede" 
	editor=""/>

<tags
	ms.service="media-services"
 	ms.date="05/03/2016"   
	wacn.date="06/27/2016"/>



#配置内容密钥授权策略 
[AZURE.INCLUDE [media-services-selector-content-key-auth-policy](../includes/media-services-selector-content-key-auth-policy.md)]


##概述

Azure 媒体服务允许你传送受高级加密标准 (AES)（使用 128 位加密密钥）或受 [Microsoft PlayReady DRM](https://www.microsoft.com/playready/overview/) 保护的 MPEG-DASH 流、平滑流式处理流和 HTTP 实时流式处理 (HLS) 流。AMS 还允许你传送通过 Widevine DRM 加密的 DASH 流。PlayReady 和 Widevine 都是按通用加密 (ISO/IEC 23001-7 CENC) 规范加密的。

媒体服务还提供了一个**密钥\\许可证传送服务**，客户端可从中获取 AES 密钥或 PlayReady/Widevine 许可证，以用于播放加密的内容。 
本主题介绍了如何使用 **Azure 管理门户**配置内容密钥授权策略。以后，可以使用该密钥来动态加密你的内容。请注意，当前你可以加密以下流格式：HLS、MPEG DASH 和平滑流式处理。无法加密 HDS 流格式或渐进式下载。
 
当播放器请求已设置为动态加密的流时，媒体服务将使用配置的密钥通过 AES 或 DRM 加密来动态加密你的内容。为了解密流，播放器将从密钥传送服务请求密钥。为了确定用户是否被授权获取密钥，服务将评估你为密钥指定的授权策略。


如果你打算创建多个内容密钥，或者想要指定除媒体服务密钥传送服务以外的**密钥\\许可证传送服务** URL，请使用媒体服务 .NET SDK 或 REST API。

[使用适用于 .NET 的媒体服务 SDK 配置内容密钥授权策略](/documentation/articles/media-services-dotnet-configure-content-key-auth-policy/)

[使用媒体服务 REST API 配置内容密钥授权策略](/documentation/articles/media-services-rest-configure-content-key-auth-policy/)

###请注意以下事项：

- 为了能够使用动态打包和动态加密，必须确保至少有一个流式处理保留单位。有关详细信息，请参阅[如何缩放媒体服务](/documentation/articles/media-services-manage-origins/#scale_streaming_endpoints)。 
- 你的资产必须包含一组自适应比特率 MP4 或自适应比特率平滑流文件。有关详细信息，请参阅[对资产进行编码](/documentation/articles/media-services-encode-asset/)。  
- 密钥传送服务将 ContentKeyAuthorizationPolicy 及其相关对象（策略选项和限制）缓存 15 分钟。如果你创建 ContentKeyAuthorizationPolicy 并指定使用“令牌”限制，然后对其进行测试，再将策略更新为“开放”限制，则现有策略切换到“开放”版本的策略需要大约 15 分钟。


##如何：配置密钥授权策略

若要配置密钥授权策略，请选择“内容保护”页。
	
媒体服务支持通过多种方式对发出密钥请求的用户进行身份验证。内容密钥授权策略可能且受到**开放**、**令牌**或 **IP** 授权限制（可以使用 REST 或 .NET SDK 配置 **IP**）。

###开放限制

**开放**限制意味着系统会将密钥传送到发出密钥请求的任何用户。此限制可能适用于测试用途。

![OpenPolicy][open_policy]

###令牌限制

若要选择令牌限制策略，请按“令牌”按钮。

**令牌**限制策略必须附带由“安全令牌服务”(STS) 颁发的令牌。媒体服务支持采用简单的“Web 令牌”([SWT](https://msdn.microsoft.com/zh-cn/library/gg185950.aspx#BKMK_2)) 格式和“JSON Web 令牌”(JWT) 格式的令牌。有关信息，请参阅 [JWT 令牌身份验证](http://www.gtrifonov.com/2015/01/03/jwt-token-authentication-in-azure-media-services-and-dynamic-encryption/)。

媒体服务不提供 **安全令牌服务**。你可以创建自定义 STS 或利用 Azure ACS 来颁发令牌。必须将 STS 配置为创建令牌，该令牌使用指定密钥以及你在令牌限制配置中指定的颁发声明进行签名。如果令牌有效，而且令牌中的声明与为内容密钥配置的声明相匹配，则媒体服务密钥传送服务会将加密密钥返回到客户端。有关详细信息，请参阅[使用 Azure ACS 颁发令牌](http://mingfeiy.com/acs-with-key-services)。

在配置“令牌”限制策略时，必须设置“验证密钥”、“颁发者”和“受众”的值。主验证密钥包含用来为令牌签名的密钥，颁发者是颁发令牌的安全令牌服务。受众（有时称为范围）描述该令牌的意图，或者令牌授权访问的资源。媒体服务密钥交付服务将验证令牌中的这些值是否与模板中的值匹配。

###PlayReady

使用 **PlayReady** 保护你的内容时，需要在授权策略中指定的项目之一是用于定义 PlayReady 许可证模板的 XML 字符串。默认情况下，已设置以下策略：
		
	<PlayReadyLicenseResponseTemplate xmlns:i="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/Azure/MediaServices/KeyDelivery/PlayReadyTemplate/v1">
	  <LicenseTemplates>
	    <PlayReadyLicenseTemplate><AllowTestDevices>true</AllowTestDevices>
	      <ContentKey i:type="ContentEncryptionKeyFromHeader" />
	      <LicenseType>Nonpersistent</LicenseType>
	      <PlayRight>
	        <AllowPassingVideoContentToUnknownOutput>Allowed</AllowPassingVideoContentToUnknownOutput>
	      </PlayRight>
	    </PlayReadyLicenseTemplate>
	  </LicenseTemplates>
	</PlayReadyLicenseResponseTemplate>

你可以单击“导入策略 xml”按钮，然后提供符合[此处](/documentation/articles/media-services-playready-license-template-overview/)定义的 XML 架构的另一个 XML。



##后续步骤
现在已经配置了内容密钥的授权策略，请转到[如何：使用 Azure 管理门户启用加密](/documentation/articles/media-services-manage-content/#encrypt)主题。


[open_policy]: ./media/media-services-portal-configure-content-key-auth-policy/media-services-protect-content-with-open-restriction.png
[token_policy]: ./media/media-services-key-authorization-policy/media-services-protect-content-with-token-restriction.png

 
<!---HONumber=Mooncake_0620_2016-->