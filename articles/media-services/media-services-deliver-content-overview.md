<properties 
	pageTitle="将内容传送到客户概述" 
	description="本主题概述了在使用 Azure 媒体服务传送内容时涉及到的操作。" 
	services="media-services" 
	documentationCenter="" 
	authors="Juliako" 
	manager="dwrede" 
	editor=""/>

<tags
	ms.service="media-services"
	ms.date="02/22/2016"
	wacn.date="04/05/2016"/>


#将内容传送到客户概述

##概述

在将内容传送到客户（流式传输实时事件或视频点播）时，你的目标就是：将优质视频传递到处于不同网络条件下的各种设备。

要实现此目标：

- 将你的流编码成多比特率（自适应比特率）视频流（这将会负责处理质量和网络条件），并 
- 使用媒体服务[动态打包](/documentation/articles/media-services-dynamic-packaging-overview/)将你的流动态地重新打包成不同的协议（这将会负责不同设备上的流式处理）。媒体服务支持传送以下自适应比特率流式处理技术：HTTP 实时流式处理 (HLS)、平滑流式处理、MPEG DASH 和 HDS（仅适用于 Adobe PrimeTime/Access 许可证持有人）。

本主题概述了重要的内容传送概念。


##动态打包


媒体服务所提供的动态打包可让你以媒体服务支持的流格式（MPEG DASH、HLS、Smooth Streaming、HDS）传送自适应比特率 MP4 或平滑流编码内容，而无须重新打包成这些流格式。建议使用动态打包来传送内容。

若要使用动态打包，必须执行下列操作：

- 将夹层（源）文件编码成一组自适应比特率 MP4 文件或自适应比特率平滑流文件。
- 针对你要传送内容的流式处理终结点，获取至少一个按需流式处理单位。有关详细信息，请参阅[如何缩放按需流式处理保留单位](/documentation/articles/media-services-manage-origins/#scale_streaming_endpoints)。 

通过动态打包，你只需要存储及支付一种存储格式的文件，媒体服务将会根据客户端的要求创建并提供适当的响应。

请注意，除了能够使用动态打包功能以外，点播流保留单元也为你提供可购买的专用流出容量（以 200 Mbps 为增量来购买）。默认情况下，点播流在共享实例模型中配置，该模型的服务器资源（例如计算机、出口容量等）与所有其他用户共享。若要增加按需流式处理吞吐量，建议购买按需流式处理保留单位。

有关详细信息，请参阅[动态打包](/documentation/articles/media-services-dynamic-packaging-overview/)。

##筛选器和动态清单

媒体服务可让你为资产定义筛选器。这些筛选器是服务器端规则，可让你的客户选择运行如下操作：只播放一段视频（而非播放完整视频），或只指定客户设备可以处理的一部分音频和视频再现内容（而非与该资产相关的所有再现内容）。通过按客户请求创建的**动态清单**可以实现对资产进行这种筛选，并基于指定的筛选器流式传输视频。

有关详细信息，请参阅[筛选器和动态清单](/documentation/articles/media-services-dynamic-manifest-overview/)。

##定位符

若要为用户提供一个可用来流式传输内容或下载内容的 URL，你首先需要通过创建定位符来“发布”资产。定位符提供访问资产中包含的文件的入口点。媒体服务支持两种类型的定位符：

- 用于流媒体（例如 MPEG DASH、HLS 或平滑流）或逐进式下载文件的 **OnDemandOrigin** 定位符。
-  用于将媒体文件下载到本地计算机的 **SAS**（访问签名）URL 定位符。 

**访问策略**用于定义客户端可以访问给定资产的权限（如读取、写入和列出）和持续时间。请注意，创建 OrDemandOrigin 定位符时，不应使用列表权限 (AccessPermissions.List)。

定位符附带过期日期。当你使用门户发布资产时，将会创建过期日期在 100 年后的定位符。

>[AZURE.NOTE] 如果你使用门户在 2015 年 3 月之前创建了定位符，则会创建过期日期在两年后的定位符。

若要更新定位符的过期日期，请使用 [REST](http://msdn.microsoft.com/zh-cn/library/azure/hh974308.aspx#update_a_locator) 或 [.NET] (https://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.mediaservices.client.ilocator.update(v=azure.10).aspx) API。请注意，当你更新 SAS 定位符的过期日期时，URL 会发生变化。
 
定位符不用于管理按用户的访问控制。要为不同用户提供不同的访问权限，请使用数字权限管理 (DRM) 解决方案。有关详细信息，请参阅[保护媒体](http://msdn.microsoft.com/zh-cn/library/azure/dn282272.aspx)。

请注意，当你创建定位符时，可能会由于 Azure 存储空间中所需存储和传播过程的影响，出现 30 秒的延迟。


##自适应流 

自适应比特率技术允许视频播放器应用程序确定网络条件并从多个比特率中进行选择。当网络通信质量下降时，客户端可以选择较低的比特率，从而让播放器能够继续以较低的视频质量播放视频。当网络条件改善时，客户端可以切换到较高的比特率，以改进视频质量。Azure 媒体服务支持以下自适应比特率技术：HTTP 实时流式处理 (HLS)、平滑流式处理、MPEG DASH 和 HDS。

若要为用户提供流 URL，必须先创建一个 OnDemandOrigin 定位符。通过创建定位符，将为你提供包含要流式传输的内容的资产的基本路径。但是，为了能够流式传输该内容，你需要进一步修改此路径。若要构造流清单文件的完整 URL，你必须将定位符的 Path 值与清单 (filename.ism) 文件名连接起来。然后，向定位符路径追加 /Manifest 和相应的格式（如果需要）。

>[AZURE.NOTE]你也可以通过 SSL 连接流式传输内容。为此，请确保流 URL 以 HTTPS 开头。

请注意，仅当你要从中传送内容的流式处理终结点是在 2014 年 9 月 10 日以后创建的时，才可以通过 SSL 流式传输内容。如果流 URL 基于 9 月 10 日之后创建的流式处理终结点，则 URL 会包含“streaming.mediaservices.chinacloudapi.cn”（新格式）。包含“origin.mediaservices.chinacloudapi.cn”（旧格式）的流 URL 不支持 SSL。如果你的 URL 采用旧格式，并且你希望能够通过 SSL 流式传输内容，请创建新的流式处理终结点。使用基于新流式处理终结点创建的 URL 通过 SSL 流式传输你的内容。


##流 URL 格式

**MPEG DASH 格式**

{流式处理终结点名称-媒体服务帐户名称}.streaming.mediaservices.chinacloudapi.cn/{定位符 ID}/{文件名}.ism/Manifest(format=mpd-time-csf)

示例

	http://testendpoint-testaccount.streaming.mediaservices.chinacloudapi.cn/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=mpd-time-csf)



**Apple HTTP 实时流 (HLS) V4 格式**

{流式处理终结点名称-媒体服务帐户名称}.streaming.mediaservices.chinacloudapi.cn/{定位符 ID}/{文件名}.ism/Manifest(format=m3u8-aapl)

	http://testendpoint-testaccount.streaming.mediaservices.chinacloudapi.cn/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=m3u8-aapl)

**Apple HTTP 实时流 (HLS) V3 格式**

{流式处理终结点名称-媒体服务帐户名称}.streaming.mediaservices.chinacloudapi.cn/{定位符 ID}/{文件名}.ism/Manifest(format=m3u8-aapl-v3)
	
	http://testendpoint-testaccount.streaming.mediaservices.chinacloudapi.cn/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=m3u8-aapl-v3)

**Apple HTTP 实时流 (HLS) 格式，带“仅音频”筛选器**

默认情况下，仅音频轨道已包括在 HLS 清单中。这是针对手机网络进行 Apple 应用商店认证所必需的。在这种情况下，如果客户端没有足够的带宽，或者是通过 2G 连接进行的连接，则会切换成“仅音频”播放。这样可以在不进行缓冲的情况下保持流式播放的连续性，但缺点是没有视频。但在某些情况下，相当于仅播放音频来说，用户更愿意选择缓冲播放视频。如果你希望删除“仅音频”轨道，可在 URL 中添加 (audio-only=false) 将其删除。

	http://testendpoint-testaccount.streaming.mediaservices.chinacloudapi.cn/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=m3u8-aapl-v3,audio-only=false)

有关详细信息，请参阅[此博客](http://azure.microsoft.com/blog/azure-media-services-release-dynamic-manifest-composition-remove-hls-audio-only-track-and-hls-i-frame-track-support)。


**平滑流格式**

{流式处理终结点名称-媒体服务帐户名称}.streaming.mediaservices.chinacloudapi.cn/{定位符 ID}/{文件名}.ism/Manifest

示例：

	http://testendpoint-testaccount.streaming.mediaservices.chinacloudapi.cn/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest

**平滑流 2.0 清单（旧清单）**

默认情况下，平滑流清单格式包含重复标记（r 标记）。但是，一些播放器不支持 r 标记。此类客户端可以使用禁用 r 标记的格式：

{流式处理终结点名称-媒体服务帐户名称}.streaming.mediaservices.chinacloudapi.cn/{定位符 ID}/{文件名}.ism/Manifest(format=fmp4-v20)

	http://testendpoint-testaccount.streaming.mediaservices.chinacloudapi.cn/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=fmp4-v20)

**HDS（仅适用于 Adobe PrimeTime/Access 许可证持有人）**

{流式处理终结点名称-媒体服务帐户名称}.streaming.mediaservices.chinacloudapi.cn/{定位符 ID}/{文件名}.ism/Manifest(format=f4m-f4f)

	http://testendpoint-testaccount.streaming.mediaservices.chinacloudapi.cn/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=f4m-f4f)


##渐进式下载 

渐进式下载可让你在下载完整个文件之前开始播放媒体。你无法渐进式下载 .ism*（ismv、isma、ismt、ismc）文件。

若要渐进式下载内容，请使用 OnDemandOrigin 类型的定位符。以下示例演示了基于 OnDemandOrigin 类型的定位符的 URL：

	http://amstest1.streaming.mediaservices.chinacloudapi.cn/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny_H264_650kbps_AAC_und_ch2_96kbps.mp4

请注意以下事项：

- 你必须解密你希望从源服务进行流式处理的任何存储加密的资产，然后才能进行渐进式下载。


##下载

若要下载内容，必须创建 SAS 定位符。使用 SAS 定位符可以访问文件所在的 Azure 存储空间容器。若要构建下载 URL，必须将文件名嵌入到主机和 SAS 签名之间。

以下示例演示了基于 SAS 定位符的 URL：

	https://test001.blob.core.chinacloudapi.cn/asset-ca7a4c3f-9eb5-4fd8-a898-459cb17761bd/BigBuckBunny.mp4?sv=2012-02-12&se=2014-05-03T01%3A23%3A50Z&sr=c&si=7c093e7c-7dab-45b4-beb4-2bfdff764bb5&sig=msEHP90c6JHXEOtTyIWqD7xio91GtVg0UIzjdpFscHk%3D

请注意以下事项：

- 你必须解密你希望从源服务进行流式处理的任何存储加密的资产，然后才能进行渐进式下载。
- 未在 12 小时内完成的下载将会失败。



##流式处理终结点

**流式处理终结点**表示一个流服务，该服务可以直接将内容传递给客户端播放器应用程序，也可以传递给内容传送网络 (CDN) 以进一步分发。流式传输终结点服务的出站流可以是实时流，也可以是你的媒体服务帐户中的视频点播资产。此外，还可以通过调整流式传输保留单元来控制流式处理终结点服务处理不断增长的带宽需求的能力。你至少应该为生产环境中的应用程序分配一个保留单元。有关详细信息，请参阅[如何缩放媒体服务](/documentation/articles/media-services-manage-origins/#scale_streaming_endpoints)。



##相关主题

[轮转存储密钥后更新媒体服务定位符](/documentation/articles/media-services-roll-storage-access-keys/)
 

<!---HONumber=Mooncake_0328_2016-->