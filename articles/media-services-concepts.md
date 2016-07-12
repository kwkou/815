<properties 
	pageTitle="Azure 媒体服务概念" 
	description="本部分概述 Azure 媒体服务的概念。" 
	services="media-services" 
	documentationCenter="" 
	authors="Juliako" 
	manager="dwrede" 
	editor=""/>

<tags
	ms.service="media-services"
	ms.date="02/25/2016" 
	wacn.date="04/05/2016"/>

#Azure 媒体服务概念 

本部分概述最重要的媒体服务概念。

##<a id="assets"></a>资产和存储

###资产

[资产](https://msdn.microsoft.com/zh-cn/library/azure/hh974277.aspx)包含数字文件（包括视频、音频、图像、缩略图集合、文本轨道和隐藏的解释性字幕文件）以及这些文件的相关元数据。数字文件在上载到资产中后，即可用于媒体服务编码和流式处理工作流。

资产将映射到 Azure Storage 帐户中的 BLOB 容器，资产中的文件则作为该容器中的 BLOB 存储。

确定要将哪些媒体内容上载和存储到资产中时，需注意以下事项：

- 资产应仅包含一个唯一的媒体内容实例。例如，一段电视剧、电影或广告剪辑。
- 资产不应包含多版视听文件或多段视听剪辑。其中一种不当使用资产的示例包括：尝试在资产中存储多段电视剧、广告或某一作品的不同拍摄角度。在资产中存储多版或多段视听文件会对提交编码作业、流式处理和保障资产后续在工作流中的传送造成困难。  

###资产文件 
[AssetFile](https://msdn.microsoft.com/zh-cn/library/azure/hh974275.aspx) 表示 Blob 容器中存储的实际视频或音频文件。一个资产文件始终与一个资产关联，而一个资产则可能包含一个或多个文件。如果资产文件对象未与 BLOB 容器中的数字文件关联，则媒体服务编码器任务将失败。

**AssetFile** 实例和实际媒体文件是两个不同的对象。AssetFile 实例包含有关媒体文件的元数据，而媒体文件包含实际媒体内容。

在不使用媒体服务 API 的情况下，你不应该尝试更改媒体服务生成的 BLOB 容器内容。

###资产加密选项

根据你要上载、存储和传递的内容的不同类型，媒体服务提供了多个加密选项供你选择。

**无**：不使用加密。这是默认值。请注意，使用此选项时，你的内容在传送过程中或静态存储过程中都不会受到保护。

如果计划使用渐进式下载交付 MP4，则使用此选项上载内容。

**StorageEncrypted** - 使用此选项可以通过 AES 256 位加密在本地加密明文内容，然后将其上载到 Azure 存储空间中以加密形式静态存储相关内容。受存储加密保护的资产将在编码前自动解密并放入经过加密的文件系统中，并可选择在重新上载为新的输出资产前重新加密。存储加密的主要用例是在磁盘上通过静态增强加密来保护高品质的输入媒体文件。

若要传送存储加密资产，必须配置资产的传送策略，以使媒体服务了解要如何传送你的内容。在流式传输资产之前，流式处理服务器会删除存储加密，然后再使用指定的传传送策略（例如 AES、PlayReady 或无加密）流式传输你的内容。

**CommonEncryptionProtected** - 如果要采用常用加密或 PlayReady DRM 加密（或上载已加密的）内容（例如，受 PlayReady DRM 保护的平滑流），请使用此选项。

**EnvelopeEncryptionProtected** - 如果要保护（或上载已加密的）采用高级加密标准 (AES) 加密的 HTTP 实时流 (HLS)，请使用此选项。请注意，如果上载已采用 AES 加密的 HLS，则该 HLS 必须已经由 Transform Manager 加密。

###访问策略 

[AccessPolicy](https://msdn.microsoft.com/zh-cn/library/azure/hh974297.aspx) 定义对资产的访问权限（如读取、写入和列出）和持续时间。通常将 AccessPolicy 对象传递给某个定位符，然后使用该定位符来访问资产中包含的文件。


###Blob 容器

一个 Blob 容器包含一组 Blob 集。Blob 容器用作媒体服务中的访问控制分界点和资产上的共享访问签名 (SAS) 定位符。一个 Azure 存储帐户可以包含无数个 Blob 容器。一个容器可以存储无限个 Blob。

>[AZURE.NOTE]在不使用媒体服务 API 的情况下，你不应该尝试更改媒体服务生成的 BLOB 容器内容。

###<a id="locators"></a>定位符

[定位符](https://msdn.microsoft.com/zh-cn/library/azure/hh974308.aspx)提供访问资产中包含的文件的入口点。访问策略用于定义客户端对给定资产具有的访问权限和持续时间。定位符与访问策略的关系可以为多对一的关系，因此，不同定位符可以向不同客户端提供不同的开始时间和连接类型，而全部使用相同的权限和持续时间设置；但是，由于 Azure 存储服务设置的共享访问策略限制，一项给定的资产一次最多只能与五个唯一的定位符相关联。

媒体服务支持两种类型的定位符：OnDemandOrigin 定位符，用于对媒体进行流式处理（例如，MPEG DASH、HLS 或平滑流式处理）；渐进式下载媒体和 SAS URL 定位符，用于与 Azure 存储空间相互上载或下载媒体文件。

请注意，创建 OrDemandOrigin 定位符时，不应使用列表权限 (AccessPermissions.List)。

###存储帐户

对 Azure 存储空间进行的所有访问都要通过存储帐户完成。一个 Media Service 帐户可与一个或多个存储帐户相关联。一个帐户可以包含无限个容器，只要每个帐户的容器总大小不超过 500TB 即可。媒体服务提供 SDK 级工具，可用于管理多个存储帐户，并在上载到这些帐户时基于指标或随机分发使资产分发达到负载平衡。有关详细信息，请参阅[使用 Azure 存储空间](/documentation/services/storage/)。

##作业和任务

[作业](https://msdn.microsoft.com/zh-cn/library/azure/hh974289.aspx)通常用于处理（例如，索引或编码）一个音频/视频演示。如果要处理多个视频，应为要编码的每个视频创建一个作业。

作业包含有关要执行的处理的元数据。每个作业包含一个或多个[任务](https://msdn.microsoft.com/zh-cn/library/azure/hh974286.aspx)，这些任务指定一个原子处理任务、该任务的输入资产和输出资产、一个媒体处理器及其关联的设置。作业中的各个任务可连接在一起，其中一个任务的输出资产指定为下一任务的输入资产。因此，一个作业可以包含播放媒体所必需的全部处理过程。

##<a id="encoding"></a>编码

Azure 媒体服务提供了多个用于在云中对媒体进行编码的选项。

一开始使用媒体服务时，了解编解码器与文件格式之间的区别很重要。编解码器是实现压缩/解压缩算法的软件，而文件格式是用于保存压缩视频的容器。

媒体服务所提供的动态打包可让你以媒体服务支持的流格式（MPEG DASH、HLS、Smooth Streaming、HDS）传送自适应比特率 MP4 或平滑流编码内容，而无须重新打包成这些流格式。

若要使用[动态打包](/documentation/articles/media-services-dynamic-packaging-overview/)，必须执行下列操作：

- 将夹层（源）文件编码成一组自适应比特率 MP4 文件或自适应比特率平滑流文件（本教程稍后将演示编码步骤）。
- 针对你要传送内容的流式处理终结点，获取至少一个按需流式处理单位。有关详细信息，请参阅[如何缩放按需流式处理保留单位](/documentation/articles/media-services-manage-origins/#scale_streaming_endpoints/)。

媒体服务支持将在本文中介绍的以下按需编码器：

- [媒体编码器标准版](/documentation/articles/media-services-encode-asset/#media-encoder-standard)


有关受支持的编码器的信息，请参阅[编码器](/documentation/articles/media-services-encode-asset/)。


##实时流式处理

在 Azure 媒体服务中，通道表示用于处理实时流内容的管道。通道通过以下两种方式之一接收实时输入流：

- 本地实时编码器将多比特率 RTMP 或平滑流（分片 MP4）发送到通道。可以使用以下输出多比特率平滑流的实时编码器：Elemental、Envivio、Cisco。以下实时编码器输出 RTMP：Adobe Flash Live、Telestream Wirecast 和 Tricaster 转码器。引入的流将会直接通过通道，而不会经过任何进一步的处理。收到请求时，媒体服务会将该流传递给客户。

- 将单比特率流（采用以下格式之一：RTP (MPEG-TS)、RTMP 或平滑流（分片 MP4）发送到能够使用媒体服务执行实时编码的通道。然后，频道将对传入的单比特率流执行实时编码，使之转换为多比特率（自适应）视频流。收到请求时，媒体服务会将该流传递给客户。

###通道

在媒体服务中，[通道](https://msdn.microsoft.com/zh-cn/library/azure/dn783458.aspx)负责处理实时流内容。通道提供输入终结点（引入 URL），然后你将该终结点提供给实时转码器。通道从实时转码器接收实时输入流，并通过一个或多个 StreamingEndpoints 使其可用于流式处理。通道还提供可用于预览的预览终结点（预览 URL），并在进一步处理和传递流之前对流进行验证。

可以在创建通道时获取引入 URL 和预览 URL。若要获取这些 URL，通道不一定要处于已启动状态。当你准备好开始将数据从实时转码器推送到通道时，通道必须已启动。实时转码器开始引入数据后，你可以预览流。

每个媒体服务帐户均可包含多个通道、多个节目以及多个 StreamingEndpoint。根据带宽和安全性需求，StreamingEndpoint 服务可专用于一个或多个通道。任何 StreamingEndpoint 都可以从任何通道拉取。


###节目 

[节目](https://msdn.microsoft.com/zh-cn/library/azure/dn783463.aspx)用于控制实时流中的片段的发布和存储。通道管理节目。频道和节目的关系非常类似于传统媒体，频道具有恒定的内容流，而节目的范围限定为该频道上的一些定时事件。
你可以通过设置 **ArchiveWindowLength** 属性，指定你希望保留多少小时的节目录制内容。此值的设置范围是最短 5 分钟，最长 25 小时。

ArchiveWindowLength 还决定了客户端能够从当前实时位置按时间向后搜索的最长时间。超出指定时间长度后，节目也能够运行，但落在时间窗口长度后面的内容将全部被丢弃。此属性的这个值还决定了客户端清单能够增加多长时间。

每个节目都与某个资产关联。若要发布节目，必须为关联的资产创建定位符。创建此定位符后，你可以生成提供给客户端的流 URL。

一个通道最多支持三个并发运行的节目，因此你可以为同一传入流创建多个存档。这样，你便可以根据需要发布和存档事件的不同部分。例如，你的业务要求是存档 6 小时的节目，但只广播过去 10 分钟的内容。为了实现此目的，你需要创建两个同时运行的节目。一个节目设置为存档 6 小时的事件但不发布该节目。另一个节目设置为存档 10 分钟的事件，并且要发布该节目。


有关详细信息，请参阅：

- [使用能够使用 Azure 媒体服务执行实时编码的频道](/documentation/articles/media-services-manage-live-encoder-enabled-channels/)
- [使用从本地编码器接收多比特率实时流的频道](/documentation/articles/media-services-manage-channels-overview/)
- [配额和限制](/documentation/articles/media-services-quotas-and-limitations/)  

##保护内容

###动态加密

使用 Azure 媒体服务，可以在媒体从离开计算机到存储、处理和传送的整个过程中确保其安全。借助媒体服务，你可以传送使用高级加密标准（AES，使用 128 位加密密钥）和通用加密（CENC，使用 PlayReady 和/或 Widevine DRM）进行动态加密的内容。媒体服务还提供了用于向已授权客户端传送 AES 密钥和 PlayReady 许可证的服务。

当前你可以加密以下流格式：HLS、MPEG DASH 和平滑流。无法加密 HDS 流格式或渐进式下载。

如果你需要媒体服务来加密资产，则需要将加密密钥（CommonEncryption 或 EnvelopeEncryption）与资产相关联，并且配置密钥的授权策略。

如果要流式传输存储加密的资产，你必须配置资产的传送策略，以指定你要如何传送资产。

当播放器请求流时，媒体服务将使用指定的密钥通过信封加密（使用 AES）或通用加密（使用 PlayReady 或 Widevine）来动态加密你的内容。为了解密流，播放器将从密钥传送服务请求密钥。为了确定用户是否被授权获取密钥，服务将评估你为密钥指定的授权策略。


###令牌限制

内容密钥授权策略可能受到一种或多种授权限制：开放、令牌限制或 IP 限制。令牌限制策略必须附带由安全令牌服务 (STS) 颁发的令牌。媒体服务支持采用简单 Web 令牌 (SWT) 格式和 JSON Web 令牌 (JWT) 格式的令牌。媒体服务不提供安全令牌服务。你可以创建自定义 STS 或利用 Azure ACS 来颁发令牌。必须将 STS 配置为创建令牌，该令牌使用指定密钥以及你在令牌限制配置中指定的颁发声明进行签名。如果令牌有效，而且令牌中的声明与为密钥（或许可证）配置的声明相匹配，则媒体服务密钥传送服务会将请求的密钥（或许可证）返回到客户端。

在配置令牌限制策略时，必须指定主验证密钥、颁发者和受众参数。主验证密钥包含用来为令牌签名的密钥，颁发者是颁发令牌的安全令牌服务。受众（有时称为范围）描述该令牌的意图，或者令牌授权访问的资源。媒体服务密钥交付服务将验证令牌中的这些值是否与模板中的值匹配。

有关详细信息，请参阅以下文章：

[保护内容概述](/documentation/articles/media-services-content-protection-overview/)
[使用 AES-128 提供保护](/documentation/articles/media-services-protect-with-aes128/)
[使用 DRM 提供保护](/documentation/articles/media-services-protect-with-drm/)

##传送

###<a id="dynamic_packaging"></a>动态打包

在使用媒体服务时，建议你始终将夹层文件编码为自适应比特率 MP4 集，然后使用[动态打包](/documentation/articles/media-services-dynamic-packaging-overview/)将该集转换为所需的格式。


###流式处理终结点

StreamingEndpoint 表示一个流服务，该服务可以直接将内容传递给客户端播放器应用程序，也可以传递给内容交付网络 (CDN) 以进一步分发（Azure 媒体服务现在还提供了 Azure CDN 集成）。 StreamingEndpoint 服务的出站流可以是实时流，也可以是媒体服务帐户中的视频点播资产。此外，还可以通过调整扩展单元（也称为流单元）来控制 StreamingEndpoint 服务处理不断增长的带宽需求的能力。建议为生产环境中的应用程序分配一个或多个扩展单元。缩放单元提供能够以 200 Mbps 为增量购买的专用出口容量和附加功能（当前包括使用动态打包）。

建议使用动态打包和/或动态加密。若要使用这些功能，你必须获取你计划从中流式传输内容的终结点的至少一个流式处理单位。有关详细信息，请参阅[缩放流式处理单位](/documentation/articles/media-services-manage-origins/#scale_streaming_endpoints)。

默认情况下，每个媒体服务帐户最多可以包含 2 个流式处理终结点。若要请求更高的限制，请参阅[配额和限制](/documentation/articles/media-services-quotas-and-limitations/)。

仅当 StreamingEndpoint 处于运行状态时才进行计费。

###资产传送策略

媒体服务内容传送工作流中的步骤之一是配置[资产传送策略](https://msdn.microsoft.com/zh-cn/library/azure/dn799055.aspx)，以便对其进行流式传输。资产传送策略告知媒体服务你希望如何传送资产：应该将资产动态打包成哪种流式处理协议（例如 MPEG DASH、HLS、平滑流或全部），是否要动态加密资产以及如何加密（信封或常用加密）。

如果你有存储加密的资产，在流式传输资产之前，流式处理服务器会删除存储加密，然后再使用指定的传送策略流式传输你的内容。例如，若要传送使用高级加密标准 (AES) 加密密钥加密的资产，请将策略类型设为 DynamicEnvelopeEncryption。若要删除存储加密并以明文的形式流式传输资产，请将策略类型设为 NoDynamicEncryption。

###渐进式下载 

渐进式下载可让你在下载完整个文件之前开始播放媒体。你只能渐进式下载 MP4 文件。

请注意，如果希望已加密的资产可用于渐进式下载，则必须将这些资产解密。

若要为用户提供渐进式下载 URL，必须先创建一个 OnDemandOrigin 定位符。创建定位符可以生成资产的基本路径。然后，你需要追加 MP4 文件的名称。例如：

	http://amstest1.streaming.mediaservices.chinacloudapi.cn/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny_H264_650kbps_AAC_und_ch2_96kbps.mp4

###流 URL

将内容流式传输到客户端。若要为用户提供流 URL，必须先创建一个 OnDemandOrigin 定位符。通过创建定位符，将为你提供包含要流式传输的内容的资产的基本路径。但是，为了能够流式传输该内容，你需要进一步修改此路径。若要构造流清单文件的完整 URL，你必须将定位符的 Path 值与清单 (filename.ism) 文件名连接起来。然后，向定位符路径追加 /Manifest 和相应的格式（如果需要）。

你也可以通过 SSL 连接流式传输内容。为此，请确保流 URL 以 HTTPS 开头。

请注意，仅当你要从中传送内容的流式处理终结点是在 2014 年 9 月 10 日以后创建的时，才可以通过 SSL 流式传输内容。如果流 URL 基于 9 月 10 日之后创建的流式处理终结点，则 URL 会包含“streaming.mediaservices.chinacloudapi.cn”（新格式）。包含“origin.mediaservices.chinacloudapi.cn”（旧格式）的流 URL 不支持 SSL。如果你的 URL 采用旧格式，并且你希望能够通过 SSL 流式传输内容，请创建新的流式处理终结点。使用基于新流式处理终结点创建的 URL 通过 SSL 流式传输你的内容。

以下列表描述了流格式并提供了示例：

- 平滑流

	{流式处理终结点名称-媒体服务帐户名称}.streaming.mediaservices.chinacloudapi.cn/{定位符 ID}/{文件名}.ism/Manifest
		
		http://testendpoint-testaccount.streaming.mediaservices.chinacloudapi.cn/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest


- MPEG DASH

	{流式处理终结点名称-媒体服务帐户名称}.streaming.mediaservices.chinacloudapi.cn/{定位符 ID}/{文件名}.ism/Manifest(format=mpd-time-csf)
 
		http://testendpoint-testaccount.streaming.mediaservices.chinacloudapi.cn/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=mpd-time-csf)



- Apple HTTP 实时流 (HLS) V4

	{流式处理终结点名称-媒体服务帐户名称}.streaming.mediaservices.chinacloudapi.cn/{定位符 ID}/{文件名}.ism/Manifest(format=m3u8-aapl)

		http://testendpoint-testaccount.streaming.mediaservices.chinacloudapi.cn/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=m3u8-aapl)



- Apple HTTP 实时流 (HLS) V3

	{流式处理终结点名称-媒体服务帐户名称}.streaming.mediaservices.chinacloudapi.cn/{定位符 ID}/{文件名}.ism/Manifest(format=m3u8-aapl-v3)

		http://testendpoint-testaccount.streaming.mediaservices.chinacloudapi.cn/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=m3u8-aapl-v3)

- HDS（仅适用于 Adobe PrimeTime/Access 许可证持有人）

	{流式处理终结点名称-媒体服务帐户名称}.streaming.mediaservices.chinacloudapi.cn/{定位符 ID}/{文件名}.ism/Manifest(format=f4m-f4f)

		http://testendpoint-testaccount.streaming.mediaservices.chinacloudapi.cn/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=f4m-f4f) 


<!---HONumber=Mooncake_0328_2016-->