<properties 
	pageTitle="保护内容 - Azure 教程" 
	description="本主题概述如何使用 Media Services 保护你的内容。" 
	authors="Juliako" 
	manager="dwrede" 
	editor="" 
	services="media-services" 
	documentationCenter=""/>

<tags 
wacn.date="05/15/2015"
	ms.service="media-services" 
	ms.date="04/15/2015"/>


# 保护内容

## 概述

使用 Azure Media Services，可以在媒体从离开计算机到存储、处理和传送的整个过程中确保其安全。使用 Media Services 时，一种常见情况如下：

### 在存储中保护内容，并以动态方式传送加密的流媒体  

若要使用动态加密，首先必须获取你想要从中流式传输加密内容的流式处理终结点的至少一个流式处理保留单元。

1. 将优质夹层文件上载到资产中。向资产应用存储加密选项。
1. 编码为自适应比特率 MP4 集。向输出资产应用存储加密选项。
1. 为播放期间想要动态加密的资产创建加密内容密钥。
2. 配置内容密钥授权策略。
1. 配置资产传送策略（由动态打包和动态加密使用）。
1. 通过创建 OnDemand 定位符来发布资产。
1. 流式传输已发布的内容。  

本主题将概述主要[概念](#concepts) ，并提供说明如何执行内容传送相关[任务](#tasks)的主题的链接。

## <a id="concepts"></a>概念

### 资产加密选项

根据你要上载、存储和传递的内容的不同类型，Media Services 提供了多个加密选项供你选择。

**无** - 不使用加密。这是默认值。请注意，使用此选项时，你的内容在传送过程中或静态存储过程中都不会受到保护。

如果计划使用渐进式下载交付 MP4，则使用此选项上载内容。

**StorageEncrypted** - 使用此选项可以通过 AES 256 位加密在本地加密明文内容，然后将其上载到 Azure 存储空间中以加密形式静态存储相关内容。受存储加密保护的资产将在编码前自动解密并放入经过加密的文件系统中，并可选择在重新上载为新的输出资产前重新加密。存储加密的主要用例是在磁盘上通过静态增强加密来保护高品质的输入媒体文件。 

若要传送存储加密资产，必须配置资产的传送策略，以使 Media Services 了解要如何传送你的内容。在流式传输资产之前，流式处理服务器会删除存储加密，然后再使用指定的传传送策略（例如 AES、PlayReady 或无加密）流式传输你的内容。 

**CommonEncryptionProtected** - 如果要采用常用加密或 PlayReady DRM 加密（或上载已加密的）内容（例如，受 PlayReady DRM 保护的平滑流），请使用此选项。

**EnvelopeEncryptionProtected** - 如果要保护（或上载已加密的）采用高级加密标准 (AES) 加密的 HTTP 实时流 (HLS)，请使用此选项。请注意，如果上载已采用 AES 加密的 HLS，则该 HLS 必须已经由 Transform Manager 加密。

### 动态加密

借助 Windows Azure Media Services，你可以传送使用高级加密标准 (AES)（使用 128 位加密密钥）和 PlayReady DRM 动态加密的内容。 

当前，你可以加密下列流格式：HLS、MPEG DASH 和平滑流。无法加密 HDS 流格式或渐进式下载。

如果你需要 Media Services 来加密资产，则需要将加密密钥（CommonEncryption 或 EnvelopeEncryption）与资产相关联，并且配置密钥的授权策略。

你还需要配置资产的传送策略。如果你要流式传输存储加密的资产，请确保通过配置资产传送策略来指定该资产的传送方式。  

当播放器请求流时，Media Services 将使用指定的密钥通过 AES 或 PlayReady 加密来动态加密你的内容。为了解密流，播放器将从密钥传送服务请求密钥。为了确定用户是否被授权获取密钥，服务将评估你为密钥指定的授权策略。

>[AZURE.NOTE]若要利用动态加密，首先必须获取你计划从中传送内容的流式处理终结点的至少一个点播流单元。有关详细信息，请参阅[如何缩放 Media Services](/documentation/articles/media-services-manage-origins#scale_streaming_endpoints)。

### PlayReady DRM 许可证和 AES 明文密钥传送服务

Media Services 提供用于向已授权客户端传送 PlayReady 许可证和 AES 明文密钥的服务。你可以使用 Azure 管理门户、REST API 或 Media Services SDK for .NET 来配置许可证和密钥的授权与身份验证策略。

请注意，如果使用门户，则你可以配置一个 AES 策略（将应用到所有 AES 加密内容）和一个 PlayReady 策略（将应用到所有 PlayReady 加密内容）。如果你想要以更大的力度控制配置，请使用 Media Services SDK for .NET。

### PlayReady 许可证模板

Media Services 提供了用于传送 PlayReady 许可证的服务。当最终用户播放器（例如 Silverlight）尝试播放受 PlayReady 保护的内容时，将向许可证交付服务发送请求以获取许可证。如果许可证服务批准了该请求，则会颁发该许可证，该许可证将发送到客户端，并可用于解密和播放指定的内容。

许可证包含在用户尝试播放受保护的内容时要由 PlayReady DRM 运行时强制实施的权限和限制。Media Services 提供了可让你配置 PlayReady 许可证的 API。有关详细信息，请参阅 [Media Services PlayReady 许可证模板概述](https://msdn.microsoft.com/zh-CN/library/azure/dn783459.aspx)

### 令牌限制

内容密钥授权策略可能受到一种或多种授权限制：开放、令牌限制或 IP 限制。令牌限制策略必须附带由安全令牌服务 (STS) 颁发的令牌。Media Services 支持采用简单 Web 令牌 (SWT) 格式和 JSON Web 令牌 (JWT) 格式的令牌。Media Services 不提供安全令牌服务。你可以创建自定义 STS 或利用 Windows Azure ACS 来颁发令牌。必须将 STS 配置为创建令牌，该令牌使用指定密钥以及你在令牌限制配置中指定的颁发声明进行签名。如果令牌有效，而且令牌中的声明与为密钥（或许可证）配置的声明相匹配，则 Media Services 密钥传送服务会将请求的密钥（或许可证）返回到客户端。

在配置令牌限制策略时，必须指定主验证密钥、颁发者和受众参数。主验证密钥包含用来为令牌签名的密钥，颁发者是颁发令牌的安全令牌服务。受众（有时称为范围）描述该令牌的意图，或者令牌授权访问的资源。Media Services 密钥交付服务将验证令牌中的这些值是否与模板中的值匹配。

## <a id="tasks"></a>相关任务

### 配置流式处理终结点

有关流式处理终结点的概述以及如何管理它们的信息，请参阅[如何在 Media Services 帐户中管理流式处理终结点](/documentation/articles/media-services-manage-origins)。

### 上载媒体 

使用 **Azure 管理门户**、".NET"或"REST API"上载文件。

[AZURE.INCLUDE [media-services-selector-upload-files](../includes/media-services-selector-upload-files.md)]

### 为资产编码

使用 **Azure 管理门户**、**.NET** 或 **REST API** 通过 **Azure Media Encoder** 进行编码。
 
[AZURE.INCLUDE [media-services-selector-encode](../includes/media-services-selector-encode.md)]

### 创建内容密钥

使用 **.NET** 或 **REST API** 创建用来为资产加密的内容密钥。

[AZURE.INCLUDE [media-services-selector-create-contentkey](../includes/media-services-selector-create-contentkey.md)]

### 配置内容密钥授权策略 

使用 **.NET** 或 **REST API** 配置内容保护和密钥授权策略。

[AZURE.INCLUDE [media-services-selector-content-key-auth-policy](../includes/media-services-selector-content-key-auth-policy.md)]

### 配置资产传送策略

使用 **.NET** 或 **REST API** 配置资产传送策略。

[AZURE.INCLUDE [media-services-selector-asset-delivery-policy](../includes/media-services-selector-asset-delivery-policy)]


<!-- HONumber=53 -->  
