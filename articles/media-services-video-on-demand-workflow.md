<properties 
	pageTitle="使用 Azure 媒体服务交付按需媒体" 
	description="本主题讨论了使用 Azure 媒体服务交付按需媒体的一些常见应用场景。" 
	services="media-services" 
	documentationCenter="" 
	authors="Juliako" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="media-services" 
	ms.date="06/08/2015" 
	wacn.date="08/29/2015"/>


#使用 Azure 媒体服务交付按需媒体

##概述

本主题介绍典型 Azure Media Services (AMS) 视频点播工作流的步骤。每个步骤都链接到相关的主题。对于可以使用不同技术实现的任务，将提供相应的按钮链接到所选的技术（例如，.NET 或 REST）。

请注意，你可以将 Media Services 与现有工具和过程相集成。例如，在现场为内容编码，再将其上载到媒体服务以转码为多种格式，然后通过 Azure CDN 或第三方 CDN 交付内容。

下图显示了参与视频点播工作流的主要媒体服务平台部分。![VoD 工作流][vod-overview]

##<a id="vod_scenarios"></a>常见应用场景：按需交付媒体。 

###保护存储中的内容并以明文（非加密）形式交付流式处理媒体

1. 将优质夹层文件上载到资产中。
	
	建议向资产应用存储加密选项，以便在内容上载期间以及当内容在存储中处于静态时，为其提供保护。 
1. 编码为自适应比特率 MP4 集。 

	建议向输出资产应用存储加密选项，以便保护静态内容。
	
1. 配置资产传送策略（由动态打包使用）。
	
	如果你的资产已经过存储加密，则**必须**配置资产传送策略。

1. 通过创建 OnDemand 定位符来发布资产。

	确保你要从中以流形式传输内容的流式传输终结点上至少有一个流式传输保留单元。

1. 流式传输已发布的内容。

###在存储中保护内容，并以动态方式传送加密的流媒体  

若要使用动态加密，首先必须获取你想要从中流式传输加密内容的流式处理终结点的至少一个流式处理保留单元。

1. 将优质夹层文件上载到资产中。向资产应用存储加密选项。
1. 编码为自适应比特率 MP4 集。向输出资产应用存储加密选项。
1. 为播放期间想要动态加密的资产创建加密内容密钥。
2. 配置内容密钥授权策略。
1. 配置资产传送策略（由动态打包和动态加密使用）。
1. 通过创建 OnDemand 定位符来发布资产。
1. 流式传输已发布的内容。 

###为内容编制索引

1. 将优质夹层文件上载到资产中。
1. 为内容编制索引。

	索引作业将生成可用作视频播放中的隐藏式字幕 (CC) 的文件。它还将生成让你能够执行视频内搜索并跳转到视频确切位置的文件。

1. 使用已编制索引的内容。


###提供渐进式下载 

1. 将优质夹层文件上载到资产中。
1. 编码为自适应比特率 MP4 集或单个 MP4。
1. 通过创建 OnDemand 或 SAS 定位符来发布资产。

	如果使用 OnDemand 定位符，请确保你要从中以渐进方式下载内容的流式处理终结点上至少有一个串流保留单位。

	如果使用 SAS 定位符，将从 Azure blob 存储中下载内容。在这种情况下，不需要串流保留单位。
  
1. 渐进式下载内容。

本文包含一些链接，指向说明如何设置开发环境和执行上述任务的主题。


##概念

有关与按需交付内容相关的概念，请参阅[媒体服务概念](media-services-concepts.md)。

##常见任务：按需交付媒体

###创建 Media Services 帐户

使用 **Azure 管理门户**来[创建 Azure 媒体服务帐户](/documentation/articles/media-services-create-account)。

##设置开发环境  

为开发环境选择**“.NET”**或**“REST API”**。

[AZURE.INCLUDE [media-services-selector-setup](../includes/media-services-selector-setup.md)]

##以编程方式建立连接  

选择**“.NET”**或**“REST API”**以编程方式连接到 Azure Media Services。

[AZURE.INCLUDE [media-services-selector-connect](../includes/media-services-selector-connect.md)]


###配置流式处理终结点

有关流式处理终结点的概述以及如何管理它们的信息，请参阅[如何在媒体服务帐户中管理流式处理终结点](/documentation/articles/media-services-manage-origins)。

###上载媒体 

使用 **Azure 管理门户**、**.NET** 或 **REST API** 上载文件。

[AZURE.INCLUDE [media-services-selector-upload-files](../includes/media-services-selector-upload-files.md)]

###创建作业/任务

作业是包含有关一组任务（例如，编码或索引编制）的元数据的实体。每个任务对输入资产执行原子操作。有关如何创建编码作业的示例，请参阅：

有关概述，请参阅[处理 Azure 媒体服务作业](/documentation/articles/media-services-jobs)。

使用 **.NET** 或 **REST API** 获取适用于你的任务的媒体处理器。

[AZURE.INCLUDE [media-services-selector-get-media-processor](../includes/media-services-selector-get-media-processor.md)]

下面的示例使用 **Azure 管理门户**、**.NET** 或 **REST API** 创建编码作业。

[AZURE.INCLUDE [media-services-selector-encode](../includes/media-services-selector-encode.md)]

####索引

[AZURE.INCLUDE [media-services-selector-index-content](../includes/media-services-selector-index-content.md)]

####编码 

**概述**：

- [动态打包概述](/documentation/articles/media-services-dynamic-packaging-overview)
- [使用 Azure 媒体服务对按需内容进行编码](/documentation/articles/media-services-encode-asset)。

使用 **Azure 管理门户**、**.NET** 或 **REST API** 通过 **Azure 媒体编码器**进行编码。
 
[AZURE.INCLUDE [media-services-selector-encode](../includes/media-services-selector-encode.md)]

<!--使用 **.NET** 通过**媒体编码器高级工作流**进行高级编码。

[AZURE.INCLUDE [media-services-selector-advanced-encoding](../includes/media-services-selector-advanced-encoding.md)]-->

####监视作业进度

使用 **Azure 管理门户**、**.NET** 或 **REST API** 监视作业进度。

[AZURE.INCLUDE [media-services-selector-job-progress](../includes/media-services-selector-job-progress.md)]

###保护内容 

**概述**：

[内容保护概述](/documentation/articles/media-services-content-protection-overview)

如果要使用高级加密标准 (AES)（使用 128 位加密密钥）或 PlayReady DRM 对资产加密，则需要创建内容密钥。

使用 **.NET** 或 **REST API** 创建密钥。

[AZURE.INCLUDE [media-services-selector-create-contentkey](../includes/media-services-selector-create-contentkey.md)]

创建内容密钥后，可以使用 **.NET** 或 **REST API** 配置密钥授权策略。

[AZURE.INCLUDE [media-services-selector-content-key-auth-policy](../includes/media-services-selector-content-key-auth-policy.md)]


使用 **.NET** 或 **REST API** 配置资源传送策略。

[AZURE.INCLUDE [media-services-selector-asset-delivery-policy](../includes/media-services-selector-asset-delivery-policy.md)]


####与合作伙伴集成

[使用 castLabs 将 DRM 许可证传送到 Azure Media Services](/documentation/articles/media-services-castlabs-integration)

###发布和传送资源

动态打包概述

> [AZURE.SELECTOR]
- [Overview](/documentation/articles/media-services-dynamic-packaging-overview)


传送内容概述

> [AZURE.SELECTOR]
- [Overview](/documentation/articles/media-services-deliver-content-overview)

使用 **Azure 管理门户**、**.NET** 或 **REST API** 发布资产（通过创建定位符）。

[AZURE.INCLUDE [media-services-selector-publish](../includes/media-services-selector-publish.md)]

###启用 Azure CDN

Media Services 支持与 Azure CDN 集成。有关如何启用 Azure CDN 的信息，请参阅[如何在 Media Services 帐户中管理流式处理终结点](/documentation/articles/media-services-manage-origins#enable_cdn)。

###缩放 Media Services 帐户

通过指定要为帐户预配的**串流保留单位**和**编码保留单位**的数量，可以缩放**媒体服务**。

也可以通过向媒体服务帐户添加存储帐户来缩放该帐户。每个存储帐户大小限制为 500 TB。若要在默认限制之外扩展存储，可选择将多个存储帐户附加到单个媒体服务帐户。

[本](/documentation/articles/media-services-how-to-scale)主题链接到相关的主题。

###使用现有播放器播放内容

有关详细信息，请参阅[使用现有播放器播放内容](/documentation/articles/media-services-playback-content-with-existing-players)。


[vod-overview]: ./media/media-services-video-on-demand-workflow/media-services-video-on-demand.png
 

<!---HONumber=67-->