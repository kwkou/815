<properties
    pageTitle="使用 Azure 门户配置内容密钥授权策略 | Azure"
    description="了解如何配置内容密钥的授权策略。"
    services="media-services"
    documentationcenter=""
    author="juliako"
    manager="erikre"
    editor="" />
<tags
    ms.assetid="ee82a3fa-c34b-48f2-a108-8ba321f1691e"
    ms.service="media-services"
    ms.workload="media"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/05/2017"
    wacn.date="04/10/2017"
    ms.author="juliako" />  


#配置内容密钥授权策略
[AZURE.INCLUDE [media-services-selector-content-key-auth-policy](../../includes/media-services-selector-content-key-auth-policy.md)]


##概述

Azure 媒体服务允许传送受高级加密标准 \(AES\)（使用 128 位加密密钥）或受 [Microsoft PlayReady DRM](https://www.microsoft.com/playready/overview/) 保护的 MPEG-DASH 流、平滑流式处理流和 HTTP 实时流式处理 \(HLS\) 流。PlayReady 是按通用加密 \(ISO/IEC 23001-7 CENC\) 规范加密的。

媒体服务还提供了一个**密钥\\许可证传送服务**，客户端可从中获取 AES 密钥或 PlayReady 许可证，以用于播放加密的内容。

本主题介绍了如何使用 **Azure 经典管理门户**配置内容密钥授权策略。以后，可以使用该密钥来动态加密内容。注意，当前可以加密以下流格式：HLS、MPEG DASH 和平滑流式处理。无法加密渐进式下载。
 
播放器请求已设置为动态加密的流时，媒体服务会使用配置的密钥通过 AES 或 DRM 加密来动态加密内容。为了解密流，播放器会从密钥传送服务请求密钥。为了确定用户是否有权获取密钥，该服务会评估为密钥指定的授权策略。


如果打算创建多个内容密钥，或者想要指定除媒体服务密钥传送服务以外的**密钥\\许可证传送服务** URL，请使用媒体服务 .NET SDK 或 REST API。

[使用适用于 .NET 的媒体服务 SDK 配置内容密钥授权策略](/documentation/articles/media-services-dotnet-configure-content-key-auth-policy/)

[使用媒体服务 REST API 配置内容密钥授权策略](/documentation/articles/media-services-rest-configure-content-key-auth-policy/)

###请注意以下事项：

- 创建 AMS 帐户时，系统会将**默认**流式处理终结点以“已停止”状态添加到用户的帐户。若要开始对内容进行流式处理并利用动态打包和动态加密功能，必须确保流式处理终结点处于“正在运行”状态。
- 资产必须包含一组自适应比特率 MP4 或自适应比特率平滑流文件。有关详细信息，请参阅[对资产进行编码](/documentation/articles/media-services-encode-asset/)。
- 密钥传送服务将 ContentKeyAuthorizationPolicy 及其相关对象（策略选项和限制）缓存 15 分钟。如果创建 ContentKeyAuthorizationPolicy 并指定使用“令牌”限制，然后对其进行测试，再将策略更新为“开放”限制，则现有策略切换到“开放”版本的策略需要大约 15 分钟。


##如何：配置密钥授权策略

若要配置密钥授权策略，请选择“内容保护”页。
	
媒体服务支持通过多种方式对发出密钥请求的用户进行身份验证。内容密钥授权策略可能且受到**开放**、**令牌**或 **IP** 授权限制（可以使用 REST 或 .NET SDK 配置 **IP**）。

###开放限制

**开放**限制意味着系统会将密钥传送到发出密钥请求的任何用户。此限制可能适用于测试用途。

![OpenPolicy][open_policy]  


###令牌限制

若要选择令牌限制策略，请按“令牌”按钮。

**令牌**限制策略必须附带由“安全令牌服务”(STS) 颁发的令牌。媒体服务支持采用简单的“Web 令牌”([SWT](https://msdn.microsoft.com/zh-cn/library/gg185950.aspx#BKMK_2)) 格式和“JSON Web 令牌”(JWT) 格式的令牌。有关信息，请参阅 [JWT 令牌身份验证](http://www.gtrifonov.com/2015/01/03/jwt-token-authentication-in-azure-media-services-and-dynamic-encryption/)。

媒体服务不提供 **安全令牌服务**。可以创建自定义 STS 或利用 Azure ACS 来颁发令牌。必须将 STS 配置为创建令牌，该令牌使用指定密钥以及令牌限制配置中指定的颁发声明进行签名。如果令牌有效，而且令牌中的声明与为内容密钥配置的声明相匹配，媒体服务密钥传送服务会将加密密钥返回到客户端。有关详细信息，请参阅[使用 Azure ACS 颁发令牌](http://mingfeiy.com/acs-with-key-services)。

配置“令牌”限制策略时，必须设置“验证密钥”、“颁发者”和“受众”的值。主验证密钥包含用于为令牌签名的密钥，颁发者是颁发令牌的安全令牌服务。受众（有时称为范围）描述该令牌的意图，或者令牌授权访问的资源。媒体服务密钥传送服务验证令牌中的这些值是否与模板中的值匹配。

###PlayReady

使用 **PlayReady** 保护内容时，需要在授权策略中指定的项目之一是定义 PlayReady 许可证模板的 XML 字符串。默认情况下，已设置以下策略：
		
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

可以单击“导入策略 xml”按钮，然后提供符合[此处](/documentation/articles/media-services-playready-license-template-overview/)定义的 XML 架构的另一个 XML。


[open_policy]: ./media/media-services-portal-configure-content-key-auth-policy/media-services-protect-content-with-open-restriction.png
[token_policy]: ./media/media-services-key-authorization-policy/media-services-protect-content-with-token-restriction.png

 

<!---HONumber=Mooncake_0220_2017-->
<!--Update_Description: update notfications list; remove “后续步骤” section-->