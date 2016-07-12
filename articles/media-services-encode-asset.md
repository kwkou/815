<properties 
	pageTitle="简要介绍并比较 Azure 按需媒体编码器" 
	description="本主题概括介绍并比较了 Azure 按需媒体编码器。" 
	services="media-services" 
	documentationCenter="" 
	authors="juliako,anilmur" 
	manager="dwrede" 
	editor=""/>

<tags
	ms.service="media-services"
 	ms.date="02/25/2016"  
	wacn.date="04/05/2016"/>

#简要介绍并比较 Azure 按需媒体编码器

##编码概述

Azure 媒体服务提供了多个用于在云中对媒体进行编码的选项。

一开始使用媒体服务时，了解编解码器与文件格式之间的区别很重要。编解码器是实现压缩/解压缩算法的软件，而文件格式是用于保存压缩视频的容器。

媒体服务所提供的动态打包可让你以媒体服务支持的流格式（MPEG DASH、HLS、Smooth Streaming、HDS）传送自适应比特率 MP4 或平滑流编码内容，而无须重新打包成这些流格式。

若要使用[动态打包](/documentation/articles/media-services-dynamic-packaging-overview/)，必须执行下列操作：

- 将夹层（源）文件编码成一组自适应比特率 MP4 文件或自适应比特率平滑流文件（本教程稍后将演示编码步骤）。
- 针对你要传送内容的流式处理终结点，获取至少一个按需流式处理单位。有关详细信息，请参阅[如何缩放按需流式处理保留单位](/documentation/articles/media-services-manage-origins/#scale_streaming_endpoints)。

媒体服务支持将在本文中介绍的以下按需编码器：

- [媒体编码器标准版](/documentation/articles/media-services-encode-asset.md/#media-encoder-standard)
- [Azure Media Encoder](/documentation/articles/media-services-encode-asset.md/#azure-media-encoder)

本文简要概述了按需媒体编码器，并提供了指向介绍更多详细信息的文章的链接。本主题还提供了编码器的比较。

请注意，默认情况下每个媒体服务帐户同时只能有一个活动的编码任务。你可以保留编码单元，使用它们可以同时运行多个编码任务，你购买的每个编码保留单位对应一个任务。有关信息，请参阅[缩放编码单位](/documentation/articles/media-services-portal-encoding-units/)。

##媒体编码器标准版

###如何使用

[如何使用媒体编码器标准版进行编码](/documentation/articles/media-services-dotnet-encode-with-media-encoder-standard/)

###格式

[格式和编解码器](/documentation/articles/media-services-media-encoder-standard-formats/)

###预设

媒体编码器标准版使用[此处](https://msdn.microsoft.com/zh-cn/library/azure/mt269960.aspx)所述的编码器预设之一进行配置。

###输入和输出元数据

编码器输入元数据在[此处](http://msdn.microsoft.com/zh-cn/library/azure/dn783120.aspx)说明。

[此处](http://msdn.microsoft.com/zh-cn/library/azure/dn783217.aspx)说明了编码器输出元数据。

###生成缩略图

有关信息，请参阅[如何使用媒体编码器标准生成缩略图](/documentation/articles/media-services-custom-mes-presets-with-dotnet/#thumbnails)。

###修剪视频（裁剪）

有关信息，请参阅[如何使用媒体编码器标准修剪视频](/documentation/articles/media-services-custom-mes-presets-with-dotnet/#trim_video)。

###创建覆盖层

有关信息，请参阅[如何使用媒体编码器标准创建覆盖层](/documentation/articles/media-services-custom-mes-presets-with-dotnet/#overlay)。

###另请参阅

[媒体服务博客](https://azure.microsoft.com/blog/2015/07/16/announcing-the-general-availability-of-media-encoder-standard/)
 
##媒体编码器高级工作流

###概述

[在 Azure 媒体服务中引入高级编码](http://azure.microsoft.com/blog/2015/03/05/introducing-premium-encoding-in-azure-media-services)

###如何使用

媒体编码器高级工作流使用复杂的工作流进行配置。可以使用[工作流设计器](/documentation/articles/media-services-workflow-designer/)工具创建和更新工作流文件。

[如何在 Azure 媒体服务中使用高级编码](https://azure.microsoft.com/blog/2015/03/06/how-to-use-premium-encoding-in-azure-media-services/)

###已知问题

如果输入视频不包含隐藏式字幕，输出资产仍将包含一个空的 TTML 文件。


##<a id="compare_encoders"></a>比较编码器

###<a id="billing"></a>每个编码器使用的计费表

媒体处理器名称|适用定价|说明
---|---|---
**媒体编码器标准版** |编码器|编码任务将根据“编码器”列下输出资产的大小（以 GB 为单位）按[此处][1]指定的费率进行收费。
**媒体编码器高级工作流** |高级编码器|编码任务将根据“高级编码器”列下输出资产的大小（以 GB 为单位）按[此处][1]指定的费率进行收费。


本部分将比较**媒体编码器标准版**和**媒体编码器高级工作流**的编码功能。


###输入容器/文件格式

输入容器/文件格式|媒体编码器标准版|媒体编码器高级工作流
---|---|---
Adobe® Flash® F4V |是|是
MXF/SMPTE 377M |是|是
GXF |是|是
Mpeg-2 传输流 |是|是
MPEG-2 节目流 |是|是
MPEG-4/MP4 |是|是
Windows Media/ASF |是|是
AVI（8 位/10 位未压缩）|是|是
3GPP/3GPP2 |是|否
平滑流文件格式 (PIFF 1.3)|是|否
[Microsoft Digital Video Recording(DVR-MS)](https://msdn.microsoft.com/zh-cn/library/windows/desktop/dd692984)|是|否
Matroska/WebM |是|否
QuickTime (.mov) |是|否

###输入视频编解码器

输入视频编解码器|媒体编码器标准版|媒体编码器高级工作流
---|---|---
AVC 8 位/10 位，最高支持 4:2:2，包括 AVCIntra |8 位 4:2:0 和 4:2:2|是
Avid DNxHD（MXF 格式） |是|是
DVCPro/DVCProHD（MXF 格式） |是|是
JPEG2000 |是|是
Mpeg-2（最高支持 422 Profile 和 High Level；包括 XDCAM、XDCAM HD、XDCAM IMX、CableLabs® 和 D10 等变体）|最高支持 422 Profile|是
MPEG-1 |是|是
Windows Media 视频/VC-1 |是|是
Canopus HQ/HQX |否|否
Mpeg-4 第 2 部分 |是|否
[Theora](https://zh.wikipedia.org/wiki/Theora) |是|否
Apple ProRes 422 |是|否
Apple ProRes 422 LT |是|否
Apple ProRes 422 HQ |是|否
Apple ProRes Proxy|是|否
Apple ProRes 4444 |是|否
Apple ProRes 4444 XQ |是|否

###输入音频编解码器

输入音频编解码器|媒体编码器标准版|媒体编码器高级工作流
---|---|---
AES（SMPTE 331M 和 302M、AES3-2003） |否|是
Dolby® E |否|是
Dolby® Digital (AC3) |否|是
Dolby® Digital Plus (E-AC3) |否|是
AAC(AAC-LC、AAC-HE 和 AAC-HEv2；最高支持 5.1）|是|是
MPEG Layer 2|是|是
MP3 (MPEG-1 Audio Layer 3)|是|是
Windows Media 音频|是|是
WAV/PCM|是|是
[FLAC](https://zh.wikipedia.org/wiki/FLAC)</a>|是|否
[Opus](https://zh.wikipedia.org/wiki/Opus_(audio_format) |是|否
[Vorbis](https://zh.wikipedia.org/wiki/Vorbis)</a>|是|否


###输出容器/文件格式

输出容器/文件格式|媒体编码器标准版|媒体编码器高级工作流
---|---|---
Adobe® Flash® F4V|否|是
MXF（OP1a、XDCAM 和 AS02）|否|是
DPP（包括 AS11）|否|是
GXF|否|是
MPEG-4/MP4|是|是
MPEG-TS|是|是
Windows Media/ASF|否|是
AVI（8 位/10 位未压缩）|否|是
平滑流文件格式 (PIFF 1.3)|否|是

###输出视频编解码器

输出视频编解码器|媒体编码器标准版|媒体编码器高级工作流
---|---|---
AVC（H.264；8 位；最高支持 High Profile、Level 5.2；4K Ultra HD；AVC Intra）|仅限 8 位 4:2:0|是
Avid DNxHD（MXF 格式）|否|是
DVCPro/DVCProHD（MXF 格式）|否|是
Mpeg-2（最高支持 422 Profile 和 High Level；包括 XDCAM、XDCAM HD、XDCAM IMX、CableLabs® 和 D10 等变体）|否|是
MPEG-1|否|是
Windows Media 视频/VC-1|否|是
JPEG 缩图创建|否|是

###输出音频编解码器

输出音频编解码器|媒体编码器标准版|媒体编码器高级工作流
---|---|---
AES（SMPTE 331M 和 302M、AES3-2003）|否|是
Dolby® Digital (AC3)|否|是
Dolby® Digital Plus (E-AC3)，最高支持 7.1|否|是
AAC(AAC-LC、AAC-HE 和 AAC-HEv2；最高支持 5.1）|是|是
MPEG Layer 2|否|是
MP3 (MPEG-1 Audio Layer 3)|否|是
Windows Media 音频|否|是


##错误代码  

下表列出了在执行编码任务期间发生错误的情况下可能返回的错误代码。若要获取 .NET 代码中的错误详细信息，请使用 [ErrorDetails](https://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.mediaservices.client.errordetail.aspx) 类。若要获取 REST 代码中的错误详细信息，请使用 [ErrorDetail](https://msdn.microsoft.com/zh-cn/library/jj853026.aspx) REST API。

ErrorDetail.Code|出错的可能原因
-----|-----------------------
Unknown| 执行任务期间发生未知的错误
ErrorDownloadingInputAssetMalformedContent|涵盖下载输入资产时出错（例如，错误的文件名称、文件长度为零、格式不正确，等等）的错误类别。
ErrorDownloadingInputAssetServiceFailure|涵盖服务端问题（例如，下载时发生网络或存储错误）的错误类别。
ErrorParsingConfiguration|任务 <see cref="MediaTask.PrivateData"/>（配置）无效的错误类别，例如，配置不是有效的系统预设或包含无效的 XML。
ErrorExecutingTaskMalformedContent|在执行任务期间因输入媒体文件内部问题导致失败的错误类别。
ErrorExecutingTaskUnsupportedFormat|媒体处理器无法处理提供的文件（不支持的媒体格式或与配置不匹配）的错误类别。例如，尝试从只包含视频的资产生成只包含音频的输出
ErrorProcessingTask|媒体处理器在处理与内容无关的任务时发生的其他错误类别。
ErrorUploadingOutputAsset|上载输出资产时的错误类别
ErrorCancelingTask|涵盖尝试取消任务时失败的错误类别
TransientError|涵盖暂时性问题（例如 Azure 存储空间发生暂时性网络问题）的错误类别








##相关文章

- [通过自定义媒体编码器标准预设执行高级编码任务](/documentation/articles/media-services-custom-mes-presets-with-dotnet/)
- [配额和限制](/documentation/articles/media-services-quotas-and-limitations/)

 
<!--Reference links in article-->
[1]: /home/features/media-services/pricing/

<!---HONumber=Mooncake_0328_2016-->