<properties 
	pageTitle="如何创建媒体处理器 | Azure" 
	description="了解如何创建一个媒体处理器组件用来为 Azure 媒体服务编码、转换格式、加密或解密媒体内容。代码示例用 C# 编写且使用适用于 .NET 的媒体服务 SDK。" 
	services="media-services" 
	documentationCenter="" 
	authors="juliako" 
	manager="dwrede" 
	editor=""/>

<tags
	ms.service="media-services"
	ms.date="03/01/2016" 
	wacn.date="04/05/2016"/>


#如何：获取媒体处理器实例

> [AZURE.SELECTOR]
- [.NET](/documentation/articles/media-services-get-media-processor/)
- [REST](/documentation/articles/media-services-rest-get-media-processor/)
 

##概述

在媒体服务中，媒体处理器是完成特定处理任务（例如，对媒体内容进行编码、格式转换、加密或解密）的组件。通常，当你创建一个任务以便对媒体内容进行编码、加密或格式转换时，就需要创建一个媒体处理器。

下表提供了每个可用媒体处理器的名称和说明。

媒体处理器名称|说明|更多信息
---|---|---
媒体编码器标准版|为按需编码提供标准功能。 |[简要介绍并比较 Azure 按需媒体编码器](/documentation/articles/media-services-encode-asset/)
媒体编码器高级工作流|允许你使用媒体编码器高级工作流运行编码任务。|[简要介绍并比较 Azure 按需媒体编码器](/documentation/articles/media-services-encode-asset/)
Azure Media Indexer| 使媒体文件和内容可搜索，以及生成隐藏字幕跟踪和关键字。|[Azure Media Indexer](/documentation/articles/media-services-index-content/)
Azure Media Hyperlapse（预览）|使你能够通过视频防抖动功能消除视频中的“晃动”。也可使将内容制作为可用剪辑的速度加快。|[Azure Media Hyperlapse](/documentation/articles/media-services-hyperlapse-content/)
Azure Media Encoder|已过时
存储解密| 已过时|
Azure 媒体包装器|已过时|
Azure 媒体加密器|已过时|

##获取媒体处理器

以下方法演示了如何获取媒体处理器实例。该代码示例假设使用名为 **\_context** 的模块级变量来引用[如何：以编程方式连接到媒体服务](/documentation/articles/media-services-dotnet-connect_programmatically/)部分中描述的服务器上下文。

	private static IMediaProcessor GetLatestMediaProcessorByName(string mediaProcessorName)
	{
		var processor = _context.MediaProcessors.Where(p => p.Name == mediaProcessorName).
		ToList().OrderBy(p => new Version(p.Version)).LastOrDefault();
		
		if (processor == null)
		throw new ArgumentException(string.Format("Unknown media processor", mediaProcessorName));
		
		return processor;
	}



##后续步骤

了解如何获取媒体处理器实例后，请转到[如何对资产进行编码](/documentation/articles/media-services-dotnet-encode-with-media-encoder-standard/)主题，其中说明了如何使用媒体编码器标准版对资产进行编码。



<!---HONumber=Mooncake_0328_2016-->