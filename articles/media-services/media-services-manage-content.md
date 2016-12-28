<properties 
	pageTitle="如何使用 Azure 经典管理门户管理 Azure 媒体服务的媒体内容" 
	description="了解如何管理 Azure 媒体服务中的媒体内容。包括：上传、编制索引、编码、加密以及发布。" 
	services="media-services" 
	documentationCenter="" 
	authors="Juliako" 
	manager="erikre" 
	editor=""/>  


<tags 
	ms.service="media-services" 
	ms.workload="media" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="08/17/2016"  
	wacn.date="12/27/2016"
	ms.author="juliako"/>  


# 使用 Azure 经典管理门户管理 Azure 媒体服务的内容


本主题说明如何使用 Azure 经典管理门户来管理媒体服务帐户中的媒体内容。

本主题说明如何直接从门户执行以下内容操作：

- 查看内容信息，例如，发布状态、发布的 URL、大小、上次更新的日期时间和资产是否已加密。
- 上传新内容
- 为内容编制索引
- 对内容进行编码
- 加密
- 发布/取消发布内容
- 播放内容


##<a id="upload"></a>如何：上传内容


[AZURE.INCLUDE [media-services-selector-upload-files](../../includes/media-services-selector-upload-files.md)]


1. 在 [Azure 经典管理门户](http://manage.windowsazure.cn)中，单击“媒体服务”，然后单击媒体服务帐户名。
2. 选择“内容”页。 
3. 单击该页上或者门户底部的“上传”按钮。
4. 在“上传内容”对话框中，浏览到所需的资产文件。单击该文件，然后单击“打开”或按 **Enter**。

	![UploadContentDialog][uploadcontent]

5. 在“上传内容”对话框中，单击复选按钮以接受文件和内容名称。
6. 上传随即开始。可以从门户底部跟踪进度。

	![JobStatus][status]

上传完成后，内容列表中会列出新的资产。根据约定，名称的末尾将附加“**-Source**”，以便将新内容作为编码任务的源内容进行跟踪。

![ContentPage][contentpage]

如果在上传过程停止后未更新文件大小值，请按“同步元数据”按钮。这会将资产文件大小与存储中的实际文件大小同步，并刷新“内容”页上的值。

##<a id="index"></a>如何：为内容编制索引

> [AZURE.SELECTOR]
- [.NET](/documentation/articles/media-services-index-content/)
- [门户](/documentation/articles/media-services-manage-content/#index)

使用 Azure 媒体索引器，可以使媒体文件内容可供搜索，并为隐藏的字幕和关键字生成全文本脚本。可根据下面所示的步骤，使用 Azure 经典管理门户为内容编制索引。但是，如果想以更大的力度控制文件和索引作业的完成方式，可以使用适用于 .NET 的媒体服务 SDK 或 REST API。有关详细信息，请参阅[使用 Azure 媒体索引器为媒体文件编制索引](/documentation/articles/media-services-index-content/)。

下面的步骤演示如何使用 Azure 经典管理门户为内容编制索引。

1. 选择要编制索引的文件。

	如果此文件类型支持索引，则“内容”页底部将启用“处理”按钮。
	
1. 按“处理”按钮。
2. 在“处理”对话框中，选择“Azure 媒体索引器”处理器。
3. 然后，在“处理”对话框中，填写输入媒体文件的详细**标题**和**说明**信息。

![过程][process]

##<a id="encode"></a>如何：对内容进行编码

> [AZURE.SELECTOR]
- [.NET](/documentation/articles/media-services-dotnet-encode-with-media-encoder-standard/)
- [REST](/documentation/articles/media-services-rest-encode-asset/)
- [门户](/documentation/articles/media-services-manage-content/#encode)

若要通过 Internet 传送数字视频，必须对媒体进行压缩。媒体服务提供了一个 Media Encoder ，可让你指定如何为内容编码（例如，要使用的编解码器、文件格式、分辨率和比特率。）

使用 Azure 媒体服务最常见的方案之一是将自适应比特率流传送至客户端。通过自适应比特率流，客户端可以在视频显示时，根据当前网络带宽、CPU 利用率和其他因素，切换至较高或较低的比特率流。媒体服务支持以下自适应比特率流式处理技术：HTTP 实时流式处理 (HLS)、平滑流式处理、MPEG DASH 和 HDS（仅适用于 Adobe PrimeTime/Access 许可证持有人）。

使用媒体服务提供的动态打包，可采用媒体服务支持的流格式（MPEG DASH、HLS、Smooth Streaming、HDS）传送自适应比特率 MP4 或平滑流式处理编码内容，而无须重新打包成这些流式处理格式。

若要使用动态打包，必须执行下列操作：

- 将夹层（源）文件编码成一组自适应比特率 MP4 文件或自适应比特率平滑流文件（本教程稍后将演示编码步骤）。
- 针对你要传送内容的流式处理终结点，获取至少一个按需流式处理单位。有关详细信息，请参阅[如何缩放按需流式处理保留单元](/documentation/articles/media-services-manage-origins/#scale_streaming_endpoints)。

通过动态打包，只需要存储及支付一种存储格式的文件，媒体服务就会根据客户端的要求创建并提供适当的响应。

除了能够使用动态打包功能以外，按需流式处理保留单元还提供了可按照 200 Mbps 的增量购买的专用出口容量。默认情况下，按需流式处理在共享实例模型中配置，该模型的服务器资源（例如计算、出口容量）与所有其他用户共享。若要增加按需流式处理吞吐量，建议购买按需流式处理保留单元。

本部分介绍通过 Azure 经典管理门户使用媒体编码器标准版为内容编码时可以执行的步骤。

1.  选择要编码的文件。

	如果此文件类型支持编码，则“内容”页底部将启用“处理”按钮。

2. 在“处理”对话框中，选择“媒体编码器标准版”处理器。
3. 选择其中一个“编码配置”。

	![Process2][process2]  



	[媒体编码器标准版的任务预设字符串](https://msdn.microsoft.com/zh-cn/library/mt269960)主题说明了每个预设的含义。

4. 然后，输入所需的友好输出内容名称或接受默认值。然后，单击复选按钮开始编码操作。可在门户底部跟踪进度。
5. 按“确定”。

完成编码后，“内容”页将包含已编码的文件。

若要查看编码作业的进度，请切换到“作业”页。

如果在完成编码后未更新文件大小值，请按“同步元数据”按钮。这会将输出资产文件大小与存储中的实际文件大小同步，并刷新“内容”页上的值。

##<a id="encrypt"></a>如何：对内容进行加密


如果希望媒体服务采用 AES 密钥或 PlayReady DRM 动态加密资产，请确保执行以下操作：

- 将夹层（源）文件编码成一组自适应比特率 MP4 文件或自适应比特率平滑流式处理文件（编码步骤将在[编码](#encode)部分演示）。
- 针对要传送内容的流式处理终结点，获取至少一个按需流式处理单位。有关详细信息，请参阅[如何缩放按需流式处理保留单元](/documentation/articles/media-services-manage-origins/#scale_streaming_endpoints)。
- 配置“默认 aes 明文密钥服务策略”或“默认 playready 许可证服务策略”。有关详细信息，请参阅[配置内容密钥授权策略](/documentation/articles/media-services-portal-configure-content-key-auth-policy/)。


	准备好启用加密后，请按“内容”页底部的“加密”按钮。

	![加密][encrypt]

	启用加密后，每当播放器请求流时，媒体服务将使用指定的密钥通过 AES 或 PlayReady 加密来动态加密你的内容。为了解密流，播放器将从密钥传送服务请求密钥。为了确定用户是否有权获取密钥，服务将评估为密钥指定的授权策略。

另请参阅：

- [使用 PlayReady DRM 进行保护](/documentation/articles/media-services-rest-deliver-streaming-content/)
- [使用 AES-128 密钥进行保护](/documentation/articles/media-services-protect-with-aes128/)

##<a id="publish"></a>如何：发布内容

> [AZURE.SELECTOR]
- [.NET](/documentation/articles/media-services-deliver-streaming-content/)
- [REST](/documentation/articles/media-services-rest-deliver-streaming-content/)
- [门户](/documentation/articles/media-services-manage-content/#publish)

###概述

若要为用户提供可用于流式传输内容或下载内容的 URL，首先需要通过创建定位符来“发布”资产。定位符提供对资产中所含文件的访问权限。媒体服务支持两种类型的定位符：用于流媒体（例如 MPEG DASH、HLS 或平滑流式处理）的 OnDemandOrigin 定位符，以及用于下载媒体文件的访问签名 (SAS) 定位符。

使用 Azure 经典管理门户发布资产时，系统将为你创建定位符并提供基于 OnDemantOrigin 的 URL（如果你的资产包含 .ism 文件）或 SAS URL。

SAS URL 采用以下格式：

	{blob container name}/{asset name}/{file name}/{SAS signature}

流式处理 URL 采用以下格式，可用于播放平滑流资产：

	{streaming endpoint name-media services account name}.streaming.mediaservices.chinacloudapi.cn/{locator ID}/{filename}.ism/Manifest

若要生成 HLS 流式处理 URL，请将 (format=m3u8-aapl) 附加到 URL。

	{streaming endpoint name-media services account name}.streaming.mediaservices.chinacloudapi.cn/{locator ID}/{filename}.ism/Manifest(format=m3u8-aapl)

若要生成 MPEG DASH 流式处理 URL，请将 (format=mpd-time-csf) 追加到 URL。

	{streaming endpoint name-media services account name}.streaming.mediaservices.chinacloudapi.cn/{locator ID}/{filename}.ism/Manifest(format=mpd-time-csf)


附带到期日期的定位符。使用门户发布资产时，会创建到期日期在 100 年以后的定位符。

>[AZURE.NOTE] 如果在 2015 年 3 月之前使用了门户创建定位符，则会创建到期日期在两年以后的定位符。

若要更新定位符的到期日期，请使用 [REST](https://docs.microsoft.com/zh-cn/rest/api/media/operations/locator) 或 [.NET] (https://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.mediaservices.client.ilocator.update(v=azure.10).aspx) API。请注意，更新 SAS 定位符的到期日期时，URL 会发生变化。

###发布

若要使用门户发布资产，请执行以下操作：

1. 选择资源。
2. 然后单击“发布”按钮。
	
 ![PublishedContent][publishedcontent]  



## 如何：从门户播放内容

**Azure 经典管理门户**提供了可用于测试视频的内容播放器。

单击所需的视频，然后单击门户底部的“播放”按钮。
 
请注意以下事项：

- 确保视频已发布。
- **媒体服务内容播放器**从默认的流式处理终结点播放。如果要从非默认流式处理终结点播放，请使用其他播放器。例如 [Azure 媒体服务播放器](http://amsplayer.azurewebsites.net/azuremediaplayer.html)。

![AMSPlayer][AMSPlayer]  


<!-- Images -->

[portaloverview]: ./media/media-services-manage-content/media-services-content-page.png
[publishedcontent]: ./media/media-services-manage-content/media-services-upload-content-published.png
[uploadcontent]: ./media/media-services-manage-content/UploadContent.png
[status]: ./media/media-services-manage-content/Status.png
[encoder]: ./media/media-services-manage-content/EncoderDialog2.png
[branding]: ./media/branding-reporting.png
[contentpage]: ./media/media-services-manage-content/media-services-content-page.png
[process]: ./media/media-services-manage-content/media-services-process-video.png
[process2]: ./media/media-services-manage-content/media-services-process-video2.png
[encrypt]: ./media/media-services-manage-content/media-services-encrypt-content.png
[AMSPlayer]: ./media/media-services-manage-content/media-services-portal-player.png

<!---HONumber=Mooncake_Quality_Review_1118_2016-->