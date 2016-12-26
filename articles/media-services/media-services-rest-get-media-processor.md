<properties 
	pageTitle="如何创建媒体处理器 | Azure" 
	description="了解如何创建一个媒体处理器组件，对 Azure 媒体服务的媒体内容进行编码、格式转换、加密或解密。" 
	services="media-services" 
	documentationCenter="" 
	authors="Juliako" 
	manager="erikre" 
	editor=""/>  


<tags 
	ms.service="media-services" 
	ms.workload="media" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="09/26/2016" 
	wacn.date="12/26/2016" 
	ms.author="juliako"/>


#如何：获取媒体处理器实例


> [AZURE.SELECTOR]
- [.NET](/documentation/articles/media-services-get-media-processor/)
- [REST](/documentation/articles/media-services-rest-get-media-processor/)

##概述

在媒体服务中，媒体处理器是处理特定处理任务（例如，对媒体内容进行编码、格式转换、加密或解密）的组件。通常，当你创建一个任务以对媒体内容进行编码、加密或格式转换时，需要创建一个媒体处理器。

下表提供了每个可用媒体处理器的名称和说明。

媒体处理器名称|说明|详细信息
---|---|---
Media Encoder Standard|为按需编码提供标准功能。 |[简要介绍并比较 Azure 按需媒体编码器](/documentation/articles/media-services-encode-asset/)
Azure Media Indexer| 使媒体文件和内容可搜索，以及生成隐藏式字幕跟踪和关键字。|[Azure Media Indexer](/documentation/articles/media-services-index-content/)。
Azure Media Hyperlapse（预览版）|使你能够通过视频防抖动功能消除视频中的“晃动”。也可使将内容制作为可用剪辑的速度加快。|[Azure Media Hyperlapse](/documentation/articles/media-services-hyperlapse-content/)
Azure Media Encoder|已过时
存储解密| 已过时|
Azure 媒体包装器|已过时|
Azure 媒体加密器|已过时|

##获取 MediaProcessor

>[AZURE.NOTE] 使用媒体服务 REST API 时，需注意以下事项：
>
>访问媒体服务中的实体时，必须在 HTTP 请求中设置特定标头字段和值。有关详细信息，请参阅[媒体服务 REST API 开发的设置](/documentation/articles/media-services-rest-how-to-use/)。

>请按照[使用 REST API 连接到媒体服务](/documentation/articles/media-services-rest-connect-programmatically/)中所述对媒体服务 URI 执行后续调用。


以下 REST 调用演示了如何按名称获取媒体处理器实例（在本例中为 **Media Encoder Standard**）。



	
请求：

	GET https://wamsshaclus001rest-hs.chinacloudapp.cn/api/MediaProcessors()?$filter=Name%20eq%20'Media%20Encoder%20Standard' HTTP/1.1
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	User-Agent: Microsoft ADO.NET Data Services
	Authorization: Bearer <token>
	x-ms-version: 2.11
	Host: wamsshaclus001rest-hs.chinacloudapp.cn
	
响应：
		
	{  
	   "odata.metadata":"https://wamsshaclus001rest-hs.chinacloudapp.cn/api/$metadata#MediaProcessors",
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

现在已了解如何获取媒体处理器实例，请转到[如何对资产进行编码](/documentation/articles/media-services-rest-get-started/)主题，其中说明了如何使用 Media Encoder Standard 对资产进行编码。

<!---HONumber=Mooncake_Quality_Review_1215_2016-->