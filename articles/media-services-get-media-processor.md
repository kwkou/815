<properties linkid="develop-media-services-how-to-guides-create-media-processor" urlDisplayName="Create a Media Processor" pageTitle="How to Create a Media Processor - Azure" metaKeywords="" description="Learn how to create a media processor component to encode, convert format, encrypt, or decrypt media content for Azure Media Services. Code samples are written in C# and use the Media Services SDK for .NET." metaCanonical="" services="media-services" documentationCenter="" title="How to: Get a Media Processor Instance" authors="migree" solutions="" manager="" editor="" />

如何：获取媒体处理器实例
========================

本文是介绍 Azure Media Services 编程的系列主题中的一篇。前一个主题是[如何：创建加密的资产并上载到存储中](http://go.microsoft.com/fwlink/?LinkID=301733&clcid=0x409)。

在 Media Services 中，媒体处理器是完成特定处理任务（例如，对媒体内容进行编码、格式转换、加密或解密）的组件。通常，当你创建一个任务以便对媒体内容进行编码、加密或格式转换时，就需要创建一个媒体处理器。

下表提供了每个可用媒体处理器的名称和说明。

<table data-morhtml="true" border="2" cellspacing="0" cellpadding="5" style="border: 2px solid #000000;">
  <thead data-morhtml="true">
    <tr data-morhtml="true">
<th data-morhtml="true">媒体处理器名称</th>
<th data-morhtml="true">说明</th>
	<th data-morhtml="true">更多信息</th>
    </tr>
  </thead>
  <tbody data-morhtml="true">
    <tr data-morhtml="true">
<td data-morhtml="true">Azure 媒体编码器</td>
<td data-morhtml="true">让你使用媒体编码器运行编码任务。</td>
<td data-morhtml="true"><a data-morhtml="true" href="http://msdn.microsoft.com/zh-cn/library/jj129582.aspx"> Azure 媒体编码器的任务预设字符串</a></td>
    </tr>
    <tr data-morhtml="true">
<td data-morhtml="true">Azure 媒体包装器</td>
<td data-morhtml="true">让你将媒体资产从 .mp4 格式转换为平滑流式处理格式。还可让你将媒体资产从平滑流式处理格式转换为 Apple HTTP 实时流 (HLS) 格式。</td>
		<td data-morhtml="true"><a data-morhtml="true" href="http://msdn.microsoft.com/zh-cn/library/hh973635.aspx">Azure 媒体包装器的任务预设字符串</a></td>
    </tr>
    <tr data-morhtml="true">
<td data-morhtml="true">Azure 媒体加密器</td>
<td data-morhtml="true">让你使用 PlayReady 保护加密媒体资产。</td>
<td data-morhtml="true"><a data-morhtml="true" href="http://msdn.microsoft.com/zh-cn/library/hh973610.aspx">Azure 媒体包装器的任务预设字符串</a></td>
    </tr>
    <tr data-morhtml="true">
<td data-morhtml="true">存储解密</td>
<td data-morhtml="true">让你解密使用存储加密技术加密的媒体资产。</td>
		<td data-morhtml="true">不适用</td>
    </tr>
  </tbody>
</table>

以下方法演示了如何获取媒体处理器实例。该代码示例假设使用名为 **\_context** 的模块级变量来引用[如何：以编程方式连接到 Media Services](http://www.windowsazure.com/zh-cn/develop/media-services/how-to-guides/set-up-computer-for-media-services) 部分中所述的服务器上下文。

``` {}
private static IMediaProcessor GetLatestMediaProcessorByName(string mediaProcessorName)
{
var processor = _context.MediaProcessors.Where(p => p.Name == mediaProcessorName).
ToList().OrderBy(p => new Version(p.Version)).LastOrDefault();

if (processor == null)
throw new ArgumentException(string.Format("Unknown media processor", mediaProcessorName));

return processor;
}
```

后续步骤
--------

了解如何获取媒体处理器实例后，请转到[如何为资产编码](http://go.microsoft.com/fwlink/?LinkId=301753)主题，其中说明了如何使用 Azure 媒体编码器对资产进行编码。

