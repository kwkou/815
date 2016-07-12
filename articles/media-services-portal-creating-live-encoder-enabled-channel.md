<properties 
	pageTitle="如何使用 Azure 媒体服务执行实时流式处理以通过 Azure 管理门户创建多比特率流" 
	description="本教程将指导你使用 Azure 管理门户完成创建通道的步骤，该通道接收单比特率实时流，并将其编码为多比特率流。" 
	services="media-services" 
	documentationCenter="" 
	authors="juliako,anilmur" 
	manager="dwrede" 
	editor=""/>

<tags
	ms.service="media-services"
 	ms.date="05/05/2016" 
	wacn.date="06/27/2016"/>


#如何使用 Azure 媒体服务执行实时流式处理以通过 Azure 管理门户创建多比特率流

> [AZURE.SELECTOR]
- [门户](/documentation/articles/media-services-portal-creating-live-encoder-enabled-channel/)
- [.NET](/documentation/articles/media-services-dotnet-creating-live-encoder-enabled-channel/)
- [REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn783458.aspx)

本教程将指导你完成创建**频道**的步骤，该频道接收单比特率实时流，并将其编码为多比特率流。

>[AZURE.NOTE]有关为实时编码启用的通道的更多相关概念信息，请参阅[使用 Azure 媒体服务执行实时流式处理以创建多比特率流](/documentation/articles/media-services-manage-live-encoder-enabled-channels/)。

##常见的实时流方案

以下是在创建常见的实时流应用程序时涉及的常规步骤。

>[AZURE.NOTE] 目前，实时事件的最大建议持续时间为 8 小时。如果你需要运行一个需要更长时间的频道，请通过 Azure.cn 联系 amslived。

1. 将视频摄像机连接到计算机。启动并配置可以通过以下协议之一输出单比特率流的本地实时编码器：RTMP、平滑流式处理或 RTP (MPEG-TS)。有关详细信息，请参阅 [Azure 媒体服务 RTMP 支持和实时编码器](https://azure.microsoft.com/zh-cn/blog/azure-media-services-rtmp-support-and-live-encoders/)。
	
	此步骤也可以在创建频道后执行。

1. 创建并启动频道。

1. 检索频道引入 URL。

	实时编码器使用引入 URL 将流发送到频道。
1. 检索频道预览 URL。 

	使用此 URL 来验证频道是否正常接收实时流。

3. 创建节目（这同时还会创建一个资源）。
1. 发布节目（这将为关联的资源创建按需定位符）。  

	确保你要从中以流形式传输内容的流式传输终结点上至少有一个流式传输保留单元。
1. 在准备好开始流式传输和存档时，启动节目。
2. （可选）可以向实时编码器发信号，以启动广告。将广告插入到输出流中。
1. 在要停止对事件进行流式传输和存档时，停止节目。
1. 删除节目（并选择性地删除资产）。   

##本教程的内容

在本教程中，将使用 Azure 管理门户完成以下任务：

2.  配置流式处理终结点。
3.  创建能够执行实时编码的频道。
1.  获取引入 URL，以便将其提供给实时编码器。实时编码器将使用此 URL 将流引入频道。
1.  创建节目（和资产）
1.  发布资产并获取流 URL  
1.  播放内容 
2.  清理

##先决条件
以下是完成本教程所需具备的条件。

- 若要完成本教程，你需要一个 Azure 帐户。如果你没有帐户，可以创建一个试用帐户，只需几分钟即可完成。有关详细信息，请参阅 [Azure 试用](/pricing/1rmb-trial/)。
- 一个媒体服务帐户。若要创建媒体服务帐户，请参阅[创建帐户](/documentation/articles/media-services-create-account/)。
- 可以发送单比特率实时流的摄像头和编码器。

##使用门户配置流式处理终结点

使用 Azure 媒体服务时最常见的方案之一是将自适应比特率流传送至你的客户端。通过自适应比特率流，客户端可以在视频显示时，根据当前网络带宽、CPU 利用率和其他因素，切换至较高或较低的比特率流。媒体服务支持以下自适应比特率流式处理技术：HTTP 实时流式处理 (HLS)、平滑流式处理、MPEG DASH 和 HDS（仅适用于 Adobe PrimeTime/Access 许可证持有人）。

使用实时流时，本地实时编码器（在本例中为 Wirecast）会将多比特率实时流引入你的通道。当用户请求流时，媒体服务会使用动态打包将源流重新打包成自适应比特率流（HLS、DASH 或平滑流）。

若要利用动态打包，你需要获取计划从中传送内容的**流式处理终结点**的至少一个流式处理单元。

若要更改流式处理保留单元数，请执行以下操作：

1. 在 [Azure 管理门户](https://manage.windowsazure.cn/)中单击“媒体服务”。然后，单击媒体服务的名称。

2. 选择“流式处理终结点”页。然后，单击要修改的流式处理终结点。

3. 若要指定流式处理单元数，请选择“缩放”选项卡并移动**“保留容量”**滑块。

	![“缩放”页](./media/media-services-portal-creating-live-encoder-enabled-channel/media-services-origin-scale.png)

4. 按“保存”按钮保存更改。

	分配所有新的单元大约需要 20 分钟才能完成。

	 
	>[AZURE.NOTE] 当前，将流式处理单位的任何正值设置回“无”可将流式处理功能禁用最多 1 小时。
	>
	> 为 24 小时期间指定的最大单位数将用于计算成本。有关定价详细信息，请参阅[媒体服务定价详细信息](/home/features/media-services/pricing/)。

 
##<a id="create-a-channel"></a>创建频道

1.	在 [Azure 管理门户](http://manage.windowsazure.cn/)中，单击“媒体服务”，然后单击媒体服务帐户名。
2.	选择“频道”页。
3.	选择“添加+”以添加新频道。

选择**“标准”**编码类型。此类型指定要创建能够进行实时编码的频道。这意味着传入单比特率流将发送到频道，并使用指定的实时编码器设置编码为多比特率流。有关详细信息，请参阅[使用 Azure 媒体服务执行实时流式处理以创建多比特率流](/documentation/articles/media-services-manage-live-encoder-enabled-channels/)。

![standard0][standard0]

对于**“标准”**编码类型，有效引入协议选项包括：

- 单比特率分片 MP4（平滑流）
- 单比特率 RTMP
- RTP (MPEG-TS)：RTP 上的 MPEG-2 传输流。

有关每个协议的详细说明，请参阅[使用 Azure 媒体服务执行实时流式处理以创建多比特率流](/documentation/articles/media-services-manage-live-encoder-enabled-channels/)。

![standard1][standard1]

当频道或其关联的节目正在运行时，无法更改输入协议。如果你需要不同的协议，应当针对每个输入协议创建单独的频道。

在**“广告配置”**页上，可以指定 Ad 标记信号源。使用门户时，只能选择“API”，它指示频道内的实时编码器应侦听异步 Ad 标记 API。使用门户时，只能选择“API”。

有关详细信息，请参阅[使用 Azure 媒体服务执行实时流式处理以创建多比特率流](/documentation/articles/media-services-manage-live-encoder-enabled-channels/)。

![standard2][standard2]

在**“编码预设”**页上，可以选择系统预设。目前，唯一可以选择的系统预设是**“默认 720p”**。

![standard3][standard3]

在**“创建频道”**页上，可以定义允许将视频发布到此频道的 IP 地址。允许的 IP 地址可以指定为单个 IP 地址（例如“10.0.0.1”）、使用一个 IP 地址和 CIDR 子网掩码的 IP 范围（例如“10.0.0.1/22”），或使用一个 IP 地址和点分十进制子网掩码的 IP 范围（例如“10.0.0.1(255.255.252.0)”）。

如果未指定 IP 地址并且没有规则定义，则不会允许任何 IP 地址。若要允许任何 IP 地址，请创建一个规则并设置 0.0.0.0/0。


![standard4][standard4]

>[AZURE.NOTE] 目前，通道启动可能最多需要 30 分钟。频道重置可能最多需要 5 分钟。

创建频道后，可以选择**“编码器”**选项卡，从中可以查看频道配置。此外，还可以管理广告和清单。

![standard5][standard5]

有关详细信息，请参阅[使用 Azure 媒体服务执行实时流式处理以创建多比特率流](/documentation/articles/media-services-manage-live-encoder-enabled-channels/)。


##获取引入 URL

创建通道后，你可以获得要提供给实时编码器的引入 URL。编码器将使用这些 URL 来输入实时流。

![readychannel](./media/media-services-portal-creating-live-encoder-enabled-channel/media-services-ready-channel.png)


![ingesturls](./media/media-services-portal-creating-live-encoder-enabled-channel/media-services-ingest-urls.png)


##<a id="create-and-manage-a-program"></a>创建和管理节目

###概述

频道与节目相关联，使用节目，你可以控制实时流中的段的发布和存储。通道管理节目。频道和节目的关系非常类似于传统媒体，频道具有恒定的内容流，而节目的范围限定为该频道上的一些定时事件。

可以通过设置**存档窗口**长度，指定你希望保留节目录制内容的小时数。此值的设置范围是最短 5 分钟，最长 25 小时。存储时间窗口长度还决定了客户端能够从当前实时位置按时间向后搜索的最长时间。超出指定时间长度后，节目也能够运行，但落在时间窗口长度后面的内容将全部被丢弃。此属性的这个值还决定了客户端清单能够增加多长时间。

每个节目都与某个资产关联。若要发布节目，必须为关联的资产创建按需定位符。创建此定位符后，你可以生成提供给客户端的流 URL。

一个通道最多支持三个并发运行的节目，因此你可以为同一传入流创建多个存档。这样，你便可以根据需要发布和存档事件的不同部分。例如，你的业务要求是存档 6 小时的节目，但只广播过去 10 分钟的内容。为了实现此目的，你需要创建两个同时运行的节目。一个节目设置为存档 6 小时的事件但不发布该节目。另一个节目设置为存档 10 分钟的事件，并且要发布该节目。

不应当将现有节目重用于新事件。而是应当为每个事件创建并启动一个新节目，如“对实时流式传输应用程序进行编程”部分中所述。

在准备好开始流式传输和存档时，启动节目。在要停止对事件进行流式传输和存档时，停止节目。

若要删除存档的内容，请停止并删除节目，然后删除关联的资产。如果资产被某个节目使用，则无法将其删除，必须先删除该节目。

即使你停止并删除了节目，只要你没有删除资产，用户也将能够按需将你的已存档内容作为视频进行流式传输。

如果希望保留已存档的内容但不希望其可供流式传输，请删除流式传输定位符。

###创建/启动/停止节目

将流传输到通道后，你可以通过创建资产、节目和流定位符来启动流式传输事件。这将会存档流，并使观看者可通过流式处理终结点使用该流。

可通过两种方式启动该事件：

1. 在**“频道”**页上，按**“添加”**以添加新节目。

	指定：节目名称、资产名称、存档时段和加密选项。
	
	![createprogram](./media/media-services-portal-creating-live-encoder-enabled-channel/media-services-create-program.png)
	
	如果保留选中**“立即发布此节目”**，将会创建节目的发布 URL。
	
	在准备好流式传输节目时，可以按**“开始”**。

	启动节目后，你可以按“播放”开始播放内容。


	![createdprogram](./media/media-services-portal-creating-live-encoder-enabled-channel/media-services-created-program.png)

2. 或者，可以使用快捷方式或按**“频道”**页上的**“开始流式传输”**按钮。这将会创建资产、节目和流定位符。

	节目命名为 DefaultProgram，存档时段设置为 1 小时。

	你可以从“通道”页播放发布的节目。

	![channelpublish](./media/media-services-portal-creating-live-encoder-enabled-channel/media-services-channel-play.png)


如果你在**“频道”**页上单击**“停止流式传输”**，将会停止并删除默认节目。资源仍在那里，并且你可以从**“内容”**页发布或取消发布它。

如果你切换到**“内容”**页，你将看到为节目创建的资源。

![contentasset](./media/media-services-portal-creating-live-encoder-enabled-channel/media-services-content-assets.png)


##播放内容

若要为用户提供一个可用来流式传输内容的 URL，你首先需要通过创建定位符（当你使用门户发布资源时，系统已经为你创建了定位符）来“发布”资源（如上一部分中所述）。定位符提供对资产中所含文件的访问权限。

根据你要用来播放内容的流式处理协议，你可能需要修改从频道\\节目的**“发布 URL”**链接获取的 URL。

动态打包负责将实时流打包成指定的协议。

默认情况下，流 URL 采用以下格式，你可以用它来播放平滑流资产：

	{streaming endpoint name-media services account name}.streaming.mediaservices.chinacloudapi.cn/{locator ID}/{filename}.ism/Manifest

若要生成 HLS 流 URL，请将 (format=m3u8-aapl) 附加到 URL。

	{streaming endpoint name-media services account name}.streaming.mediaservices.chinacloudapi.cn/{locator ID}/{filename}.ism/Manifest(format=m3u8-aapl)

若要生成 MPEG DASH 流 URL，请将 (format=mpd-time-csf) 追加到 URL。

	{streaming endpoint name-media services account name}.streaming.mediaservices.chinacloudapi.cn/{locator ID}/{filename}.ism/Manifest(format=mpd-time-csf)

有关传送内容的详细信息，请参阅[传送内容](/documentation/articles/media-services-deliver-content-overview/)。

你可以使用 [AMS 播放器](http://amsplayer.azurewebsites.net/azuremediaplayer.html)播放平滑流，或使用 iOS 和 Android 设备播放 HLS 版本 3。

##清理

如果你已完成流式处理事件，并想要清理先前设置的资源，请遵循以下过程。

- 停止从编码器推送流。
- 停止通道。停止通道后，将不会产生任何费用。当你需要重新启动它时，它将采用相同的引入 URL，因此你无需重新配置编码器。
- 除非你想要继续以点播流形式提供实时事件的存档，否则你可以停止流式处理终结点。如果通道处于停止状态，将不会产生任何费用。
  

##注意事项

- 目前，实时事件的最大建议持续时间为 8 小时。如果你需要运行一个需要更长时间的频道，请通过 Azure.cn 联系 amslived。
- 确保你要从中以流形式传输内容的流式传输终结点上至少有一个流式传输保留单元。


[standard0]: ./media/media-services-portal-creating-live-encoder-enabled-channel/media-services-create-channel-standard0.png
[standard1]: ./media/media-services-portal-creating-live-encoder-enabled-channel/media-services-create-channel-standard1.png
[standard2]: ./media/media-services-portal-creating-live-encoder-enabled-channel/media-services-create-channel-standard2.png
[standard3]: ./media/media-services-portal-creating-live-encoder-enabled-channel/media-services-create-channel-standard3.png
[standard4]: ./media/media-services-portal-creating-live-encoder-enabled-channel/media-services-create-channel-standard4.png
[standard5]: ./media/media-services-portal-creating-live-encoder-enabled-channel/media-services-create-channel-standard_encode.png
<!---HONumber=Mooncake_0620_2016-->