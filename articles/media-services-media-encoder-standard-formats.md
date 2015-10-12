<properties 
	pageTitle="媒体编码器标准格式和编解码器" 
	description="本主题概述 Azure 媒体编码器标准格式和编解码器。" 
	services="media-services" 
	documentationCenter="" 
	authors="juliako" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="media-services" 
	ms.date="08/11/2015"
	wacn.date="10/03/2015"/>

#媒体编码器标准格式和编解码器


本文档包含最常见的导入和导出文件格式的列表，你可以将这些格式与媒体编码器标准配合使用。


[媒体编码器导入格式](#import_formats)

[媒体编码器导出格式](#export_formats)


##<a id="import_formats"></a>媒体编码器导入格式 

以下部分列出了导入操作支持的编解码器和文件格式。


##视频编解码器：

- MPEG-2
- H.264
- YUV420（未压缩或夹层）
- DNxHD
- VC-1/WMV9
- Mpeg-4 第 2 部分
- JPEG 2000
- Theora

###音频编解码器

- PCM
- AAC（AAC-LC、AAC-HE 和 AAC-HEv2）
- WMA9/Pro
- MP3 (MPEG-1 Audio Layer 3)
- FLAC
- Opus
- Vorbis
 
###格式

<table border="1">
<tr><th>文件格式</th><th>文件扩展名</th></tr>
<tr><td>FLV（使用 H.264 和 AAC 编解码器） </td><td>.flv</td></tr>
<tr><td>MP4/ISMV</td><td>.ismv</td></tr>
<tr><td>MPEG2 PS、MPEG2 TS、3GP</td><td>.ts、.ps、.3gp</td></tr>
<tr><td>MXF</td><td>.mxf</td></tr>
<tr><td>WMV/ASF</td><td>.mwv、.asf</td></tr>
<tr><td>DVR-MS</td><td>.dvr-ms </td></tr>
<tr><td>AVI</td><td>.avi</td></tr>
<tr><td>Matroska</td><td>.mkv</td></tr>
<tr><td>GXF</td><td>.gxf</td></tr>
<tr><td>WAVE/WAV </td><td>.wav</td></tr>
</table>

##<a id="export_formats"></a>媒体编码器导出格式

下表列出了导出操作支持的编解码器和文件格式。


<table border="1">
<tr><th>文件格式</th><th>视频编解码器</th><th>音频编解码器</th></tr>
<tr><td>MP4 (*.mp4)<br/><br/>（包括多码率 MP4 容器） </td><td>H.264（High、Main 和 Baseline Profile）</td><td>AAC-LC、HE-AAC v1、HE-AAC v2 </td></tr>
<tr><td>MPEG2-TS </td><td>H.264（High、Main 和 Baseline Profile）</td><td>AAC-LC、HE-AAC v1、HE-AAC v2 </td></tr>
</table>

##另请参阅

[使用 Azure 媒体服务对按需内容进行编码](/documentation/articles/media-services-encode-asset)

<!---HONumber=71-->