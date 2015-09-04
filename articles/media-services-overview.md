<properties 
	pageTitle="Azure 媒体服务概述" 
	description="本主题提供 Azure 媒体服务的概述" 
	services="media-services" 
	documentationCenter="" 
	authors="Juliako" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="media-services" 
	ms.date="05/26/2015" 
	wacn.date="08/29/2015"/>

#Azure 媒体服务概述

Microsoft Azure 媒体服务是一个可扩展的基于云的平台，使开发人员能够生成可缩放的媒体管理和传送应用程序。媒体服务基于 REST API，你可以使用这些 API 安全地上载、存储、编码和打包视频或音频内容，以供点播以及以实时流形式传送到各种客户端（例如，电视、电脑和移动设备）。

可以完全使用媒体服务构建端到端工作流。也可以选择使用第三方组件来构建工作流的某些组成部分。例如，使用第三方编码器进行编码。然后，使用媒体服务进行上载、保护、打包和传送。

若要构建媒体服务解决方案，你可以使用：

- [媒体服务 REST API](https://msdn.microsoft.com/zh-cn/library/azure/hh973617.aspx)
- 以下可用客户端 SDK 之一：[适用于 .NET Azure 媒体服务 SDK](https://github.com/Azure/azure-sdk-for-media-services)、[适用于 Java 的 Azure SDK](https://github.com/Azure/azure-sdk-for-java)、[适用于 Node.js 的 Azure 媒体服务](https://github.com/fritzy/node-azure-media)、[Azure PHP SDK](https://github.com/Azure/azure-sdk-for-php)
- 现有工具：[Azure 管理门户](http://manage.windowsazure.cn/)或 [Azure-Media-Services-Explorer](https://github.com/Azure/Azure-Media-Services-Explorer)。


以下海报描绘了 Azure 媒体服务的从媒体创建到媒体使用的整个工作流。可从此处下载该海报：[Azure 媒体服务海报](http://www.microsoft.com/download/details.aspx?id=38195)。

![概述][overview]

**服务级别协议 (SLA)**：

- 对于媒体服务编码，我们保证 REST API 事务可实现 99.9% 的可用性。
- 对于流式处理，我们将以 99.9% 的可用性保证成功处理现有媒体内容的请求。
- 对于实时频道，我们保证运行中的频道在至少 99.9% 的时间都能建立外部连接。
- 对于内容保护，我们保证将在至少 99.9% 的时间成功满足密钥请求。
- 对于索引器，我们将使用编码保留单位在 99.9% 的时间成功处理索引器任务请求。

	有关更多信息，请参阅 [Microsoft Azure SLA](http://www.windowsazure.cn/support/legal/sla)。

##概念

有关详细信息，请参阅[概念](/documentation/articles/media-services-concepts)。


##使用 Azure 媒体服务交付按需媒体

以下主题介绍常见媒体服务视频点播工作流的步骤。本主题链接到演示如何使用媒体服务支持的技术实现这些步骤的其他主题。

[使用 Azure 媒体服务交付按需媒体](/documentation/articles/media-services-video-on-demand-workflow)。

##使用 Azure 媒体服务传送实时流式处理

以下主题介绍常见媒体服务实时传送视频流工作流的步骤。本主题链接到演示如何使用媒体服务支持的技术实现这些步骤的其他主题。

[使用 Azure 媒体服务传送实时传送视频流](/documentation/articles/media-services-live-streaming-workflow)。

##使用的内容

Azure 媒体服务提供你所需的工具，以便你创建适用于大多数平台的丰富、动态的客户端播放器应用程序，这些平台包括：iOS 设备、Android 设备、Windows、Windows Phone、Xbox 和机顶盒。以下主题提供了可用来开发自己的客户端应用程序（这些应用程序使用媒体服务中的流媒体）的 SDK 和播放器框架的链接。

[开发视频播放器应用程序](media-services-develop-video-players.md)

##模式与实践指南

[模式与实践指南](https://wamsg.codeplex.com/)[联机文档](https://msdn.microsoft.com/zh-cn/library/dn735912.aspx)[可下载的电子书](https://www.microsoft.com/download/details.aspx?id=42629)

##支持

[Azure 支持](http://www.windowsazure.cn/support/plans/)为 Azure（包括媒体服务）提供支持选项。

##后续步骤

[使用 Azure 媒体服务传送实时流式处理](/documentation/articles/media-services-live-streaming-workflow)

[开发视频播放器应用程序](/documentation/articles/media-services-develop-video-players)
 
[开发视频播放器应用程序](/documentation/articles/media-services-develop-video-players)


<!-- Images -->
[overview]: ./media/media-services-overview/media-services-overview.png

<!---HONumber=67-->