<properties 
	pageTitle="筛选器和动态清单" 
	description="本主题介绍如何创建筛选器，以便客户端能够使用它们来流式传输流的特定部分。媒体服务将创建动态清单来存档此选择性的流。" 
	services="media-services" 
	documentationCenter="" 
	authors="cenkdin,Juliako" 
	manager="dwrede" 
	editor=""/>

<tags
	ms.service="media-services"
 	ms.date="05/03/2016" 
	wacn.date="06/20/2016"/>

#筛选器和动态清单

从 2.11 版开始，媒体服务可让你为资产定义筛选器。这些筛选器是服务器端规则，可让你的客户选择运行如下操作：只播放一段视频（而非播放完整视频），或只指定客户设备可以处理的一部分音频和视频再现内容（而非与该资产相关的所有再现内容）。通过按客户请求创建的**动态清单**可以实现对资产进行这种筛选，并基于指定的筛选器流式传输视频。

本主题讨论对你客户很有帮助的常见筛选器使用方案，并提供主题链接以演示如何以编程方式创建筛选器（目前你只能使用 REST API 创建筛选器）。

##概述

在将内容传送到客户（流式传输实时事件或视频点播）时，你的目标就是：将优质视频传递到处于不同网络条件下的各种设备。若要实现此目标，请执行以下操作：

- 将你的流编码成多比特率（[自适应比特率](http://zh.wikipedia.org/wiki/自适性串流)）视频流（这将会负责处理质量和网络条件），并 
- 使用媒体服务[动态打包](/documentation/articles/media-services-dynamic-packaging-overview/)将你的流动态地重新打包成不同的协议（这将会负责不同设备上的流式处理）。媒体服务支持传送以下自适应比特率流式处理技术：HTTP 实时流式处理 (HLS)、平滑流式处理、MPEG DASH 和 HDS（仅适用于 Adobe PrimeTime/Access 许可证持有人）。 

###清单文件 

将资产编码为以自适应比特率流式传输时，会创建一个**清单**（播放列表）文件（此文件基于文本或 XML）。**清单**文件包含流元数据，例如：轨迹类型（音频、视频或文本）、轨迹名称、开始和结束时间、比特率（质量）、轨迹语言、演播窗口（持续时间固定的滑动窗口）和视频编解码器 (FourCC)。此文件还会通过提供有关下一个可播放视频片段及其位置的信息，来指示播放器检索下一个片段。片段（或段）实际上是视频内容的“区块”。


下面是清单文件的一个示例：

	
	<?xml version="1.0" encoding="UTF-8"?>	
	<SmoothStreamingMedia MajorVersion="2" MinorVersion="0" Duration="330187755" TimeScale="10000000">
	
	<StreamIndex Chunks="17" Type="video" Url="QualityLevels({bitrate})/Fragments(video={start time})" QualityLevels="8">
	<QualityLevel Index="0" Bitrate="5860941" FourCC="H264" MaxWidth="1920" MaxHeight="1080" CodecPrivateData="0000000167640028AC2CA501E0089F97015202020280000003008000001931300016E360000E4E1FF8C7076850A4580000000168E9093525" />
	<QualityLevel Index="1" Bitrate="4602724" FourCC="H264" MaxWidth="1920" MaxHeight="1080" CodecPrivateData="0000000167640028AC2CA501E0089F97015202020280000003008000001931100011EDC00002CD29FF8C7076850A45800000000168E9093525" />
	<QualityLevel Index="2" Bitrate="3319311" FourCC="H264" MaxWidth="1280" MaxHeight="720" CodecPrivateData="000000016764001FAC2CA5014016EC054808080A00000300020000030064C0800067C28000103667F8C7076850A4580000000168E9093525" />
	<QualityLevel Index="3" Bitrate="2195119" FourCC="H264" MaxWidth="960" MaxHeight="540" CodecPrivateData="000000016764001FAC2CA503C045FBC054808080A000000300200000064C1000044AA0000ABA9FE31C1DA14291600000000168E9093525" />
	<QualityLevel Index="4" Bitrate="1469881" FourCC="H264" MaxWidth="960" MaxHeight="540" CodecPrivateData="000000016764001FAC2CA503C045FBC054808080A000000300200000064C04000B71A0000E4E1FF8C7076850A4580000000168E9093525" />
	<QualityLevel Index="5" Bitrate="978815" FourCC="H264" MaxWidth="640" MaxHeight="360" CodecPrivateData="000000016764001EAC2CA50280BFE5C0548303032000000300200000064C08001E8480004C4B7F8C7076850A45800000000168E9093525" />
	<QualityLevel Index="6" Bitrate="638374" FourCC="H264" MaxWidth="640" MaxHeight="360" CodecPrivateData="000000016764001EAC2CA50280BFE5C0548303032000000300200000064C080013D60000C65DFE31C1DA1429160000000168E9093525" />
	<QualityLevel Index="7" Bitrate="388851" FourCC="H264" MaxWidth="320" MaxHeight="180" CodecPrivateData="000000016764000DAC2CA505067E7C054830303200000300020000030064C040030D40003D093F8C7076850A45800000000168E9093525" />
	
	<c t="0" d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="20000000" /><c d="9600000"/>
	</StreamIndex>
	
	
	<StreamIndex Chunks="17" Type="audio" Url="QualityLevels({bitrate})/Fragments(AAC_und_ch2_128kbps={start time})" QualityLevels="1" Name="AAC_und_ch2_128kbps">
	<QualityLevel AudioTag="255" Index="0" BitsPerSample="16" Bitrate="125658" FourCC="AACL" CodecPrivateData="1210" Channels="2" PacketSize="4" SamplingRate="44100" />
	
	<c t="0" d="20201360" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201360" /><c d="6965987" /></StreamIndex>
	
	
	<StreamIndex Chunks="17" Type="audio" Url="QualityLevels({bitrate})/Fragments(AAC_und_ch2_56kbps={start time})" QualityLevels="1" Name="AAC_und_ch2_56kbps">
	<QualityLevel AudioTag="255" Index="0" BitsPerSample="16" Bitrate="53655" FourCC="AACL" CodecPrivateData="1210" Channels="2" PacketSize="4" SamplingRate="44100" />
	
	<c t="0" d="20201360" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201361" /><c d="20201360" /><c d="20201361" /><c d="20201360" /><c d="6965987" /></StreamIndex>
	
	</SmoothStreamingMedia>
	
###动态清单

在某些[情况](/documentation/articles/media-services-dynamic-manifest-overview/#scenarios)下，默认资产的清单文件中描述的内容无法满足客户端所需的灵活性。例如：

- 特定于设备：只传送内容播放设备所支持的指定再现内容和/或指定的语言轨迹（“再现内容筛选”）。 
- 缩小清单以显示实时事件的子剪辑（“子剪辑筛选”）。
- 修剪视频开头（“修剪视频”）。
- 调整演播窗口，以便在播放器中提供长度有限的 DVR 窗口（“调整演播窗口”）。
 
为实现这种灵活性，媒体服务会根据预定义的[筛选器](/documentation/articles/media-services-dynamic-manifest-overview/#filters)提供**动态清单**。在你定义筛选器后，客户端将会使用筛选器来流式传输视频的特定再现内容或子剪辑。客户端将在流 URL 中指定筛选器。筛选器可应用到[动态打包](/documentation/articles/media-services-dynamic-packaging-overview/)支持的自适应比特率流协议：HLS、MPEG DASH、平滑流和 HDS。例如：

包含筛选器的 MPEG DASH URL

	http://testendpoint-testaccount.streaming.mediaservices.chinacloudapi.cn/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=mpd-time-csf,filter=MyLocalFilter)

包含筛选器的平滑流 URL

	http://testendpoint-testaccount.streaming.mediaservices.chinacloudapi.cn/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(filter=MyLocalFilter)


有关如何传送内容和构建流 URL 的详细信息，请参阅[传送内容概述](/documentation/articles/media-services-deliver-content-overview/)。


>[AZURE.NOTE]请注意，动态清单不会更改资产和该资产的默认清单。客户端可以选择请求包含或不包含筛选器的流。


###<a id="filters"></a>筛选器 

有两种类型的资产筛选器：

- 全局筛选器（可以应用到 Azure 媒体服务帐户中所有的资产，拥有帐户的生存期）和 
- 本地筛选器（在创建后只能应用到与筛选器关联的资产，拥有资产的生存期）。 

全局和本地筛选器类型具有完全相同的属性。两者的主要差异在于它们更适合哪些方案。全局筛选器通常适用于设备配置文件（再现内容筛选），而本地筛选器可用于修剪特定的资产。


##<a id="scenarios"></a>常见方案 

如前所述，在将内容传送到客户（流式传输实时事件或视频点播）时，你的目标就是：将优质视频传递到处于不同网络条件下的各种设备。此外，你可能在筛选资产与使用**动态清单**方面具有其他的要求。以下部分提供了不同筛选方案的简要概述。

- 仅指定某些设备可以处理的音频和视频再现内容子集（而不是与该资产关联的所有再现内容）。 
- 仅播放视频的某个部分（而不是整个视频）。
- 调整 DVR 演播窗口。

##再现内容筛选 

你可以选择将资产编码成多个编码配置文件（H.264 Baseline、H.264 High、AACL、AACH、Dolby Digital Plus），以及多个优质比特率。不过，并非所有的客户端设备都支持资产的所有配置文件和比特率。例如，早期的 Android 设备只支持 H.264 Baseline+AACL。将较高的比特率发送到不能利用这些优势的设备会浪费带宽及设备计算资源。此类设备必须解码所有给定信息，而目的仅仅是为了缩小信号以便能够显示。

有了动态清单，你可以创建设备配置文件（例如移动配置文件、控制台、HD/SD 等），并包含你想要纳入配置文件中的轨迹与质量。

 
![再现内容筛选示例][renditions2]

以下示例使用编码器将夹层资产编码成七个 ISO MP4 视频再现内容（从 180p 到 1080p）。编码的资产可以动态打包成以下任一流协议：HLS、平滑流、MPEG DASH 和 HDS。图表顶部显示了不包含筛选器的资产的 HLS 清单（包含全部七个再现内容）。左下角显示名为“ott”的筛选器已应用到 HLS 清单。“ott”筛选器指定要删除所有不低于 1Mbps 的比特率，因此将最差的两个质量级别从响应中剥除。右下角显示名为“mobile”的筛选器已应用到 HLS 清单。“mobile”筛选器指定要删除分辨率大于 720p 的再现内容，因此将剥除两个 1080p 再现内容。

![再现内容筛选][renditions1]

##删除语言轨迹

你的资产可能包含多种音频语言，例如英语、西班牙语、法语等。通常，播放器 SDK 管理器会按默认选择音频轨迹，并根据用户的选择来选择可用音频轨迹。开发此类播放器 SDK 相当有挑战性，因为各个设备特定的播放器框架之间需要不同的实现。此外，播放器 API 在某些平台上受到限制，且不包含音频选择功能，因此用户无法选择或更改默认的音频轨迹。使用资产筛选器，可以通过创建只包含所需音频语言的筛选器来控制行为。

![语言轨迹筛选][language_filter]


##修剪资产开头 

在大多数实时流事件中，操作员必须在发生实际事件之前进行某些测试。例如，他们可以在事件开始之前包含如下静态内容：“节目即将开始”。如果节目正在存档，则测试和静态数据也会一并存档并包含在演播中。但是，此信息不应向客户端显示。通过动态清单，你可以创建开始时间筛选器，并从清单中删除不需要的数据。

![剪裁开始][trim_filter]

##基于实时存档创建子剪辑（视图）

许多实时事件的运行时间较长，并且实时存档可能包含多个事件。实时事件结束后，广播者可能想要将实时存档分解成有序的节目启动和停止序列。然后分别发布这些虚拟节目，但不后续处理实时存档，也不创建独立的资产（这将无法利用 CDN 中现有缓存片段的优势）。此类虚拟节目（子剪辑）的示例包括足球或篮球比赛中的上下半场（第几节）、棒球比赛中的局，或者奥运会下午赛程的单项赛事。

借助动态清单，你可以使用开始/结束时间创建筛选器，并基于实时存档创建虚拟视图。

![子剪辑筛选器][subclip_filter]

筛选的资产：

![滑雪][skiing]

##调整演播窗口（DVR）

目前，Azure 媒体服务提供持续时间可设为 5 分钟到 25 小时的循环存档。清单筛选可以基于存档创建循环 DVR 窗口存档，而不会删除媒体。在许多情况下，广播者想要提供受限制的 DVR 窗口，此窗口可随着实时边缘移动，并同时保留更大的存档窗口。广播者可能想要使用超出 DVR 窗口的数据来突出显示剪辑，或者想要为不同的设备提供不同的 DVR 窗口。例如，大多数移动设备不处理大的 DVR 窗口（你可以让移动设备有 2 分钟的 DVR 窗口，桌面客户端有 1 小时的 DVR 窗口）。

![DVR 窗口][dvr_filter]

##调整实时补偿（实时位置）

清单筛选可用于删除实时节目的实时边缘几秒钟的时间。这样，广播者便可以在观众收到流之前（通常倒退 30 秒）先观赏预览发布点的演播，并创建广告插入点。广播者接着可将这些广告及时推送到其客户端框架，以便能够接收与处理信息，然后借机播放广告。

除了广告支持外，实时补偿可用于调整客户端实时下载位置，以便在客户端偏移并命中实时边缘时，仍然可以从服务器获取片段，而不会收到 404 或 412 HTTP 错误。

![livebackoff\_filter][livebackoff_filter]


##将多个规则合并成单个筛选器

你可以将多个筛选规则合并成单个筛选器。例如，你可以定义一个范围规则，以便将静态内容从实时存档中删除，并筛选可用的比特率。对于多个筛选规则而言，最终结果将是这些规则的构成部分（只有交集）。

![多规则][multiple-rules]

##以编程方式创建筛选器

以下主题讨论与筛选器相关的媒体服务实体。该主题还说明如何以编程方式创建筛选器。

[使用 REST API 创建筛选器](/documentation/articles/media-services-rest-dynamic-manifest/)。

## 组合多个筛选器（筛选器组合）

你也可以在一个 URL 中组合多个筛选器。

以下方案演示了为什么你可能需要组合多个筛选器：

1. 你需要筛选视频质量（目的是限制视频质量）以供 Android 或 iPAD 之类的移动设备使用。若要删除其质量不符合需要的视频，可创建一个适合设备配置文件的全局筛选器。如上所述，全局筛选器可以用于你在同一媒体服务帐户下的所有资产，这些资产并没有更多的其他联系。 
2. 你还可以修改资产的开始时间和结束时间。为此，你可以创建一个本地筛选器并设置开始/结束时间。 
3. 你希望能够将这些筛选器组合起来（如果不组合的话，则需要将质量筛选添加到进行修改的筛选器上，这会导致筛选器的使用很困难）。

为了组合筛选器，你需要在清单/播放列表 URL 中设置筛选器名称，用分号对名称进行分隔。假设你有一个名为 *MyMobileDevice* 的筛选器，用于筛选质量，另外还有一个名为 *MyStartTime* 的筛选器，用于设置具体的开始时间。你可以将它们组合成下面这样：

	http://teststreaming.streaming.mediaservices.chinacloudapi.cn/3d56a4d-b71d-489b-854f-1d67c0596966/64ff1f89-b430-43f8-87dd-56c87b7bd9e2.ism/Manifest(filter=MyMobileDevice;MyStartTime)

最多可以组合 3 个筛选器。

有关详细信息，请参阅[此博客](https://azure.microsoft.com/blog/azure-media-services-release-dynamic-manifest-composition-remove-hls-audio-only-track-and-hls-i-frame-track-support/)。


##已知问题和限制

- 动态清单在 GOP 边界（主键帧）内运行，因此修剪后具有精确的 GOP。 
- 可以对本地和全局筛选器使用相同的筛选器名称。请注意，本地筛选器的优先顺序更高，会取代全局筛选器。
- 如果你更新筛选器，则流式处理终结点需要 2 分钟的时间来刷新规则。如果内容是通过使用某些筛选器提供的（并在代理和 CDN 缓存中缓存），则更新这些筛选器会导致播放器失败。建议在更新筛选器之后清除缓存。如果此选项不可用，请考虑使用其他筛选器。



##另请参阅

[将内容传送到客户概述](/documentation/articles/media-services-deliver-content-overview/)

[renditions1]: ./media/media-services-dynamic-manifest-overview/media-services-rendition-filter.png
[renditions2]: ./media/media-services-dynamic-manifest-overview/media-services-rendition-filter2.png

[rendered_subclip]: ./media/media-services-dynamic-manifests/media-services-rendered-subclip.png
[timeline_trim_event]: ./media/media-services-dynamic-manifests/media-services-timeline-trim-event.png
[timeline_trim_subclip]: ./media/media-services-dynamic-manifests/media-services-timeline-trim-subclip.png

[multiple-rules]: ./media/media-services-dynamic-manifest-overview/media-services-multiple-rules-filters.png

[subclip_filter]: ./media/media-services-dynamic-manifest-overview/media-services-subclips-filter.png
[trim_event]: ./media/media-services-dynamic-manifests/media-services-timeline-trim-event.png
[trim_subclip]: ./media/media-services-dynamic-manifests/media-services-timeline-trim-subclip.png
[trim_filter]: ./media/media-services-dynamic-manifest-overview/media-services-trim-filter.png
[redered_subclip]: ./media/media-services-dynamic-manifests/media-services-rendered-subclip.png
[livebackoff_filter]: ./media/media-services-dynamic-manifest-overview/media-services-livebackoff-filter.png
[language_filter]: ./media/media-services-dynamic-manifest-overview/media-services-language-filter.png
[dvr_filter]: ./media/media-services-dynamic-manifest-overview/media-services-dvr-filter.png
[skiing]: ./media/media-services-dynamic-manifest-overview/media-services-skiing.png
 
<!---HONumber=Mooncake_0613_2016-->