<properties 
	pageTitle="如何创建媒体处理器 - Azure" 
	description="了解如何创建一个媒体处理器组件用来为 Azure 媒体服务编码、转换格式、加密或解密媒体内容。代码示例用 C# 编写且使用 Media Services SDK for .NET。" 
	services="media-services" 
	documentationCenter="" 
	authors="juliako" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="media-services" 
	ms.date="09/07/2015" 
	wacn.date="10/22/2015"/>


#如何：获取媒体处理器实例

> [AZURE.SELECTOR]
- [.NET](/documentation/articles/media-services-get-media-processor)
- [REST](/documentation/articles/media-services-rest-get-media-processor)

##概述

在媒体服务中，媒体处理器是完成特定处理任务（例如，对媒体内容进行编码、格式转换、加密或解密）的组件。通常，当你创建一个任务以便对媒体内容进行编码、加密或格式转换时，就需要创建一个媒体处理器。

下表提供了每个可用媒体处理器的名称和说明。

<table border="2" cellspacing="0" cellpadding="5" style="border: 2px solid #000000;">
  <thead>
    <tr>
       <th>媒体处理器名称</th>
       <th>说明</th>
	<th>更多信息</th>
    </tr>
  </thead>
  <tbody>
    <tr>
       <td>Azure 媒体编码器</td>
       <td>让你使用 Azure 媒体编码器运行编码任务。</td>
       <td><a href="/documentation/articles/media-services-dotnet-encoding-units/">Azure 媒体编码器的任务预设字符串</a></td>
    </tr>   
	<tr>
        <td>Azure 媒体索引器</td>
        <td>使媒体文件和内容可搜索，以及生成隐藏字幕跟踪和关键字。</td>
		<td><a href="/documentation/articles/media-services-index-content/">使用Azure 媒体索引器为媒体文件编制索引</a>。</td>
    </tr>
    <tr>
        <td>Windows Azure 媒体包装器</td>
        <td>让你将媒体资产从 .mp4 格式转换为平滑流式处理格式。还可让你将媒体资产从平滑流式处理格式转换为 Apple HTTP 实时流 (HLS) 格式。</td>
		<td><a href="http://msdn.microsoft.com/zh-cn/library/hh973635.aspx">Azure Media Packager 的任务预设字符串</a></td>
    </tr>
    <tr>
        <td>Azure 媒体加密器</td>
        <td>让你使用 PlayReady 保护加密媒体资产。</td>
        <td><a href="http://msdn.microsoft.com/zh-cn/library/hh973610.aspx">Azure 媒体包装器的任务预设字符串</a></td>
    </tr>
	<tr>
		<td>Azure Media Hyperlapse（预览）</td>
		<td>使你能够通过视频防抖动功能消除视频中的“晃动”。也可使将内容制作为可用剪辑的速度加快。</td>
		<td><a href="http://go.microsoft.com/fwlink/?LinkId=613274">Azure Media Hyperlapse</a></td>
	</tr>
    <tr>
        <td>存储解密</td>
        <td>让你解密使用存储加密技术加密的媒体资产。</td>
		<td>不适用</td>
    </tr>  </tbody>
</table>

<br />

##获取 MediaProcessor

以下方法演示了如何获取媒体处理器实例。该代码示例假设使用名为 **\_context** 的模块级变量来引用[如何：以编程方式连接到媒体服务]部分中描述的服务器上下文。

	private static IMediaProcessor GetLatestMediaProcessorByName(string mediaProcessorName)
	{
	     var processor = _context.MediaProcessors.Where(p => p.Name == mediaProcessorName).
	        ToList().OrderBy(p => new Version(p.Version)).LastOrDefault();
	
	    if (processor == null)
	        throw new ArgumentException(string.Format("Unknown media processor", mediaProcessorName));
	
	    return processor;
	}

##后续步骤
了解如何获取媒体处理器实例后，请转到[如何对资产进行编码][]主题，其中说明了如何使用 Azure 媒体编码器对资产进行编码。

[如何对资产进行编码]: /documentation/articles/media-services-encode-asset
[Task Preset Strings for the Azure Media Encoder]: http://msdn.microsoft.com/zh-cn/library/jj129582.aspx
[如何：以编程方式连接到媒体服务]: /documentation/articles/media-services-set-up-computer

<!---HONumber=74-->