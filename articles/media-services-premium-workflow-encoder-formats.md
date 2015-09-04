<properties 
	pageTitle="Media Encoder Premium Workflow 格式和编解码器" 
	description="本主题概述 Media Encoder Premium Workflow 的格式和编解码器" 
	services="media-services" 
	documentationCenter="" 
	authors="juliako" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="media-services" 
	ms.date="06/29/2015" 
	wacn.date="08/29/2015"/>

#Media Encoder Premium Workflow 格式和编解码器


**请注意**，本主题中所述的 Media Encoder Premium Workflow 媒体处理器在中国不可用。

本文档包含的输入和输出文件格式及编解码器列表受 **Media Encoder Premium Workflow** 公开预览版编码器支持。

[Media Encoder Premium Workflow 输入格式和编解码器](#input_formats)

[Media Encoder Premium Workflow 输出格式和编解码器](#output_formats)

**Media Encoder Premium Workflow** 支持[此](#closed_captioning)部分中所述的隐藏字幕。


##<a id="input_formats"></a>Media Encoder Premium Workflow 输入格式和编解码器

以下部分列出了此媒体处理器支持的作为输入的编解码器和文件格式。

###输入容器/文件格式

- Adobe® Flash® F4V
- MXF/SMPTE 377M
- GXF
- Mpeg-2 传输流
- MPEG-2 节目流
- MPEG-4/MP4
- Windows Media/ASF
- AVI（8 位/10 位未压缩）

###输入视频编解码器

- AVC 8 位/10 位，最高支持 4:2:2，包括 AVCIntra
- Avid DNxHD（MXF 格式）
- DVCPro/DVCProHD（MXF 格式）
- JPEG2000
- Mpeg-2（最高支持 422 Profile 和 High Level；包括 XDCAM、XDCAM HD、XDCAM IMX、CableLabs® 和 D10 等变体）
- MPEG-1
- Windows Media 视频/VC-1

###输入音频编解码器

- AES（SMPTE 331M 和 302M、AES3-2003）
- Dolby® E
- Dolby® Digital (AC3)
- AAC(AAC-LC、AAC-HE 和 AAC-HEv2；最高支持 5.1）
- MPEG Layer 2
- MP3 (MPEG-1 Audio Layer 3)
- Windows Media 音频
- WAV/PCM
 
##<a id="output_format"></a>Media Encoder Premium Workflow 输出格式和编解码器

以下部分列出了支持作为此媒体处理器输入的编解码器和文件格式。

###输出容器/文件格式

- Adobe® Flash® F4V
- MXF（OP1a、XDCAM 和 AS02）
- DPP（包括 AS11）
- GXF
- MPEG-4/MP4
- Windows Media/ASF
- AVI（8 位/10 位未压缩）
- 平滑流文件格式 (PIFF 1.3)
- MPEG-TS 


###输出视频编解码器

- AVC（H.264；8 位；最高支持 High Profile、Level 5.2；4K Ultra HD；AVC Intra）
- Avid DNxHD（MXF 格式）
- DVCPro/DVCProHD（MXF 格式）
- Mpeg-2（最高支持 422 Profile 和 High Level；包括 XDCAM、XDCAM HD、XDCAM IMX、CableLabs® 和 D10 等变体）
- MPEG-1
- Windows Media 视频/VC-1
- JPEG 缩图创建

###输出音频编解码器

- AES（SMPTE 331M 和 302M、AES3-2003）
- Dolby® Digital (AC3)
- Dolby® Digital Plus (E-AC3)，最高支持 7.1
- AAC(AAC-LC、AAC-HE 和 AAC-HEv2；最高支持 5.1）
- MPEG Layer 2
- MP3 (MPEG-1 Audio Layer 3)
- Windows Media 音频

##<a id="closed_captioning"></a>支持隐藏式字幕

插入时，**Media Encoder Premium Workflow** 支持：

1. SCC 文件
1. SMPTE-TT 文件
1. CEA-608/CEA-708 – 作为用户数据（H.264 基本流的 SEI 消息，ATSC/53，SCTE20）或作为 MXF 文件中的辅助数据携载
1. STL 字幕文件

输出时，提供以下选项：

1. CEA-608 到 CEA-708 的转换
1. CEA-608/CEA-708 传递（内嵌在 H.264 基本流的 SEI 消息中，或作为 MXF 文件中的辅助数据携载）
1. SCC
1. SMPTE 时序文本（来自符合 SMPTE RP2052 的源 CEA-608；包括 DFXP 文件创建）
1. SRT 字幕文件
1. DVB 字幕流

注意：不一定支持通过 Azure Media Services 中的流式传输来传送上述所有输出格式。

##已知问题

如果输入视频不包含隐藏式字幕，输出资产仍将包含一个空的 TTML 文件。

<!---HONumber=67-->