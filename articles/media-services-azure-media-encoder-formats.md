<properties 
	pageTitle="Azure Media Encoder 格式和编解码器" 
	description="本主题概述 Azure 媒体编码器格式和编解码器。" 
	services="media-services" 
	documentationCenter="" 
	authors="juliako,anilmur" 
	manager="dwrede" 
	editor=""/>

<tags
	ms.service="media-services"
	ms.date="02/03/2016"  
	wacn.date="03/17/2016"/>

#Azure Media Encoder 格式和编解码器

本文档包含最常见的输入、输出文件格式和编解码器的列表，你可以将这些格式和编解码器用于 Azure 媒体编码器。


##输入文件格式（容器）
 
文件格式（文件扩展名）|支持
---|---
3GPP、3GPP2（.3gp、.3g2、.3gp2） |是
高级系统格式 (ASF) (.asf) |是
高级视频编码高清晰 (AVCHD) [MPEG-2 传输流] (.mts, .m2ts)|是
音频视频交错 (AVI) (.avi) |是
数码摄录机 MPEG-2 (MOD) (.mod) |是
DVD 传输流 (TS) 文件 (.ts) |是
DVD 视频对象 (VOB) 文件 (.vob) |是
Expression Encoder 屏幕捕获编解码器文件 (.xesc) |是
MP4（.mp4、.m4a、.m4v）/ISMV（.isma、.ismv） |是
MPEG-1 系统流（.mpeg、.mpg） |是
MPEG-2 视频文件 (.m2v) |是
Windows Media 视频 (WMV) (.wmv) |是
AC-3（杜比数字）音频 (.ac3)|是
音频交换文件格式 (AIFF) (.aiff)|是
广播波形格式 (.bwf)|是
MP3 (MPEG-1 Audio Layer 3)(.mp3)|是
MPEG-4 有声读物 (.m4b)|是
WAVE 文件 (.wav)|是
Windows Media 音频 (.wma)|是
Adobe® Flash® F4V |否		
MXF/SMPTE 377M |受限制
GXF |否		
[Microsoft Digital Video Recording(DVR-MS)](https://msdn.microsoft.com/zh-cn/library/windows/desktop/dd692984)|否
Matroska/WebM |否


支持一些未压缩的格式。有关详细信息，请参阅[支持的未压缩视频格式](#uncompressed)

##输入视频编解码器

输入视频编解码器|支持
---|--- 
H.264（Baseline、Main 和 High Profile） |是
AVC 8 位/10 位，最高支持 4:2:2，包括 AVCIntra |仅限 8 位 4:2:0
Avid DNxHD（MXF 格式） |否
DVCPro/DVCProHD（MXF 格式） |否
JPEG2000 |否
Mpeg-2（Simple 和 Main Profile 及 4:2:2 Profile） |最高支持 4:2:2 Profile
MPEG-1（包含 MPEG-PS） |是
Windows Media 视频/VC-1 |是
Canopus HQ/HQX |是
MPEG-4 v2（Simple Visual Profile 和 Advanced Simple Profile） |是
[Theora](https://en.wikipedia.org/wiki/Theora) |否
VC-1（Simple、Main 和 Advanced Profile） |是
Windows Media 视频（Simple、Main 和 Advanced Profile） |是
DV（DVC、DVHD、DVSD、DVSL） |是
Grass Valley HQ/HQX |是
 

##输入音频编解码器

输入音频编解码器|支持
---|---
AES（SMPTE 331M 和 302M、AES3-2003） |否
Dolby® E |否
Dolby® Digital (AC3) |是
Dolby® Digital Plus (E-AC3) |否
AAC（AAC-LC、带 AAC-LC 内核的 HE-AAC v1 和带 AAC-LC 内核的 HE-AAC v2；最高支持 5.1）|是
MPEG Layer 2|是|是|是
MP3 (MPEG-1 Audio Layer 3)|是
Windows Media Audio 9（Windows Media Audio Standard、Windows Media Audio Professional 和 Windows Media Audio Lossless） |是
WAV/PCM|是
[FLAC](https://en.wikipedia.org/wiki/FLAC)|否
[Opus](https://en.wikipedia.org/wiki/Opus_codec) |否
[Vorbis](https://en.wikipedia.org/wiki/Vorbis)|否


##输入图像文件格式

文件格式（文件扩展名） | 支持
---|---
位图 (.bmp) | 是
GIF、动画 GIF (.gif)| 是
JPEG（.jpeg、.jpg）| 是
PNG (.png)| 是
TIFF (.tif)| 是
WPF Canvas XAML (.xaml)| 是


##输出格式和编解码器

下表列出了导出操作支持的编解码器和文件格式。

文件格式|视频编解码器|音频编解码器
---|---|---
Windows Media（* .wmv；* .wma）|VC-1（Advanced、Main 和 Simple Profile）|Windows Media Audio Standard、Windows Media Audio Professional、Windows Media Audio Voice、Windows Media Audio Lossless
MP4 (* .mp4)|H.264（High、Main 和 Baseline Profile）|AAC-LC、HE-AAC v1、HE-AAC v2、Dolby Digital Plus
平滑流文件格式 (PIFF 1.1)（* .ismv；* .isma）|VC-1 (Advanced Profile)<p>H.264（High、Main 和 Baseline Profile） |Windows Media Audio Standard、Windows Media Audio Professional<p><p>AAC-LC、HE-AAC v1、HE-AAC v2

有关媒体服务中支持的其他编解码器和筛选器，请参阅 [Windows DirectShow 筛选器](https://msdn.microsoft.com/zh-cn/library/windows/desktop/dd375464.aspx)。

##<a id="uncompressed"></a>支持的未压缩视频格式 

Azure 媒体服务为导入未压缩的视频数据提供支持。

下面是支持的未压缩格式的部分列表。

未压缩的视频格式|说明
---|---
标准 YVU9 格式的未压缩数据|平面 YUV 格式。每隔一个像素取一个 Y 样本，每一行上每隔三个水平像素取一个 U 和 V 样本；每隔一个垂直行取一个 Y 样本，每隔三个垂直行取一个 U 和 V 样本。每个像素 9 位。
YUV 411 格式数据|每隔一像素取一个 Y 样本，每一行上每隔三个水平像素取一个 U 和 V 样本；对每个垂直行采样。字节排序（从低到高）为 U0、Y0、V0、Y1、U4、Y2、V4、Y3、Y4、Y5、Y6、Y7，其中，后缀 0 为最左侧像素，递增的数字是从左到右递增的像素。每个 12 字节块为 8 个图像像素。
Y41P 格式数据|每隔一像素取一个 Y 样本，每一行上每隔三个水平像素取一个 U 和 V 样本；对每个垂直行采样。字节排序（最低到高）为 U0、Y0、V0、Y1、U4、Y2、V4、Y3、Y4、Y5、Y6、Y7，其中，后缀 0 为最左侧像素，递增的数字是从左到右递增的像素。每个 12 字节块为 8 个图像像素。
YUY2 格式数据|除了像素排序不同外，其他与 UYVY 相同。字节排序（从低到高）为 Y0、U0、Y1、V0、Y2、U2、Y3、V2、Y4、U4、Y5、V4，其中，后缀 0 为最左侧像素，递增的数字是从左到右递增的像素。每个 4 字节块为 2 个图像像素。
YVYU 格式数据|打包的 YUV 格式。除了像素排序不同外，其他与 UYVY 相同。字节排序（从低到高）为 Y0、V0、Y1、U0、Y2、V2、Y3、U2、Y4、V4、Y5、U4，其中，后缀 0 为最左侧像素，递增的数字是从左到右递增的像素。每个 4 字节块为 2 个图像像素。
UYVY 格式数据|打包的 YUV 格式。每隔一像素取一个 Y 样本，每一行上每隔一个水平像素取一个 U 和 V 样本；对每个垂直行采样。最常用的 YUV 4:2:2 格式。字节排序（从低到高）为 U0、Y0、V0、Y1、U2、Y2、V2、Y3、U4、Y4、V4、Y5，其中，后缀 0 为最左侧像素，递增的数字是从左到右递增的像素。每个 4 字节块为 2 个图像像素。
YUV 211 格式数据|打包的 YUV 格式。每隔一个像素取一个 Y 样本，在每一行上每隔三个水平像素取一个 U 和 V 样本；对每个垂直行采样。字节排序（最低到高）为 Y0、U0、Y2、V0、Y4、U4、Y6、V4、Y8、U8、Y10、V8，其中，后缀 0 为最左侧像素，递增的数字是从左到右递增的像素。每个 4 字节块为 4 个图像像素。
Cirrus Logic Jr YUV 411 格式|每个 Y、U 和 V 样本小于 8 位的 Cirrus Logic Jr YUV 411 格式。每隔一像素取一个 Y 样本，每一行上每隔三个水平像素取一个 U 和 V 样本；对每个垂直行采样。
Indeo 生成的 YVU9 格式|Indeo 生成的 YVU9 格式包含有关与上一帧的差别的附加信息。每个像素 9.5 位，但报告 9 位。

<!---HONumber=Mooncake_0307_2016-->