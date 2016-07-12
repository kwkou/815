<properties 
	pageTitle="如何创建媒体处理器 | Azure" 
	description="了解如何创建一个媒体处理器组件用来为 Azure 媒体服务编码、转换格式、加密或解密媒体内容。" 
	services="media-services" 
	documentationCenter="" 
	authors="Juliako" 
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

##获取 MediaProcessor

>[AZURE.NOTE] 使用媒体服务 REST API 时，需注意以下事项：
>
>访问媒体服务中的实体时，必须在 HTTP 请求中设置特定标头字段和值。有关详细信息，请参阅[媒体服务 REST API 开发的设置](/documentation/articles/media-services-rest-how-to-use/)。

>在成功连接到 https://media.chinacloudapi.cn 之后，你将接收到指定另一个媒体服务 URI 的 301 重定向。必须按[使用 REST API 连接到媒体服务](/documentation/articles/media-services-rest-connect_programmatically/)中所述对新的 URI 执行后续调用。


以下 REST 调用演示了如何按名称获取媒体处理器实例（在本例中为媒体编码器标准版）。



	
请求：

	GET https://media.chinacloudapi.cn/api/MediaProcessors()?$filter=Name%20eq%20'Media%20Encoder%20Standard' HTTP/1.1
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	User-Agent: Microsoft ADO.NET Data Services
	Authorization: Bearer <token>
	x-ms-version: 2.11
	Host: media.chinacloudapi.cn
	
响应：
		
	{  
	   "odata.metadata":"https://media.chinacloudapi.cn/api/$metadata#MediaProcessors",
	   "value":[  
	      {  
	         "Id":"nb:mpid:UUID:ff4df607-d419-42f0-bc17-a481b1331e56",
	         "Description":"Media Encoder Standard",
	         "Name":"Media Encoder Standard",
	         "Sku":"",
	         "Vendor":"Microsoft",
	         "Version":"1.1"
	      }
	   ]
	}




##后续步骤

了解如何获取媒体处理器实例后，请转到[如何对资产进行编码](/documentation/articles/media-services-rest-get-started/)主题，其中说明了如何使用媒体编码器标准版对资产进行编码。

<!---HONumber=Mooncake_0328_2016-->