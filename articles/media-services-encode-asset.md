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
	ms.date="10/15/2015"
	wacn.date="11/12/2015"/>

#简要介绍并比较 Azure 按需媒体编码器

##编码概述

Azure 媒体服务提供了多个用于在云中对媒体进行编码的选项。

一开始使用媒体服务时，了解编解码器与文件格式之间的区别很重要。编解码器是实现压缩/解压缩算法的软件，而文件格式是用于保存压缩视频的容器。

Media Services 所提供的动态打包可让你以 Media Services 支持的流格式（MPEG DASH、HLS、Smooth Streaming、HDS）传送自适应比特率 MP4 或平滑流编码内容，而无须重新打包成这些流格式。

若要使用[动态打包](/documentation/articles/media-services-dynamic-packaging-overview)，必须执行下列操作：

- 将夹层（源）文件编码成一组自适应比特率 MP4 文件或自适应比特率平滑流文件（本教程稍后将演示编码步骤）。
- 针对你要传送内容的流式处理终结点，获取至少一个按需流式处理单位。有关详细信息，请参阅[如何缩放按需流式处理保留单位](/documentation/articles/media-services-manage-origins#scale_streaming_endpoints)。

媒体服务支持将在本文中介绍的以下按需编码器：

- **媒体编码器标准版**
- **Azure 媒体编码器** 

本文简要概述了按需媒体编码器，并提供了指向介绍更多详细信息的文章的链接。本主题还提供了编码器的比较。

请注意，默认情况下每个媒体服务帐户同时只能有一个活动的编码任务。你可以保留编码单元，使用它们可以同时运行多个编码任务，你购买的每个编码保留单位对应一个任务。有关信息，请参阅[缩放编码单元](/documentation/articles/media-services-portal-encoding-units)。

##媒体编码器标准版

###概述

建议使用媒体编码器标准版编码器。但是，它当前未通过 Azure 管理门户公开。

与 Azure 媒体编码器相比，此编码器支持更多输入和输出格式和编解码器。其他优点包括：

- 对创建输入文件的方式的要求低
- 与 Azure 媒体编码器相比，具有更好的 H.264 编解码器质量
- 在更新、更灵活的管道上构建
- 更可靠/更具弹性

###如何使用

[如何使用媒体编码器标准版进行编码](/documentation/articles/media-services-dotnet-encode-with-media-encoder-standard)

###格式

[格式和编解码器](/documentation/articles/media-services-media-encoder-standard-formats)

###预设

媒体编码器标准版使用[此处](https://msdn.microsoft.com/zh-cn/library/azure/mt269960.aspx)所述的编码器预设之一进行配置。

###输入和输出元数据

编码器输入元数据在[此处](http://msdn.microsoft.com/zh-cn/library/azure/dn783120.aspx)说明。

编码器输出元数据在[此处](http://msdn.microsoft.com/zh-cn/library/azure/dn783217.aspx)说明。

###音频和/或视频叠加

目前不支持。

###另请参阅

[媒体服务博客](https://azure.microsoft.com/blog/2015/07/16/announcing-the-general-availability-of-media-encoder-standard/)
 
##Azure 媒体编码器

###概述

Azure 媒体编码器是媒体服务支持的编码器之一。从 2015 年 7 月开始，建议使用[媒体编码器标准版](/documentation/articles/media-services-encode-asset#media_encoder_standard)。

###如何使用

[如何使用 Azure 媒体编码器进行编码](/documentation/articles/media-services-dotnet-encode-asset)

###格式

[格式和编解码器](/documentation/articles/media-services-azure-media-encoder-formats)

###预设

Azure 媒体编码器使用[此处](https://msdn.microsoft.com/zh-cn/library/azure/dn619392.aspx)所述的编码器预设之一进行配置。你还可以在[此处](https://github.com/Azure/azure-media-services-samples/tree/master/Encoding%20Presets/VoD/Azure%20Media%20Encoder)获取实际的 Azure 媒体编码器预设文件。

###输入和输出元数据

编码器输入元数据在[此处](http://msdn.microsoft.com/zh-cn/library/azure/dn783120.aspx)说明。

编码器输出元数据在[此处](http://msdn.microsoft.com/zh-cn/library/azure/dn783217.aspx)说明。

###缩略图

[创建缩略图](https://msdn.microsoft.com/zh-cn/library/hh973624.aspx)

###音频和/或视频叠加

[创建叠加](/documentation/articles/media-services-azure-media-customize-ame-presets#creating-overlays)。

###命名约定

[如何修改输出文件名](/documentation/articles/media-services-azure-media-customize-ame-presets#controlling-azure-media-encoder-output-file-names)

###另请参阅

[使用 Dolby Digital Plus 编码媒体](/documentation/articles/media-services-encode-with-dolby-digital-plus)



##<a id="compare_encoders"></a>比较编码器

###<a id="billing"></a>每个编码器使用的计费表

媒体处理器名称|适用定价|说明
---|---|---
**媒体编码器标准版** |编码器|编码任务将根据“编码器”列下输出资产的大小（以 GB 为单位）按[此处][1]指定的费率进行收费。
**Azure 媒体编码器** |编码器|编码任务将根据“编码器”列下输出资产的大小（以 GB 为单位）按[此处][1]指定的费率进行收费。


本部分比较了**媒体编码器标准版**和 **Azure 媒体编码器**的编码功能。


###输入容器/文件格式

输入容器/文件格式|媒体编码器标准版|Azure 媒体编码器|媒体编码器高级工作流
---|---|---|---
Adobe® Flash® F4V |是|否 |是
MXF/SMPTE 377M |是|受限制|是
GXF |是|否 |是
Mpeg-2 传输流 |是|是 |是
MPEG-2 节目流 |是|是 |是
MPEG-4/MP4 |是|是 |是
Windows Media/ASF |是|是 |是
AVI（8 位/10 位未压缩）|是|是 |是
3GPP/3GPP2 |是|是 |否
平滑流文件格式 (PIFF 1.3)|是|是|否
[Microsoft Digital Video Recording(DVR-MS)](https://msdn.microsoft.com/zh-cn/library/windows/desktop/dd692984)|是|否|否
Matroska/WebM |是|否|否

###输入视频编解码器

输入视频编解码器|媒体编码器标准版|Azure 媒体编码器|媒体编码器高级工作流
---|---|---|---
AVC 8 位/10 位，最高支持 4:2:2，包括 AVCIntra |8 位 4:2:0 和 4:2:2|仅限 8 位 4:2:0|是
Avid DNxHD（MXF 格式） |是|否|是
DVCPro/DVCProHD（MXF 格式） |是|否|是
JPEG2000 |是|否|是
Mpeg-2（最高支持 422 Profile 和 High Level；包括 XDCAM、XDCAM HD、XDCAM IMX、CableLabs® 和 D10 等变体）|最高支持 422 Profile|最高支持 422 Profile|是
MPEG-1 |是|是|是
Windows Media 视频/VC-1 |是|是|是
Canopus HQ/HQX |否|是|否
Mpeg-4 第 2 部分 |是|否|否
[Theora](https://en.wikipedia.org/wiki/Theora) |是|否|否

###输入音频编解码器

输入音频编解码器|媒体编码器标准版|Azure 媒体编码器|媒体编码器高级工作流
---|---|---|---
AES（SMPTE 331M 和 302M、AES3-2003） |否|否|是
Dolby® E |否|否|是
Dolby® Digital (AC3) |否|是|是
Dolby® Digital Plus (E-AC3) |否|否|是
AAC(AAC-LC、AAC-HE 和 AAC-HEv2；最高支持 5.1）|是|是|是
MPEG Layer 2|是|是|是
MP3 (MPEG-1 Audio Layer 3)|是|是|是
Windows Media 音频|是|是|是
WAV/PCM|是|是|是
[FLAC](https://en.wikipedia.org/wiki/FLAC)</a>|是|否|否
[Opus](https://en.wikipedia.org/wiki/Opus_(audio_format)) |是|否|否
[Vorbis](https://en.wikipedia.org/wiki/Vorbis)</a>|是|否|否


###输出容器/文件格式

输出容器/文件格式|媒体编码器标准版|Azure 媒体编码器|媒体编码器高级工作流
---|---|---|---
Adobe® Flash® F4V|否|否|是
MXF（OP1a、XDCAM 和 AS02）|否|否|是
DPP（包括 AS11）|否|否|是
GXF|否|否|是
MPEG-4/MP4|是|是|是
MPEG-TS|是|否|是
Windows Media/ASF|否|是|是
AVI（8 位/10 位未压缩）|否|否|是
平滑流文件格式 (PIFF 1.3)|否|是|是

###输出视频编解码器

输出视频编解码器|媒体编码器标准版|Azure 媒体编码器|媒体编码器高级工作流
---|---|---|---
AVC（H.264；8 位；最高支持 High Profile、Level 5.2；4K Ultra HD；AVC Intra）|仅限 8 位 4:2:0|仅限 8 位 4:2:0，最高支持 1080p|是
Avid DNxHD（MXF 格式）|否|否|是
DVCPro/DVCProHD（MXF 格式）|否|否|是
Mpeg-2（最高支持 422 Profile 和 High Level；包括 XDCAM、XDCAM HD、XDCAM IMX、CableLabs® 和 D10 等变体）|否|否|是
MPEG-1|否|否|是
Windows Media 视频/VC-1|否|是|是
JPEG 缩图创建|否|是|是

###输出音频编解码器

输出音频编解码器|媒体编码器标准版|Azure 媒体编码器|媒体编码器高级工作流
---|---|---|---
AES（SMPTE 331M 和 302M、AES3-2003）|否|否|是
Dolby® Digital (AC3)|否|是|是
Dolby® Digital Plus (E-AC3)，最高支持 7.1|否|最高支持 5.1|是
AAC(AAC-LC、AAC-HE 和 AAC-HEv2；最高支持 5.1）|是|是|是
MPEG Layer 2|否|否|是
MP3 (MPEG-1 Audio Layer 3)|否|否|是
Windows Media 音频|否|是|是


##媒体服务学习路径

你可以在此处查看 AMS 学习路径：

- [AMS 实时流式处理工作流](http://azure.microsoft.com/documentation/learning-paths/media-services-streaming-live/)
- [AMS 按需流式处理工作流](http://azure.microsoft.com/documentation/learning-paths/media-services-streaming-on-demand/)

##相关文章

- [配额和限制](/documentation/articles/media-services-quotas-and-limitations)

 
<!--Reference links in article-->
[1]: /home/features/media-services/#price

<!---HONumber=79-->