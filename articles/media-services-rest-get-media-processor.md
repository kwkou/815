<properties 
	pageTitle="如何创建媒体处理器 - Azure" 
	description="了解如何创建一个媒体处理器组件用来为 Azure 媒体服务编码、转换格式、加密或解密媒体内容。" 
	services="media-services" 
	documentationCenter="" 
	authors="Juliako" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="media-services" 
	ms.date="09/07/2015" 
	wacn.date="11/02/2015"/>


#如何：获取媒体处理器实例


> [AZURE.SELECTOR]
- [.NET](/documentation/articles/media-services-get-media-processor)
- [REST](/documentation/articles/media-services-rest-get-media-processor)

##概述

在媒体服务中，媒体处理器是完成特定处理任务（例如，对媒体内容进行编码、格式转换、加密或解密）的组件。通常，当你创建一个任务以便对媒体内容进行编码、加密或格式转换时，就需要创建一个媒体处理器。

下表提供了每个可用媒体处理器的名称和说明。

媒体处理器名称|说明|更多信息
---|---|---
Azure Media Encoder|让你使用 Azure 媒体编码器运行编码任务。|[Azure Media Encoder](/documentation/articles/media-services-encode-asset#azure_media_encoder)
媒体编码器标准版|让你使用媒体编码器标准版运行编码任务。|[Azure Media Encoder](/documentation/articles/media-services-encode-asset#media_encoder_standard)
Azure Media Indexer| 使媒体文件和内容可搜索，以及生成隐藏字幕跟踪和关键字。|[使用 Azure Media Indexer 为媒体文件编制索引](/documentation/articles/media-services-index-content)。
Azure Media Hyperlapse（预览）|使你能够通过视频防抖动功能消除视频中的“晃动”。也可使将内容制作为可用剪辑的速度加快。|		[Azure Media Hyperlapse](http://azure.microsoft.com/blog/?p=286281&preview=1&_ppp=61e1a0b3db)</a>
存储解密| 让你解密使用存储加密技术加密的媒体资产。|不适用
Windows Azure Media Packager|让你将媒体资产从 .mp4 格式转换为平滑流式处理格式。还可让你将媒体资产从平滑流式处理格式转换为 Apple HTTP 实时流 (HLS) 格式。|[Azure Media Packager 的任务预设字符串](http://msdn.microsoft.com/zh-cn/library/hh973635.aspx)
Windows Azure Media Encryptor|让你使用 PlayReady 保护加密媒体资产。|[Azure Media Packager 的任务预设字符串](http://msdn.microsoft.com/zh-cn/library/hh973610.aspx)

##获取 MediaProcessor

>[AZURE.NOTE]使用媒体服务 REST API 时，需注意以下事项：
>
>访问媒体服务中的实体时，必须在 HTTP 请求中设置特定标头字段和值。有关详细信息，请参阅[媒体服务 REST API 开发的设置](/documentation/articles/media-services-rest-how-to-use)。

>在成功连接到 https://media.chinacloudapi.cn 之后，你将接收到指定另一个媒体服务 URI 的 301 重定向。必须根据[使用 REST API 连接到媒体服务](/documentation/articles/media-services-rest-connect_programmatically/)中所述对新的 URI 执行后续调用。



以下 REST 调用演示了如何按名称获取媒体处理器实例（在本例中为 **Azure 媒体编码器**）。

	
请求：

	GET https://media.chinacloudapi.cn/api/MediaProcessors()?$filter=Name%20eq%20'Azure%20Media%20Encoder' HTTP/1.1
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	User-Agent: Microsoft ADO.NET Data Services
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=juliakoams1&urn%3aSubscriptionId=bbbef702-e769-477b-9f16-bc4d3aa97387&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1423635565&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.chinacloudapi.cn%2f&HMACSHA256=6zwXEn7YJzVJbVCNpqDUjBLuE5iUwsdJbWvJNvpY3%2b8%3d
	x-ms-version: 2.11
	Host: media.chinacloudapi.cn
	
响应：
	
	HTTP/1.1 200 OK
	Cache-Control: no-cache
	Content-Length: 273
	Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
	Server: Microsoft-IIS/8.5
	x-ms-client-request-id: 8a291764-4ed7-405d-aa6e-d3ebabb0b3f6
	request-id: dceeb559-48b5-48e1-81d3-d324b6203d51
	x-ms-request-id: dceeb559-48b5-48e1-81d3-d324b6203d51
	X-Content-Type-Options: nosniff
	DataServiceVersion: 3.0;
	X-Powered-By: ASP.NET
	Strict-Transport-Security: max-age=31536000; includeSubDomains
	Date: Wed, 11 Feb 2015 00:19:56 GMT
	
	{"odata.metadata":"https://wamsbayclus001rest-hs.chinacloudapp.cn/api/$metadata#MediaProcessors","value":[{"Id":"nb:mpid:UUID:1b1da727-93ae-4e46-a8a1-268828765609","Description":"Azure Media Encoder","Name":"Azure Media Encoder","Sku":"","Vendor":"Microsoft","Version":"4.4"}]}


##后续步骤
了解如何获取媒体处理器实例后，请转到[如何对资产进行编码][]主题，其中说明了如何使用 Azure 媒体编码器对资产进行编码。

[如何对资产进行编码]: /documentation/articles/media-services-rest-encode-asset
[Task Preset Strings for the Azure Media Encoder]: http://msdn.microsoft.com/zh-cn/library/jj129582.aspx
[How to: Connect to Media Services Programmatically]: /documentation/articles/media-services-rest-connect_programmatically

<!---HONumber=76-->