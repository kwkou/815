<properties 
	pageTitle="使用能够使用 Azure 媒体服务执行实时编码的频道" 
	description="本主题介绍如何设置频道，以从本地编码器接收单比特率实时流，然后使用媒体服务执行实时编码以将其转换为自适应比特率流。然后，该流可以使用以下自适应流式处理协议之一通过一个或多个流式处理终结点传递给客户端播放应用程序：HLS、平滑流、MPEG DASH、HDS。" 
	services="media-services" 
	documentationCenter="" 
	authors="Juliako" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="media-services" 
	ms.date="09/16/2015"
	wacn.date="11/02/2015"/>

#使用能够使用 Azure 媒体服务（预览版）执行实时编码的频道

##概述

在 Azure 媒体服务中，**频道**表示用于处理实时流内容的管道。**频道**通过以下两种方式之一接收实时输入流：

- 本地实时编码器将多比特率 **RTMP** 或 **平滑流**（分片 MP4）发送到频道。可以使用以下输出多比特率平滑流的实时编码器：Elemental、Envivio、Cisco。以下实时编码器输出 RTMP：Adobe Flash Live、Telestream Wirecast 和 Tricaster 转码器。引入流将通过**频道**，而不会进行任何进一步处理。收到请求时，媒体服务会将该流传递给客户。
- 将单比特率流（采用以下格式之一：**RTP** (MPEG-TS)、**RTMP** 或**平滑流**（分片 MP4）发送到能够使用媒体服务执行实时编码的**频道**。然后，**频道**将对传入的单比特率流执行实时编码，使之转换为多比特率（自适应）视频流。收到请求时，媒体服务会将该流传递给客户。 

	使用媒体服务对实时流进行编码的功能当前处于**预览**状态。

从媒体服务 2.10 发行版开始，在创建频道时，可以指定你想要频道接收输入流的方式，以及是否想要频道对流执行实时编码。可以使用两个选项：

- **无** - 如果你打算使用输出多比特率流的本地实时编码器，请指定此值。在这种情况下，传入流将传递到输出，而不会进行任何编码。这是 2.10 发行版以前的频道行为。有关使用此类型的频道的更详细信息，请参阅[使用从本地编码器接收多比特率实时流的频道](/documentation/articles/media-services-manage-channels-overview)。

- **标准**（预览版）- 如果你打算使用媒体服务将单比特率实时流编码为多比特率流，请选择此值。

	使用媒体服务对实时流进行编码的功能当前处于预览状态。

>[AZURE.NOTE]本主题讨论为执行实时编码（“标准”编码类型）启用的频道的属性。有关使用未为执行实时编码启用的频道的信息，请参阅[使用从本地编码器接收多比特率实时流的频道](/documentation/articles/media-services-manage-channels-overview)。

下图表示了实时流式处理工作流，其中频道通过以下协议之一接收单比特率流：RTMP、平滑流式处理或 RTP (MPEG-TS)；然后，将该流编码为多比特率流。

![实时工作流][live-overview]



##本主题内容

- 概述[常见的实时流方案](/documentation/articles/media-services-manage-live-encoder-enabled-channels#scenario)
- [频道及其相关组件的说明](/documentation/articles/media-services-manage-live-encoder-enabled-channels#channel)
- [注意事项](/documentation/articles/media-services-manage-live-encoder-enabled-channels#considerations)


##<a id="scenario"></a>常见的实时流方案

以下是在创建常见的实时流应用程序时涉及的常规步骤。

1. 将视频摄像机连接到计算机。启动并配置可以通过以下协议之一输出**单**比特率流的本地实时编码器：RTMP、平滑流式处理或 RTP (MPEG-TS)。有关详细信息，请参阅 [Azure 媒体服务 RTMP 支持和实时编码器](https://azure.microsoft.com/blog/azure-media-services-rtmp-support-and-live-encoders/)。
	
	此步骤也可以在创建频道后执行。

1. 创建并启动频道。

1. 检索频道引入 URL。

	实时编码器使用引入 URL 将流发送到频道。
1. 检索频道预览 URL。 

	使用此 URL 来验证频道是否正常接收实时流。

3. 创建节目。

	使用 Azure 管理门户时，创建节目的同时还会创建资源。

	使用 .NET SDK 或 REST 时，你需要创建一个资源并指定在创建节目时要使用该资源。 
1. 发布与节目关联的资源。   

	确保你要从中以流形式传输内容的流式传输终结点上至少有一个流式传输保留单元。
1. 在准备好开始流式传输和存档时，启动节目。
2. （可选）可以向实时编码器发信号，以启动广告。将广告插入到输出流中。
1. 在要停止对事件进行流式传输和存档时，停止节目。
1. 删除节目（并选择性地删除资产）。   

[实时流式处理任务](/documentation/articles/media-services-manage-channels-overview#tasks)部分将链接到演示如何完成上述任务的主题。


##<a id="channel"></a>频道输入（引入）配置

###<a id="Ingest_Protocols"></a>引入流式传输协议

如果**“编码器类型”**设为**“标准”**，则有效选项为：

- **RTP** (MPEG-TS)：RTP 上的 MPEG-2 传输流。  
- 单比特率 **RTMP**
- 单比特率**分片 MP4**（平滑流）

有关详细信息，请参阅 [Azure 媒体服务 RTMP 支持和实时编码器](https://azure.microsoft.com/blog/azure-media-services-rtmp-support-and-live-encoders/)。

####RTP (MPEG-TS) - RTP 上的 MPEG-2 传输流。  

典型用例：

专业广播者通常使用供应商提供的高端本地实时编码器（如 Elemental Technologies、Ericsson、Ateme、Imagine 或 Envivio）来发送流。通常与 IT 部门和专用网络配合使用。

注意事项：

- 强烈建议使用单节目传输流 (SPTS) 输入。但是，支持使用多语言音频曲目
- 视频流应具有小于 15 Mbps 的平均比特率
- 音频流的聚合平均比特率应小于 1 Mbps
- 以下是支持的编解码器：
	- MPEG-2/H.262 Video 
		
		- Main Profile (4:2:0)
		- High Profile (4:2:0, 4:2:2)
		- 422 Profile (4:2:0, 4:2:2)

	- MPEG-4 AVC/H.264 Video
	
		- Baseline、Main、High Profile（8 位 4:2:0）
		- High 10 Profile（10 位 4:2:0）
		- High 422 Profile（10 位 4:2:2）


	- MPEG-2 AAC-LC Audio
	
		- Mono、Stereo、Surround (5.1, 7.1)
		- MPEG-2 样式 ADTS 打包

	- Dolby Digital (AC-3) Audio

		- Mono、Stereo、Surround (5.1, 7.1)

	- MPEG Audio（层 II 和层 III）
			
		- Mono、Stereo

- 建议的广播编码器包括：
	- Ateme AM2102
	- Ericsson AVP2000
	- eVertz 3480
	- Ericsson RX8200
	- Imagine Communications Selenio ENC 1
	- Imagine Communications Selenio ENC 2
	- AdTec EN-30
	- AdTec EN-91P
	- AdTec EN-100
	- Harmonic ProStream 1000
	- Thor H-2 4HD-EM
	- eVertz 7880 SLKE
	- Cisco Spinnaker
	- Elemental Live

####<a id="single_bitrate_RTMP"></a>单比特率 RTMP

注意事项：

- 传入流不能包含多码率视频
- 视频流应具有小于 15 Mbps 的平均比特率
- 音频流应具有小于 1 Mbps 的平均比特率
- 以下是支持的编解码器：

	- MPEG-4 AVC/H.264 Video  
	
		- Baseline、Main、High Profile（8 位 4:2:0）
		- High 10 Profile（10 位 4:2:0）
		- High 422 Profile（10 位 4:2:2）

	- MPEG-2 AAC-LC Audio

		- Mono、Stereo、Surround (5.1, 7.1)
		- 44\.1 kHz 采样率
		- MPEG-2 样式 ADTS 打包
	
- 建议的编码器包括：

	- Telestream Wirecast
	- Flash 媒体实时编码器
	- Tricaster

####单比特率分片 MP4（平滑流）

典型用例：

使用供应商提供的本地实时编码器（如 Elemental Technologies、Ericsson、Ateme、Envivio）通过开放的 Internet 将输入流发送到附近的 Azure 数据中心。

注意事项：

这同样适用于[单比特率 RTMP](/documentation/articles/media-services-manage-live-encoder-enabled-channels#single_bitrate_RTMP)。

####其他注意事项

- 当频道或其关联的节目正在运行时，无法更改输入协议。如果你需要不同的协议，应当针对每个输入协议创建单独的频道。 
- 传入视频流的最大分辨率为 1920 x 1080，如果隔行扫描，速率最大为 60 个字段/秒，如果渐进式下载，则速度为 30 帧/秒。


###摄取 URL（终结点） 

频道提供你在实时编码器中指定的输入终结点（引入 URL），因此编码器可以将流推送到你的频道。

创建频道后，你可以获得引入 URL。若要获取这些 URL，频道不一定要处于**“正在运行”**状态。当准备好开始将数据推送到频道时，频道必须处于**“正在运行”**状态。在频道开始引入数据后，你可以通过预览 URL 来预览流。

可以选择通过 SSL 连接引入分片 MP4（平滑流）实时流。若要通过 SSL 进行摄取，请确保将摄取 URL 更新为 HTTPS。

###允许的 IP 地址

你可以定义允许向此频道发布视频的 IP 地址。允许的 IP 地址可以指定为单个 IP 地址（例如“10.0.0.1”）、使用一个 IP 地址和 CIDR 子网掩码的 IP 范围（例如“10.0.0.1/22”），或使用一个 IP 地址和点分十进制子网掩码的 IP 范围（例如“10.0.0.1(255.255.252.0)”）。

如果未指定 IP 地址并且没有规则定义，则不会允许任何 IP 地址。若要允许任何 IP 地址，请创建一个规则并设置 0.0.0.0/0。


##频道预览 

###预览 URL

频道还提供了一个预览终结点（预览 URL），你可以使用它在进一步处理和传递流之前预览流并对其进行验证。

你可以在创建频道时获取预览 URL。若要获取该 URL，频道不一定要处于**“正在运行”**状态。

在频道开始摄取数据后，你可以预览流。

**注意**：当前，不管指定了哪种输入类型，都只能以分片 MP4（平滑流）格式来传送预览流。你可以使用 [http://smf.cloudapp.net/healthmonitor](http://smf.cloudapp.net/healthmonitor) 播放器来测试平滑流。你还可以使用 Azure 管理门户中承载的播放器来查看你的流。

###允许的 IP 地址

你可以定义允许连接到预览终结点的 IP 地址。如果未指定 IP 地址，则将允许任何 IP 地址。允许的 IP 地址可以指定为单个 IP 地址（例如“10.0.0.1”）、使用一个 IP 地址和 CIDR 子网掩码的 IP 范围（例如“10.0.0.1/22”），或使用一个 IP 地址和点分十进制子网掩码的 IP 范围（例如“10.0.0.1(255.255.252.0)”）。

##实时编码设置

本部分介绍当频道的**“编码类型”**设为**“标准”**时，如何调整频道内的实时编码器的设置。

###Ad 标记源

你可以指定 Ad 标记信号源。默认值是 **Api**，它指示频道内的实时编码器应侦听异步 **Ad 标记 API**。

另一个有效选项是 **Scte35**（仅当引入流协议设为 RTP (MPEG-TS) 时允许）。当指定了 Scte35 时，实时编码器将分析输入 RTP (MPEG-TS) 流中的 SCTE-35 信号。

###CEA 708 关闭字幕

一个可选的标志，它指示实时编码器忽略传入视频中嵌入的任何 CEA 708 字幕数据。当该标志设为 false（默认值）时，编码器将检测 CEA 708 数据并将该数据重新插入到输出视频流中。

###视频流

可选。描述输入视频流。如果未指定此字段，则使用默认值。仅当输入流协议设为 RTP (MPEG-TS) 时，才允许使用此设置。

####索引

一个从零开始的索引，它指定哪个输入视频流应由频道内的实时编码器处理。仅当引入流协议是 RTP (MPEG-TS) 时，此设置才适用。

默认值为 0。建议在单节目传输流 (SPTS) 中发送。如果输入流包含多个节目，实时编码器将分析输入中的节目映射表 (PMT)，标识流类型名称为 MPEG-2 Video 或 H.264 的输入并以 PMT 中指定的顺序安排这些输入。然后，将使用从零开始的索引选取该安排中的第 n 个条目。

###音频流

可选。描述输入音频流。如果未指定此字段，则将应用指定的默认值。仅当输入流协议设为 RTP (MPEG-TS) 时，才允许使用此设置。

####索引

建议在单节目传输流 (SPTS) 中发送。如果输入流包含多个节目，频道中的实时编码器将分析输入中的节目映射表 (PMT)，标识流类型名称为 MPEG-2 AAC ADTS、AC-3 System-A、AC-3 System-B、MPEG-2 Private PES、MPEG-1 Audio 或 MPEG-2 Audio 的输入并以 PMT 中指定的顺序安排这些输入。然后，将使用从零开始的索引选取该安排中的第 n 个条目。

####语言

音频流的符合 ISO 639-2 的语言标识符，如 ENG。如果不存在，则默认为 UND（未定义）。

如果频道的输入为 MPEG-2 TS over RTP，则最多可以指定 8 个音频流集。但是，不能有两个具有相同索引值的条目。

###<a id="preset"></a>系统预设

指定此频道内的实时编码器要使用的预设。目前，唯一允许的值是 **Default720p**（默认值）。

**Default720p** 会将视频编码为以下 7 层。


####输出视频流

<table border="1">
<tr><th>比特率</th><th>宽度</th><th>高度</th><th>MaxFPS</th><th>配置文件</th><th>输出流名称</th></tr>
<tr><td>3500</td><td>1280</td><td>720</td><td>30</td><td>高</td><td>Video_1280x720_30fps_3500kbps</td></tr>
<tr><td>2200</td><td>960</td><td>540</td><td>30</td><td>主要</td><td>Video_960x540_30fps_2200kbps</td></tr>
<tr><td>1350</td><td>704</td><td>396</td><td>30</td><td>主要</td><td>Video_704x396_30fps_1350kbps</td></tr>
<tr><td>850</td><td>512</td><td>288</td><td>30</td><td>主要</td><td>Video_512x288_30fps_850kbps</td></tr>
<tr><td>550</td><td>384</td><td>216</td><td>30</td><td>主要</td><td>Video_384x216_30fps_550kbps</td></tr>
<tr><td>350</td><td>340</td><td>192</td><td>30</td><td>基线</td><td>Video_340x192_30fps_350kbps</td></tr>
<tr><td>200</td><td>340</td><td>192</td><td>30</td><td>基线</td><td>Video_340x192_30fps_200kbps</td></tr>
</table>

####输出音频流

音频以 64 kbps 的速率编码为立体声 AAC-LC，采样率为 44.1 kHz。

##指示广告

当频道已启用实时编码时，你在管道中有一个组件正在处理视频，并可以操作它。你可以向频道发出信号，以将清单和/或广告插入到传出自适应比特率流中。清单是在某些情况下（例如，在商业广告期间）可用于覆盖输入实时源的静止图像。广告信号是嵌入到传出流中的时间同步信号，用于指示视频播放机执行特殊操作（例如，在适当时间切换到广告）。有关用于此目的的 SCTE-35 信号机制的概述，请参阅此[博客](https://codesequoia.wordpress.com/2014/02/24/understanding-scte-35/)。下面是可以在实时事件中实现的典型方案。

1. 在事件开始前，让观看者获得事件前图像。
1. 在事件结束后，让观看者获得事件后图像。
1. 如果在事件期间出现问题（例如，在体育场中出现电源故障，让观看者获得错误事件图像。
1. 在商业广告期间发送广告时间图像以隐藏实时事件源。

以下是指示广告时可以设置的属性。

###Duration

商业广告的持续时间（以秒为单位）。这必须是非零正值，才能启动商业广告。当商业广告正在播放时，将持续时间设为 0，并且 CueId 与正在播放的商业广告匹配，则广告将被取消。

###CueId

商业广告的唯一 ID，下游应用程序将使用它来执行相应操作。必须是一个正整数。可以将此值设为任意随机正整数，或使用上游系统跟踪提示 ID。在通过 API 提交之前，请确保将任何 ID 规范化为正整数。

###显示清单

可选。指示实时编码器在商业广告期间切换到[默认静态](/documentation/articles/media-services-manage-live-encoder-enabled-channels#default_slate)图像并隐藏传入视频源。在清单期间音频也会静音。默认值为 **false**。
 
所用图像将是在创建频道时通过默认清单资源 ID 属性指定的图像。静态图像将进行拉伸以适合显示图像大小。


##插入清单图像

可以向频道中的实时编码器发出信号，以切换到清单图像。它还可以指示实时编码器结束正在显示的清单。

在某些情况下（例如，在广告期间）可以配置实时编码器以切换到清单图像，并隐藏传入的视频信号。如果未配置此类清单，则在该广告期间将不会屏蔽输入视频。

###Duration

清单的持续时间（以秒为单位）。这必须是非零正值，才能启动清单。如果正在显示清单，并且指定的持续时间为零，则将终止正在显示的清单。

###在 Ad 标记上插入清单

当设为 true 时，此设置会将实时编码器配置为在广告期间插入清单图像。默认值为 true。

###<a id="default_slate"></a>默认静态图像资产 ID

可选。指定媒体服务资源（包含清单图像）的资源 ID。默认值为 null。

**注意**：在创建频道之前，具有以下约束的静态图像应当作为专用资产上载（该资产中不应有其他文件）。

- 分辨率最大为 1920x1080。
- 大小最大为 3 MB。
- 文件名必须具有 *.jpg 扩展名。
- 必须将该图像作为资产中的唯一 AssetFile 上载到资产，此 AssetFile 应标记为主文件。资产不能加密存储。

如果未指定**默认清单资源 ID**，并且**“在 ad 标记上插入清单”**设为 **true**，则将使用默认 Azure 媒体服务图像隐藏输入视频流。在清单期间音频也会静音。



##频道的节目

频道与节目相关联，使用节目，你可以控制实时流中的段的发布和存储。通道管理节目。通道和节目的关系非常类似于传统媒体，通道具有恒定的内容流，而节目的范围限定为该通道上的一些定时事件。

可以通过设置**存档窗口**长度，指定你希望保留节目录制内容的小时数。此值的设置范围是最短 5 分钟，最长 25 小时。存储时间窗口长度还决定了客户端能够从当前实时位置按时间向后搜索的最长时间。超出指定时间长度后，节目也能够运行，但落在时间窗口长度后面的内容将全部被丢弃。此属性的这个值还决定了客户端清单能够增加多长时间。

每个节目都与存储流式处理内容的资源相关联。资产将映射到 Azure Storage 帐户中的 BLOB 容器，资产中的文件则作为该容器中的 BLOB 存储。若要发布节目，以便客户可以查看该流，必须为关联的资源创建按需定位符。创建此定位符后，你可以生成提供给客户端的流 URL。

一个频道最多支持三个同时运行的节目，因此你可以为同一传入流创建多个存档。这样，你便可以根据需要发布和存档事件的不同部分。例如，你的业务要求是存档 6 小时的节目，但只广播过去 10 分钟的内容。为了实现此目的，你需要创建两个同时运行的节目。一个节目设置为存档 6 小时的事件但不发布该节目。另一个节目设置为存档 10 分钟的事件，并且要发布该节目。

不应当将现有节目重用于新事件。而是应当为每个事件创建并启动一个新节目，如“对实时流式传输应用程序进行编程”部分中所述。

在准备好开始流式传输和存档时，启动节目。在要停止对事件进行流式传输和存档时，停止节目。

若要删除存档的内容，请停止并删除节目，然后删除关联的资产。如果资产被某个节目使用，则无法将其删除，必须先删除该节目。

即使你停止并删除了节目，只要你没有删除资产，用户也将能够按需将你的已存档内容作为视频进行流式传输。

如果希望保留已存档的内容但不希望其可供流式传输，请删除流式传输定位符。


##获取实时源的缩略图预览

启用实时编码后，你现在可以在实时源到达频道时获得实时源的预览。这可以是一个很有用的工具，用于检查实时源是否实际到达频道。

##<a id="states"></a>频道状态，以及状态如何映射到计费模式 

频道的当前状态。可能的值包括：

- **已停止**。这是频道在创建后的初始状态。在此状态下，可以更新频道属性，但不允许进行流式传输。
- **正在启动**。频道正在启动。在此状态下，不允许进行更新或流式传输。如果发生错误，则频道会返回到“已停止”状态。
- **正在运行**。频道能够处理实时流。
- **正在停止**。频道正在停止。在此状态下，不允许进行更新或流式传输。
- **正在删除**。频道正被删除。在此状态下，不允许进行更新或流式传输。

下表显示频道状态如何映射到计费模式。
 
<table border="1">
<tr><th>频道状态</th><th>门户 UI 指示器</th><th>是否计费？</th></tr>
<tr><td>正在启动</td><td>正在启动</td><td>否（暂时状态）</td></tr>
<tr><td>正在运行</td><td>准备就绪（没有正在运行的节目）<br/>或<br/>流式处理（至少有一个正在运行的节目）</td><td>是</td></tr>
<tr><td>正在停止</td><td>正在停止</td><td>否（暂时状态）</td></tr>
<tr><td>已停止</td><td>已停止</td><td>否</td></tr>
</table>


>[AZURE.NOTE]当前处于预览状态，频道启动可能最多需要 20 多分钟。频道重置可能最多需要 5 分钟。



##<a id="Considerations"></a>注意事项

- 当某个编码类型为“标准”的频道出现输入源/贡献源丢失的情况时，该频道会采取相应的补偿措施，将源视频/音频替换为表示错误的静态图像和静音。该频道会持续发出静态图像，直到输入/贡献源恢复。我们建议你不要让实时频道处于此类状态的时间超过 2 小时。如果超出该限制，该频道将无法保证输入重新连接时的行为，也无法保证其响应重置命令时的行为。这种情况下必须停止通道并将其删除，然后创建一个新的。
- 当频道或其关联的节目正在运行时，无法更改输入协议。如果你需要不同的协议，应当针对每个输入协议创建单独的频道。 
- 每次重新配置实时编码器后，请对频道调用**重置**方法。在重置频道之前，必须停止节目。在重置频道后，重新启动节目。 
- 只有当频道处于“正在运行”状态且频道中的所有节目都已停止时才能停止频道。
- 默认情况下，你只能向你的媒体服务帐户添加 5 个频道。这是所有新帐户的软配额。有关详细信息，请参阅[配额和限制](/documentation/articles/media-services-quotas-and-limitations)。
- 当频道或其关联的节目正在运行时，无法更改输入协议。如果你需要不同的协议，应当针对每个输入协议创建单独的频道。
- 仅当你的频道处于**“正在运行”**状态时才会向你收费。有关详细信息，请参阅[此](/documentation/articles/media-services-manage-live-encoder-enabled-channels#states)部分。

##已知问题

- 通道启动可能需要 20 多分钟。
- RTP 支持迎合专业的广播装置。请查看[此](http://azure.microsoft.com/blog/2015/04/13/an-introduction-to-live-encoding-with-azure-media-services/)博客中有关 RTP 的说明。
- 静态图像应符合[此处](/documentation/articles/media-services-manage-live-encoder-enabled-channels#default_slate)所述的限制。如果你想要尝试创建默认静态图像大于 1920x1080 的频道，最终请求将会出错。

###如何创建频道以执行从单比特率到自适应比特率流的实时编码 

选择**“门户”**、**.NET**、**REST API** 以了解如何创建和管理频道和节目。

> [AZURE.SELECTOR]
- [Portal](/documentation/articles/media-services-portal-creating-live-encoder-enabled-channel)
- [.NET SDK](/documentation/articles/media-services-dotnet-creating-live-encoder-enabled-channel)
- [REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn783458.aspx)


##相关主题

[使用 Azure 媒体服务传递实时流式处理事件](/documentation/articles/media-services-live-streaming-workflow)

[媒体服务概念](/documentation/articles/media-services-concepts)

[Azure 媒体服务分片 MP4 实时引入规范](/documentation/articles/media-services-fmp4-live-ingest-overview)

[live-overview]: ./media/media-services-manage-live-encoder-enabled-channels/media-services-live-streaming-new.png
 

<!---HONumber=76-->