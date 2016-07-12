<properties 
	pageTitle="使用 Azure 媒体服务 REST API 创建筛选器" 
	description="本主题介绍如何创建筛选器，以便客户端能够使用它们来流式传输流的特定部分。媒体服务将创建动态清单来存档此选择性流。"
	services="media-services" 
	documentationCenter="" 
	authors="Juliako,cenkdin" 
	manager="dwrede" 
	editor=""/>

<tags
	ms.service="media-services"
 	ms.date="04/18/2016"  
	wacn.date="06/27/2016"/>

#使用 Azure 媒体服务 REST API 创建筛选器

> [AZURE.SELECTOR]
- [.NET](/documentation/articles/media-services-dotnet-dynamic-manifest/)
- [REST](/documentation/articles/media-services-rest-dynamic-manifest/)


从 2.11 版开始，媒体服务可让你为资产定义筛选器。这些筛选器是服务器端规则，可让你的客户选择运行如下操作：只播放一段视频（而非播放完整视频），或只指定客户设备可以处理的一部分音频和视频再现内容（而非与该资产相关的所有再现内容）。通过按客户请求创建的**动态清单**可以实现对资产进行这种筛选，并基于指定的筛选器流式传输视频。

有关与筛选器和动态清单相关的更多详细信息，请参阅[动态清单概述](/documentation/articles/media-services-dynamic-manifest-overview/)。

本主题说明如何使用 REST API 创建、更新和删除筛选器。

##用于创建筛选器的类型

创建筛选器时将使用以下类型：

- [筛选器](http://msdn.microsoft.com/zh-cn/library/azure/mt149056.aspx)
- [AssetFilter](http://msdn.microsoft.com/zh-cn/library/azure/mt149053.aspx)
- [PresentationTimeRange](http://msdn.microsoft.com/zh-cn/library/azure/mt149052.aspx)
- [FilterTrackSelect 和 FilterTrackPropertyCondition](http://msdn.microsoft.com/zh-cn/library/azure/mt149055.aspx)



>[AZURE.NOTE]使用媒体服务 REST API 时，需注意以下事项：<br />
>访问媒体服务中的实体时，必须在 HTTP 请求中设置特定标头字段和值。有关详细信息，请参阅[媒体服务 REST API 开发的设置](/documentation/articles/media-services-rest-how-to-use/)。<br />
>在成功连接到 https://media.chinacloudapi.cn 之后，你将接收到指定另一个媒体服务 URI 的 301 重定向。必须根据[使用 REST API 连接到媒体服务](/documentation/articles/media-services-rest-connect_programmatically/)中所述对新的 URI 执行后续调用。


##创建筛选器

###创建全局筛选器

若要创建全局筛选器，请使用以下 HTTP 请求：

####HTTP 请求

请求标头

	POST https://media.chinacloudapi.cn/API/Filters HTTP/1.1 
	DataServiceVersion:3.0 
	MaxDataServiceVersion: 3.0 
	Content-Type: application/json 
	Accept: application/json 
	Accept-Charset: UTF-8 
	Authorization: Bearer <token value> 
	x-ms-version: 2.11 
	x-ms-client-request-id: 00000000-0000-0000-0000-000000000000 
	Host:media.chinacloudapi.cn 

请求正文

	{  
	   "Name":"GlobalFilter",
	   "PresentationTimeRange":{  
	      "StartTimestamp":"0",
	      "EndTimestamp":"9223372036854775807",
	      "PresentationWindowDuration":"12000000000",
	      "LiveBackoffDuration":"0",
	      "Timescale":"10000000"
	   },
	   "Tracks":[  
	      {  
	         "PropertyConditions":
	              [  
	            {  
	               "Property":"Type",
	               "Value":"audio",
	               "Operator":"Equal"
	            },
	            {  
	               "Property":"Bitrate",
	               "Value":"0-2147483647",
	               "Operator":"Equal"
	            }
	         ]
	      }
	   ]
	}




####HTTP 响应
	
	HTTP/1.1 201 Created 

###创建本地 AssetFilter

若要创建本地 AssetFilter，请使用以下 HTTP 请求：

####HTTP 请求

请求标头

	POST https://media.chinacloudapi.cn/API/AssetFilters HTTP/1.1 
	DataServiceVersion: 3.0 
	MaxDataServiceVersion: 3.0 
	Content-Type: application/json 
	Accept: application/json 
	Accept-Charset: UTF-8 
	Authorization: Bearer <token value> 
	x-ms-version: 2.11 
	x-ms-client-request-id: 00000000-0000-0000-0000-000000000000 
	Host: media.chinacloudapi.cn  

请求正文

	{   
	   "Name":"AssetFilter", 
	   "ParentAssetId":"nb:cid:UUID:536e555d-1500-80c3-92dc-f1e4fdc6c592", 
	   "PresentationTimeRange":{   
	      "StartTimestamp":"0", 
	      "EndTimestamp":"9223372036854775807", 
	      "PresentationWindowDuration":"12000000000", 
	      "LiveBackoffDuration":"0", 
	      "Timescale":"10000000" 
	   }, 
	   "Tracks":[   
	      {   
	         "PropertyConditions": 
	              [   
	            {   
	               "Property":"Type", 
	               "Value":"audio", 
	               "Operator":"Equal" 
	            }, 
	            {   
	               "Property":"Bitrate", 
	               "Value":"0-2147483647", 
	               "Operator":"Equal" 
	            } 
	         ] 
	      } 
	   ] 
	} 

####HTTP 响应 

	HTTP/1.1 201 Created 
	. . . 

##列出筛选器

###获取 AMS 帐户中的所有全局**筛选器**

若要列出筛选器，请使用以下 HTTP 请求：

####HTTP 请求
	 
	GET https://media.chinacloudapi.cn/API/Filters HTTP/1.1 
	DataServiceVersion:3.0 
	MaxDataServiceVersion: 3.0 
	Accept: application/json 
	Accept-Charset: UTF-8 
	Authorization: Bearer <token value> 
	x-ms-version: 2.11 
	Host: media.chinacloudapi.cn 
	
### 获取与资产关联的 **AssetFilter**。

####HTTP 请求

	GET https://media.chinacloudapi.cn/API/Assets('nb%3Acid%3AUUID%3A536e555d-1500-80c3-92dc-f1e4fdc6c592')/AssetFilters HTTP/1.1 
	DataServiceVersion: 3.0 
	MaxDataServiceVersion: 3.0 
	Accept: application/json 
	Accept-Charset: UTF-8 
	Authorization: Bearer <token value> 
	x-ms-version: 2.11 
	x-ms-client-request-id: 00000000-0000-0000-0000-000000000000 
	Host: media.chinacloudapi.cn 

###基于 ID 获取 **AssetFilter**

####HTTP 请求

	GET https://media.chinacloudapi.cn/API/AssetFilters('nb%3Acid%3AUUID%3A536e555d-1500-80c3-92dc-f1e4fdc6c592__%23%23%23__TestFilter') HTTP/1.1 
	DataServiceVersion: 3.0 
	MaxDataServiceVersion: 3.0 
	Accept: application/json 
	Accept-Charset: UTF-8 
	Authorization: Bearer <token value> 
	x-ms-version: 2.11 
	x-ms-client-request-id: 00000000


##更新筛选器
 
使用 PATCH、PUT 或 MERGE 并结合新的属性值来更新筛选器。有关这些操作的详细信息，请参阅 [PATCH、PUT、MERGE](http://msdn.microsoft.com/zh-cn/library/dd541276.aspx)。
 
如果你更新筛选器，则流式处理终结点需要 2 分钟的时间来刷新规则。如果内容是通过使用此筛选器提供的（并在代理和 CDN 缓存中缓存），则更新此筛选器会导致播放器失败。建议在更新筛选器之后清除缓存。如果此选项不可用，请考虑使用其他筛选器。
 
###更新全局筛选器

若要更新全局筛选器，请使用以下 HTTP 请求：

####HTTP 请求
 
请求标头：

	MERGE https://media.chinacloudapi.cn/API/Filters('filterName') HTTP/1.1 
	DataServiceVersion:3.0 
	MaxDataServiceVersion: 3.0 
	Content-Type: application/json 
	Accept: application/json 
	Accept-Charset: UTF-8 
	Authorization: Bearer <token value> 
	x-ms-version: 2.11 
	x-ms-client-request-id: 00000000-0000-0000-0000-000000000000 
	Host: media.chinacloudapi.cn 
	Content-Length: 384
	
请求正文：
	
	{ 
	   "Tracks":[   
	      {   
	         "PropertyConditions": 
	         [   
	            {   
	               "Property":"Type", 
	               "Value":"audio", 
	               "Operator":"Equal" 
	            }, 
	            {   
	               "Property":"Bitrate", 
	               "Value":"0-2147483647", 
	               "Operator":"Equal" 
	            } 
	         ] 
	      } 
	   ] 
	} 

###更新本地 AssetFilter

若要更新本地筛选器，请使用以下 HTTP 请求：

####HTTP 请求

请求标头：

	MERGE https://media.chinacloudapi.cn/API/AssetFilters('nb%3Acid%3AUUID%3A536e555d-1500-80c3-92dc-f1e4fdc6c592__%23%23%23__TestFilter')  HTTP/1.1 
	DataServiceVersion: 3.0 
	MaxDataServiceVersion: 3.0 
	Content-Type: application/json 
	Accept: application/json 
	Accept-Charset: UTF-8 
	Authorization: Bearer <token value> 
	x-ms-version: 2.11 
	x-ms-client-request-id: 00000000-0000-0000-0000-000000000000 
	Host: media.chinacloudapi.cn 
	
请求正文：
	
	{ 
	   "Tracks":[   
	      {   
	         "PropertyConditions": 
	         [   
	            {   
	               "Property":"Type", 
	               "Value":"audio", 
	               "Operator":"Equal" 
	            }, 
	            {   
	               "Property":"Bitrate", 
	               "Value":"0-2147483647", 
	               "Operator":"Equal" 
	            } 
	         ] 
	      } 
	   ] 
	} 


##删除筛选器


###删除全局筛选器

若要删除全局筛选器，请使用以下 HTTP 请求：
	
####HTTP 请求

	DELETE https://media.chinacloudapi.cn/api/Filters('GlobalFilter') HTTP/1.1 
	DataServiceVersion:3.0 
	MaxDataServiceVersion: 3.0 
	Accept: application/json 
	Accept-Charset: UTF-8 
	Authorization: Bearer <token value> 
	x-ms-version: 2.11 
	Host: media.chinacloudapi.cn 


###删除本地 AssetFilter

若要删除本地 AssetFilter，请使用以下 HTTP 请求：

####HTTP 请求

	DELETE https://media.chinacloudapi.cn/API/AssetFilters('nb%3Acid%3AUUID%3A536e555d-1500-80c3-92dc-f1e4fdc6c592__%23%23%23__LocalFilter') HTTP/1.1 
	DataServiceVersion: 3.0 
	MaxDataServiceVersion: 3.0 
	Accept: application/json 
	Accept-Charset: UTF-8 
	Authorization: Bearer <token value> 
	x-ms-version: 2.11 
	Host: media.chinacloudapi.cn 

##生成使用筛选器的流 URL

有关如何发布和传送资产的信息，请参阅[将内容传送到客户概述](/documentation/articles/media-services-deliver-content-overview/)。


以下示例演示了如何将筛选器添加到流 URL。

**MPEG DASH**

	http://testendpoint-testaccount.streaming.mediaservices.chinacloudapi.cn/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=mpd-time-csf, filter=MyFilter)

**Apple HTTP 实时流 (HLS) V4**

	http://testendpoint-testaccount.streaming.mediaservices.chinacloudapi.cn/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=m3u8-aapl, filter=MyFilter)

**Apple HTTP 实时流 (HLS) V3**

	http://testendpoint-testaccount.streaming.mediaservices.chinacloudapi.cn/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=m3u8-aapl-v3, filter=MyFilter)

**平滑流**

	http://testendpoint-testaccount.streaming.mediaservices.chinacloudapi.cn/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(filter=MyFilter)


**HDS**

	http://testendpoint-testaccount.streaming.mediaservices.chinacloudapi.cn/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=f4m-f4f, filter=MyFilter)


##另请参阅 

[动态清单概述](/documentation/articles/media-services-dynamic-manifest-overview/)
 

 

<!---HONumber=Mooncake_0620_2016-->