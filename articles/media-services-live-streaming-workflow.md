<properties 
	pageTitle="使用 Azure 媒体服务传送实时流" 
	description="此主题概述实时流式处理中所涉及的主要组件。" 
	services="media-services" 
	documentationCenter="" 
	authors="Juliako" 
	manager="dwrede" 
	editor=""/>

<tags
	ms.service="media-services"
	ms.date="04/18/2016" 
	wacn.date="06/27/2016"/>


#使用 Azure 媒体服务传递实时流式处理事件

##概述

使用实时流式处理时，通常涉及以下组件：

- 一个用于广播事件的相机。
- 一个将信号从相机转换为发送至实时流式处理服务的流的实时视频编码器。 
  
	（可选）多个实时编码器。对于某些需要高可用性与优质体验的重要实时事件，建议使用主动-主动冗余编码器，以实现无缝故障转移，且不会丢失数据。
- 实时流式处理服务允许你执行以下操作： 
	- 使用多种实时流式处理协议（例如 RTMP 或平滑流式处理）引入实时内容 
	- 将你的流编码为自适应比特率流
	- 预览你的实时流
	- 存储引入的内容，以便稍后进行流式处理（视频点播）
	- 直接通过常用流式处理协议（例如 MPEG DASH、Smooth、HLS、HDS）将内容传递给客户，或传递至内容传送网络 (CDN) 以供进一步分发。 
	
		
**Azure 媒体服务** (AMS) 提供了引入、编码、预览、存储和传送实时流式处理内容的功能。

在将内容传送给客户时，你的目标就是：将优质视频传递到处于不同网络条件下的各种设备。为了满足质量和网络条件的要求，使用实时编码器将流编码为多比特率（自适应比特率）视频流。为满足不同设备的流式处理要求，使用媒体服务[动态打包](/documentation/articles/media-services-dynamic-packaging-overview/)将流动态地重新打包为不同的协议。媒体服务支持传送以下自适应比特率流式处理技术：HTTP 实时流式处理 (HLS)、平滑流式处理、MPEG DASH 和 HDS（仅适用于 Adobe PrimeTime/Access 许可证持有人）。

在 Azure 媒体服务中，**频道**、**程序**和**流式处理终结点**处理所有实时流式处理功能，包括引入、格式化、DVR、安全性、缩放性和冗余。

**频道**表示用于处理实时流内容的管道。当前，频道可以通过以下方式接收实时输入流：


- 本地实时编码器（采用以下格式之一：RTP (MPEG-TS)、RTMP 或平滑流式处理 （分片 MP4））将单比特率流发送至能够使用媒体服务执行实时编码的频道。然后，频道将对传入的单比特率流执行实时编码，使之转换为多比特率（自适应）视频流。收到请求时，媒体服务会将该流传递给客户。

- 本地实时编码器将多比特率 **RTMP** 或**平滑流式处理**（分片 MP4）发送到频道。可以使用以下输出多比特率平滑流的实时编码器：Elemental、Envivio、Cisco。以下实时编码器输出 RTMP：Adobe Flash Live、Telestream Wirecast 和 Tricaster 转码器。引入流将通过**频道**，而不会进行任何进一步处理。你的实时编码器也可将单比特率流发送到并未启用实时编码的频道，并不建议这样做。收到请求时，媒体服务会将该流传递给客户。


##使用能够通过 Azure 媒体服务执行实时编码的频道


下图显示的是实时流式处理工作流中所涉及的 AMS 平台的主要组成部分，该工作流中的频道能够通过媒体服务执行实时编码。

![实时工作流][live-overview1]

有关详细信息，请参阅[使用能够通过 Azure 媒体服务执行实时编码的频道](/documentation/articles/media-services-manage-live-encoder-enabled-channels/)。


##使用从本地编码器接收多比特率实时流的频道


下图显示的是此实时流式处理工作流中所涉及的 AMS 平台的主要组成部分。

![实时工作流][live-overview2]

有关详细信息，请参阅[使用从本地编码器接收多比特率实时流的频道](/documentation/articles/media-services-live-streaming-with-onprem-encoders/)。






##相关主题

[媒体服务概念](/documentation/articles/media-services-concepts/)

[Azure 媒体服务分片 MP4 实时引入规范](/documentation/articles/media-services-fmp4-live-ingest-overview/)





[live-overview1]: ./media/media-services-live-streaming-workflow/media-services-live-streaming-new.png

[live-overview2]: ./media/media-services-live-streaming-workflow/media-services-live-streaming-current.png
 
<!---HONumber=Mooncake_0620_2016-->