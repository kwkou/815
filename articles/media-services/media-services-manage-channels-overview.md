<properties
    pageTitle="使用 Azure 媒体服务实时传送视频流概述 | Azure"
    description="本主题概述了如何使用 Azure 媒体服务实时传送视频流。"
    services="media-services"
    documentationcenter=""
    author="Juliako"
    manager="erikre"
    editor="" />
<tags
    ms.assetid="fb63502e-914d-4c1f-853c-4a7831bb08e8"
    ms.service="media-services"
    ms.workload="media"
    ms.tgt_pltfrm="na"
    ms.devlang="ne"
    ms.topic="article"
    ms.date="01/05/2017"
    wacn.date="02/24/2017"
    ms.author="juliako" />  


#使用 Azure 媒体服务实时传送视频流概述

##<a name="closed-captioning-and-ad-insertion"></a>概述

使用 Azure 媒体服务传递实时流式处理事件时，通常涉及以下组件：

- 一个用于广播事件的相机。
- 一个将信号从相机转换为发送至实时流式处理服务的流的实时视频编码器。

    （可选）多个实时同步编码器。对于某些需要高可用性与优质体验的重要实时事件，建议使用带时间同步功能的主动-主动冗余编码器，以实现无缝故障转移，且不会丢失数据。
* 实时流式处理服务允许执行以下操作：

  * 使用多种实时流式处理协议（例如 RTMP 或平滑流式处理）引入实时内容
  * （可选）将流编码为自适应比特率流
  * 预览实时流，
  * 记录和存储引入的内容，以便稍后进行流式处理（视频点播）
  * 直接通过常用流式传输协议（例如 MPEG DASH、平滑流式处理、HLS）将内容传递给客户，或传递至内容交付网络 \(CDN\) 以供进一步分发。


**Azure 媒体服务** (AMS) 提供了引入、编码、预览、存储和传送实时流式处理内容的功能。

将内容传送给客户时，目标是将优质视频传递到处于不同网络条件下的各种设备。为此，可使用实时编码器将流编码为多比特率（自适应比特率）视频流。为满足不同设备的流式处理要求，使用媒体服务[动态打包](/documentation/articles/media-services-dynamic-packaging-overview/)将流动态地重新打包为不同的协议。媒体服务支持传送以下自适应比特率流式处理技术：HTTP Live Streaming \(HLS\)、平滑流式处理、MPEG DASH。

在 Azure 媒体服务中，**频道**、**节目**和**流式处理终结点**处理所有实时流式处理功能，包括引入、格式化、DVR、安全性、缩放性和冗余。

**通道**表示用于处理实时流内容的管道。通道可以通过以下方式接收实时输入流：

- 本地实时编码器将多比特率 **RTMP** 或**平滑流式处理**（分片 MP4）发送到经配置可以进行**直通**传递的通道。**直通**传递是指引入的流将会直接通过**通道**，而不会经过任何进一步的处理。可以使用以下输出多比特率平滑流式处理的实时编码器：MediaExcel、Ateme、Imagine Communications、Envivio、Cisco、Elemental。以下实时编码器输出 RTMP：Adobe Flash Media Live Encoder \(FMLE\)、Telestream Wirecast、Haivision、Teradek 和 Tricaster 转码器。实时编码器也可将单比特率流发送到并未启用实时编码的通道，但不建议这样做。收到请求时，媒体服务会将该流传送给客户。

	>[AZURE.NOTE] 需要长时间处理多个事件，并且已经在本地编码器上进行了投入时，可以使用直通这种最经济的方法来实时传送视频流。请参阅[定价](/pricing/details/media-services/)详细信息。
	
	
- 本地实时编码器（采用以下格式之一：RTMP 或平滑流式处理（分片 MP4））将单比特率流发送至能够使用媒体服务执行实时编码的通道。还支持 RTP (MPEG-TS)，但前提是拥有到 Azure 数据中心的专用连接。以下提供 RTMP 输出的实时编码器可以使用此类型的通道：Telestream Wirecast、FMLE。然后，通道将对传入的单比特率流执行实时编码，使之转换为多比特率（自适应）视频流。收到请求时，媒体服务会将该流传送给客户。

从媒体服务2.10 发行版开始，创建通道时，可以指定希望通道接收输入流的方式，以及是否希望通道对流执行实时编码。可以使用两个选项：

- **无**（直通）- 如果计划使用输出多比特率流（直通流）的本地实时编码器，请指定此值。在这种情况下，传入流将传递到输出，而不会进行任何编码。这是 2.10 发行版以前的通道行为。

- **标准** - 如果计划使用媒体服务将单比特率实时流编码为多比特率流，请选择此值。若要针对不频繁发生的事件快速地向上缩放，此方法可以节省资金。请注意，实时编码会影响计费，应记住，将实时编码通道保持为“正在运行”状态会产生费用。建议在实时流式处理事件完成之后立即停止正在运行的通道，以避免产生额外的小时费用。

## 通道类型的比较
可通过下表来了解媒体服务中支持的两种通道类型的比较情况

功能|直通通道|标准通道
---|---|---
单比特率输入在云中被编码为多比特率|否|是
最大分辨率，层数|1080p，8 层，60+fps|720p，6 层，30 fps
输入协议|RTMP、平滑流|RTMP、平滑流和 RTP
价格|请参阅[定价页](/pricing/details/media-services/)并单击“实时视频”选项卡|请参阅[定价页](/pricing/details/media-services/) 
最长运行时间|全天候运行|8 小时
支持插入静态图像|否|是
支持发出广告指示|否|是
直通 CEA 608/708 字幕|是|是
能够从贡献源出现的短时停顿中恢复|是|否（如果超过 6 秒而没有输入数据，通道就会开始插入静态图像）
支持非一致性输入 GOP|是|否 - 输入必须是固定的 2 秒 GOP
支持可变帧率输入|是|否 - 输入必须是固定帧率。<br/>轻微的帧率变化是容许的，例如在高速运动情况下出现的轻微帧率变化。但是，编码器不能掉到 10 帧/秒的帧率。
输入源丢失时，将自动关闭通道|否|12 小时后，如果没有运行的程序 

## 使用从本地编码器（直通）接收多比特率实时流的通道
下图显示的是“直通”工作流中涉及的 AMS 平台的主要组成部分。

![实时工作流](./media/media-services-live-streaming-workflow/media-services-live-streaming-current.png)  


有关详细信息，请参阅[使用从本地编码器接收多比特率实时流的频道](/documentation/articles/media-services-live-streaming-with-onprem-encoders/)。

## 使用能够通过 Azure 媒体服务执行实时编码的通道
下图显示的是实时流式处理工作流中涉及的 AMS 平台的主要组成部分，该工作流中的频道能够通过媒体服务执行实时编码。

![实时工作流](./media/media-services-live-streaming-workflow/media-services-live-streaming-new.png)

有关详细信息，请参阅[使用能够通过 Azure 媒体服务执行实时编码的通道](/documentation/articles/media-services-manage-live-encoder-enabled-channels/)。

##频道及其相关组件的说明

###<a name="channel_input"></a>频道

在媒体服务中，[频道](https://docs.microsoft.com/zh-cn/rest/api/media/operations/channel)负责处理实时流内容。频道提供输入终结点（引入 URL），然后将该终结点提供给实时转码器。频道从实时转码器接收实时输入流，并通过一个或多个 StreamingEndpoints 使其可用于流式处理。频道还提供可用于预览的预览终结点（预览 URL），并在进一步处理和传递流之前对流进行验证。

可以在创建频道时获取引入 URL 和预览 URL。若要获取这些 URL，频道不一定要处于已启动状态。准备好开始将数据从实时转码器推送到频道时，频道必须已启动。实时转码器开始引入数据后，你可以预览流。

每个媒体服务帐户均可包含多个频道、多个节目以及多个 StreamingEndpoint。根据带宽和安全性需求，StreamingEndpoint 服务可专用于一个或多个频道。任何 StreamingEndpoint 都可以从任何频道拉取。


###节目 

[节目](https://docs.microsoft.com/zh-cn/rest/api/media/operations/program)用于控制实时流中片段的发布和存储。频道管理节目。频道和节目的关系非常类似于传统媒体，频道具有恒定的内容流，而节目的范围限定为该频道上的一些定时事件。
你可以通过设置 **ArchiveWindowLength** 属性，指定你希望保留多少小时的节目录制内容。此值的设置范围是最短 5 分钟，最长 25 小时。

ArchiveWindowLength 还决定了客户端能够从当前实时位置按时间向后搜索的最长时间。超出指定时间长度后，节目也能够运行，但落在时段长度后面的内容将全部被丢弃。此属性值还决定了客户端清单能够增加多长时间。

每个节目都与资产关联。若要发布节目，必须为关联的资产创建定位符。创建此定位符后，可以生成提供给客户端的流式处理 URL。

一个频道最多支持三个同时运行的节目，因此可为同一传入流创建多个存档。这样，便可以根据需要发布和存档事件的不同部分。例如，业务要求是存档 6 小时的节目，但只广播过去 10 分钟的内容。若要实现此目的，需要创建两个同时运行的节目。一个节目设置为存档 6 小时的事件但不发布该节目。另一个节目设置为存档 10 分钟的事件，并且要发布该节目。

## 计费影响
一旦通过 API 将通道的状态转换为“正在运行”，就会开始计费。

下表显示了通道状态如何映射到 API 和 Azure 经典管理门户中的计费状态。请注意，API 与门户 UX 之间的状态略有不同。一旦通过 API 将通道置于“正在运行”状态，或者在 Azure 经典管理门户中将其设置为“就绪”或“正在流式处理”状态，就会开始计费。

若要阻止通道进一步计费，必须通过 API 或 Azure 经典管理门户停止通道。使用完通道后，需要亲自停止通道。不停止通道会导致持续计费。

>[AZURE.NOTE]使用“标准”通道时，AMS 会自动关闭输入源丢失 12 小时后仍处于“正在运行”状态但没有程序运行的任何通道。但是，在通道处于“正在运行”状态的时间段内，仍会向你收费。

### <a id="states"></a>通道状态及其如何映射到计费模式
通道的当前状态。可能的值包括：

- **已停止**。这是通道在创建后的初始状态（除非在门户中选择了自动启动）。 此状态下不会发生计费。此状态下可以更新通道属性，但不允许进行流式传输。
- **正在启动**。通道正在启动。此状态下不会发生计费。此状态下不允许进行更新或流式传输。如果发生错误，通道会返回到“已停止”状态。
- **正在运行**。通道能够处理实时流。现在会计收使用费。必须停止通道以防止进一步计费。
- **正在停止**。通道正在停止。此暂时性状态下不会发生计费。此状态下不允许进行更新或流式传输。
- **正在删除**。正在删除通道。此暂时性状态下不会发生计费。此状态下不允许进行更新或流式传输。

下表显示通道状态如何映射到计费模式。
 
通道状态|门户 UI 指示器|是否计费？
---|---|---
正在启动|正在启动|否（暂时状态）
正在运行|准备就绪（没有正在运行的节目）<br/>或<br/>流式处理（至少有一个正在运行的节目）|是
正在停止|正在停止|否（暂时状态）
已停止|已停止|否





## 相关主题
[Azure 媒体服务分片 MP4 实时引入规范](/documentation/articles/media-services-fmp4-live-ingest-overview/)

[通过 Azure 媒体服务使用能够执行实时编码的通道](/documentation/articles/media-services-manage-live-encoder-enabled-channels/)

[使用从本地编码器接收多比特率实时流的频道](/documentation/articles/media-services-live-streaming-with-onprem-encoders/)

[配额和限制](/documentation/articles/media-services-quotas-and-limitations/)

[媒体服务概念](/documentation/articles/media-services-concepts/)

<!---HONumber=Mooncake_0220_2017-->
<!--Update_Description: add some new supported Encoders-->