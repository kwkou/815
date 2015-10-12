<properties 
	pageTitle="Azure Media Encoder 格式和编解码器" 
	description="本主题概述 Azure 媒体编码器格式和编解码器。" 
	services="media-services" 
	documentationCenter="" 
	authors="juliako" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="media-services" 
	ms.date="08/11/2015" 
	wacn.date="10/03/2015"/>

#Azure Media Encoder 格式和编解码器

编码器使用编解码器压缩数字媒体。通常，通过编码器的各种设置，你可以指定所生成媒体的属性，例如，使用的编解码器、文件格式、分辨率和比特率。文件格式是用于保存压缩视频以及用来压缩视频的编解码器的相关信息的容器。

编解码器有两个组件：一个用于压缩数字媒体文件进行传输，另一个用于解压缩数字媒体文件进行播放。有音频编解码器（用于压缩和解压缩音频）和视频编解码器（用于压缩和解压缩视频）。编解码器可以使用无损压缩或有损压缩。无损编解码器在进行压缩时会保留全部信息。文件解压缩后，得到的文件与输入媒体相同，因此无损编解码器非常适用于存档和存储。有损编解码器在编码时会丢失部分信息，可通过牺牲视频质量生成比原始文件小的文件，因此非常适用于在 Internet 上进行流式传输。Azure Media Encoder 主要使用以下两款编解码器来进行编码：H.264 和 VC-1。我们的合作伙伴编码器生态系统中可能还提供了其他编解码器。

了解编解码器与文件格式之间的区别很重要。编解码器是实现压缩/解压缩算法的软件，而文件格式是用于保存压缩视频的容器。有关详细信息，请参阅[编码与打包](http://blog-ndrouin.chinacloudsites.cn/streaming-media-terminology-explained/)。

本文档包含最常见的导入和导出文件格式的列表，你可以将这些格式用于 Azure Media Encoder。


[Azure 媒体编码器导入格式](#import_formats)

[Azure 媒体编码器导出格式](#export_formats)


##<a id="import_formats"></a>Azure 媒体编码器导入格式 

以下部分列出了导入操作支持的编解码器和文件格式。

###视频编解码器

- H.264（Baseline、Main 和 High Profile）
- MPEG-1（包含 MPEG-PS）
- Mpeg-2（Simple 和 Main Profile 及 4:2:2 Profile）
- MPEG-4 v2（Simple Visual Profile 和 Advanced Simple Profile）
- VC-1（Simple、Main 和 Advanced Profile）
- Windows Media 视频（Simple、Main 和 Advanced Profile）
- DV（DVC、DVHD、DVSD、DVSL）
- Grass Valley HQ/HQX
 
###音频编解码器

- AC-3（杜比数字音频）
- AAC（AAC-LC、带 AAC-LC 内核的 HE-AAC v1 和带 AAC-LC 内核的 HE-AAC v2）
- MP3
- Windows Media Audio 9（Windows Media Audio Standard、Windows Media Audio Professional 和 Windows Media Audio Lossless）

###视频文件格式
 
<table border="1">
<tr><th>文件格式</th><th>文件扩展名</th></tr>
<tr><td>3GPP、3GPP2</td><td>.3gp、.3g2、.3gp2</td></tr>
<tr><td>高级流格式 (ASF)</td><td>.asf</td></tr>
<tr><td>高级视频编码高清晰 (AVCHD) [MPEG-2 传输流]</td><td>.mts、.m2ts</td></tr>
<tr><td>音频视频交错 (AVI)</td><td>.avi</td></tr>
<tr><td>数码摄录机 MPEG-2 (MOD)</td><td>.mod</td></tr>
<tr><td>DVD 传输流 (TS) 文件</td><td>.ts</td></tr>
<tr><td>DVD 视频对象 (VOB) 文件</td><td>.vob</td></tr>
<tr><td>Expression Encoder 屏幕捕获编解码器文件</td><td>.xesc</td></tr>
<tr><td>MP4</td><td>.mp4</td></tr>
<tr><td>MPEG-1 系统流</td><td>.mpeg、.mpg</td></tr>
<tr><td>MPEG-2 视频文件</td><td>.m2v</td></tr>
<tr><td>平滑流文件格式 (PIFF 1.3)</td><td>.ismv</td></tr>
<tr><td>Windows Media 视频 (WMV)</td><td>.wmv</td></tr>
</table>

支持一些未压缩的格式。有关详细信息，请参阅[支持的未压缩视频格式](#uncompressed)

###音频文件格式

<table border="1">
<tr><th>文件格式</th><th>文件扩展名</th></tr>
<tr><td>AC-3（杜比数字）音频</td><td>.ac3</td></tr>
<tr><td>音频交换文件格式 (AIFF)</td><td>.aiff</td></tr>
<tr><td>广播波形格式</td><td>.bwf</td></tr>
<tr><td>MP3 (MPEG-1 Audio Layer 3)</td><td>.mp3</td></tr>
<tr><td>MP4 音频</td><td>.m4A</td></tr>
<tr><td>MPEG-4 有声读物</td><td>.m4b</td></tr>
<tr><td>WAVE 文件</td><td>.wav</td></tr>
<tr><td>Windows Media 音频</td><td>.wma</td></tr>   
</table>

###图像文件格式

<table border="1">
<tr><th>文件格式</th><th>文件扩展名</th></tr>
<tr><td>位图</td><td>.bmp</td></tr>
<tr><td>GIF、动画 GIF</td><td>.gif</td></tr>
<tr><td>JPEG</td><td>.jpeg、.jpg</td></tr>
<tr><td>PNG</td><td>.png</td></tr>
<tr><td>TIFF</td><td>.tif</td></tr>
<tr><td>WPF Canvas XAML</td><td>.xaml</td></tr>
</table>


##<a id="export_formats"></a>Azure 媒体编码器导出格式

下表列出了导出操作支持的编解码器和文件格式。


<table border="1">
<tr><th>文件格式</th><th>视频编解码器</th><th>音频编解码器</th></tr>
<tr><td>Windows Media（*.wmv；*.wma）</td><td>VC-1（Advanced、Main 和 Simple Profile）</td><td>Windows Media Audio Standard、Windows Media Audio Professional、Windows Media Audio Voice、Windows Media Audio Lossless</td></tr>
<tr><td>MP4 (*.mp4)</td><td>H.264（High、Main 和 Baseline Profile）</td><td>AAC-LC、HE-AAC v1、HE-AAC v2、Dolby Digital Plus</td></tr>
<tr><td>平滑流文件格式 (PIFF 1.1)（*.ismv；*.isma）</td><td>VC-1 (Advanced Profile)<br/><br/>
H.264（High、Main 和 Baseline Profile）</td><td>Windows Media Audio Standard、Windows Media Audio Professional<br/><br/>
AAC-LC、HE-AAC v1、HE-AAC v2</td></tr>
</table>

有关媒体服务中支持的其他编解码器和筛选器，请参阅[ Windows DirectShow 筛选器](https://msdn.microsoft.com/zh-cn/library/windows/desktop/dd375464.aspx)。

##<a id="uncompressed"></a>支持的未压缩视频格式 

Azure 媒体服务为导入未压缩的视频数据提供支持。

下面是支持的未压缩格式的部分列表。

<table border="1">
<tr><th>未压缩的视频格式</th><th>说明</th></tr>
<tr><td>标准 YVU9 格式的未压缩数据</td><td>平面 YUV 格式。每隔一个像素取一个 Y 样本，每一行上每隔三个水平像素取一个 U 和 V 样本；每隔一个垂直行取一个 Y 样本，每隔三个垂直行取一个 U 和 V 样本。每个像素 9 位。</td></tr>
<tr><td>YUV 411 格式数据</td><td>每隔一像素取一个 Y 样本，每一行上每隔三个水平像素取一个 U 和 V 样本；对每个垂直行采样。字节排序（从低到高）为 U0、Y0、V0、Y1、U4、Y2、V4、Y3、Y4、Y5、Y6、Y7，其中，后缀 0 为最左侧像素，递增的数字是从左到右递增的像素。每个 12 字节块为 8 个图像像素。</td></tr>
<tr><td>Y41P 格式数据</td><td>每隔一像素取一个 Y 样本，每一行上每隔三个水平像素取一个 U 和 V 样本；对每个垂直行采样。字节排序（最低到高）为 U0、Y0、V0、Y1、U4、Y2、V4、Y3、Y4、Y5、Y6、Y7，其中，后缀 0 为最左侧像素，递增的数字是从左到右递增的像素。每个 12 字节块为 8 个图像像素。</td></tr>
<tr><td>YUY2 格式数据</td><td>除了像素排序不同外，其他与 UYVY 相同。字节排序（从低到高）为 Y0、U0、Y1、V0、Y2、U2、Y3、V2、Y4、U4、Y5、V4，其中，后缀 0 为最左侧像素，递增的数字是从左到右递增的像素。每个 4 字节块为 2 个图像像素。</td></tr>
<tr><td>YVYU 格式数据</td><td>打包的 YUV 格式。除了像素排序不同外，其他与 UYVY 相同。字节排序（从低到高）为 Y0、V0、Y1、U0、Y2、V2、Y3、U2、Y4、V4、Y5、U4，其中，后缀 0 为最左侧像素，递增的数字是从左到右递增的像素。每个 4 字节块为 2 个图像像素。</td></tr>
<tr><td>UYVY 格式数据</td><td>打包的 YUV 格式。每隔一像素取一个 Y 样本，每一行上每隔一个水平像素取一个 U 和 V 样本；对每个垂直行采样。最常用的 YUV 4:2:2 格式。字节排序（从低到高）为 U0、Y0、V0、Y1、U2、Y2、V2、Y3、U4、Y4、V4、Y5，其中，后缀 0 为最左侧像素，递增的数字是从左到右递增的像素。每个 4 字节块为 2 个图像像素。</td></tr>
<tr><td>YUV 211 格式数据</td><td>打包的 YUV 格式。每隔一个像素取一个 Y 样本，在每一行上每隔三个水平像素取一个 U 和 V 样本；对每个垂直行采样。字节排序（最低到高）为 Y0、U0、Y2、V0、Y4、U4、Y6、V4、Y8、U8、Y10、V8，其中，后缀 0 为最左侧像素，递增的数字是从左到右递增的像素。每个 4 字节块为 4 个图像像素。</td></tr>
<tr><td>Cirrus Logic Jr YUV 411 格式</td><td>每个 Y、U 和 V 样本小于 8 位的 Cirrus Logic Jr YUV 411 格式。每隔一像素取一个 Y 样本，每一行上每隔三个水平像素取一个 U 和 V 样本；对每个垂直行采样。</td></tr>
<tr><td>Indeo 生成的 YVU9 格式</td><td>Indeo 生成的 YVU9 格式包含有关与上一帧的差别的附加信息。每个像素 9.5 位，但报告 9 位。</td></tr>
</table>

<!---HONumber=71-->