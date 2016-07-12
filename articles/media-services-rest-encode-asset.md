<properties 
	pageTitle="如何使用媒体编码器标准版对资产进行编码" 
	description="了解如何使用媒体编码器标准版为媒体服务上的媒体内容编码。代码示例使用 REST API。" 
	services="media-services" 
	documentationCenter="" 
	authors="Juliako" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="media-services" 
	ms.date="03/01/2016" 
	wacn.date="04/05/2016"/>


#如何使用媒体编码器标准版对资产进行编码


> [AZURE.SELECTOR]
- [.NET](/documentation/articles/media-services-dotnet-encode-asset/)
- [REST](/documentation/articles/media-services-rest-encode-asset/)
- [门户](/documentation/articles/media-services-manage-content/#encode)

##概述
要通过 Internet 传送数字视频，你必须对媒体进行压缩。数字视频文件相当大，可能因过大而无法通过 Internet 传送或者无法在你客户的设备上正常显示。编码是压缩视频和音频以便你的客户能够查看媒体的过程。

编码作业是媒体服务中最常见的处理操作之一。可通过创建编码作业将媒体文件从一种编码转换为另一种编码。编码时，可以使用媒体服务的内置编码器（媒体编码器标准版）。你也可以使用媒体服务合作伙伴提供的编码器；可通过 Azure 应用商店获取第三方编码器。可以使用为编码器定义的预设字符串或预设配置文件来指定编码任务的详细信息。若要查看可用预设的类型，请参阅[媒体编码器标准版的任务预设](http://msdn.microsoft.com/zh-cn/library/mt269960)。

每个作业可以有一个或多个任务，具体因要完成的处理类型而异。REST API 允许你通过以下两种方式之一创建作业及相关任务：

- 可按以下两种方式以内联形式定义任务：通过作业实体上的任务导航属性，或 
- 通过 OData 批处理。
  

建议始终将夹层文件编码为自适应比特率 MP4 集，然后使用[动态打包](/documentation/articles/media-services-dynamic-packaging-overview/)将该集转换为所需的格式。若要利用动态打包，首先必须获取你计划从中传送内容的流式处理终结点的至少一个点播流单元。有关详细信息，请参阅[如何缩放媒体服务](/documentation/articles/media-services-manage-origins/#scale_streaming_endpoints)。

如果你的输出资产已经过存储加密，则必须配置资产传送策略。有关详细信息，请参阅[配置资产传送策略](/documentation/articles/media-services-rest-configure-asset-delivery-policy/)。


>[AZURE.NOTE]在开始引用媒体处理器之前，请确认你的媒体处理器 ID 准确无误。有关详细信息，请参阅[获取媒体处理器](/documentation/articles/media-services-rest-get-media-processor/)。

##创建包含单个编码任务的作业 

>[AZURE.NOTE] 使用媒体服务 REST API 时，需注意以下事项：
>
>访问媒体服务中的实体时，必须在 HTTP 请求中设置特定标头字段和值。有关详细信息，请参阅[媒体服务 REST API 开发的设置](/documentation/articles/media-services-rest-how-to-use/)。

>在成功连接到 https://media.chinacloudapi.cn 之后，你将接收到指定另一个媒体服务 URI 的 301 重定向。必须按[使用 REST API 连接到媒体服务](/documentation/articles/media-services-rest-connect_programmatically/)中所述对新的 URI 执行后续调用。
>
>使用 JSON 并指定在请求中使用 **__metadata** 关键字（例如，为了引用某个链接对象）时，必须将 Accept 标头设置为 [JSON 详细格式](http://www.odata.org/documentation/odata-version-3-0/json-verbose-format/)：Accept: application/json;odata=verbose。

以下示例说明了如何使用一个任务集来创建和发布一个作业，从而以特定分辨率和质量来编码某个视频。使用媒体编码器标准版编码时，可以使用[此处](http://msdn.microsoft.com/zh-cn/library/mt269960)指定的任务配置预设。
	
请求：

	POST https://media.chinacloudapi.cn/API/Jobs HTTP/1.1
	Content-Type: application/json;odata=verbose
	Accept: application/json;odata=verbose
	DataServiceVersion: 3.0
	MaxDataServiceVersion: 3.0
	x-ms-version: 2.11
	Authorization: Bearer <token value>
	x-ms-client-request-id: 00000000-0000-0000-0000-000000000000
	Host: media.chinacloudapi.cn

	
	{"Name" : "NewTestJob", "InputMediaAssets" : [{"__metadata" : {"uri" : "https://media.chinacloudapi.cn/api/Assets('nb%3Acid%3AUUID%3Aaab7f15b-3136-4ddf-9962-e9ecb28fb9d2')"}}],  "Tasks" : [{"Configuration" : "H264 Multiple Bitrate 720p", "MediaProcessorId" : "nb:mpid:UUID:ff4df607-d419-42f0-bc17-a481b1331e56",  "TaskBody" : "<?xml version=\"1.0\" encoding=\"utf-8\"?><taskBody><inputAsset>JobInputAsset(0)</inputAsset><outputAsset>JobOutputAsset(0)</outputAsset></taskBody>"}]}

响应：
	
	HTTP/1.1 201 Created

	. . . 

###设置输出资产的名称

以下示例说明了如何设置 assetName 属性：

	{ "TaskBody" : "<?xml version=\"1.0\" encoding=\"utf-8\"?><taskBody><inputAsset>JobInputAsset(0)</inputAsset><outputAsset assetName=\"CustomOutputAssetName\">JobOutputAsset(0)</outputAsset></taskBody>"}

##注意事项

- TaskBody 属性必须使用文本 XML 来定义将由任务使用的输入资产或输出资产的数量。任务主题包含 XML 的 XML 架构定义。
- 在 TaskBody 定义中，必须将 <inputAsset> 和 <outputAsset> 的每个内部值设置为 JobInputAsset(value) 或 JobOutputAsset(value)。
- 一个任务可以有多个输出资产。作为作业任务的输出，一个 JobOutputAsset(x) 只能使用一次。
- 可以将 JobInputAsset 或 JobOutputAsset 指定为某任务的输入资产。
- 任务不得构成循环。
- 传递给 JobInputAsset 或 JobOutputAsset 的 value 参数代表资产的索引值。实际资产在作业实体定义的 InputMediaAssets 和 OutputMediaAssets 导航属性中定义。 
- 由于媒体服务基于 OData v3，因此 InputMediaAssets 和 OutputMediaAssets 导航属性集合中的单个资产将通过“\_\_metadata : uri”名称-值对。
- InputMediaAssets 将映射到已在媒体服务中创建的一个或多个资产。OutputMediaAssets 由系统创建。它们不引用现有资产。
- OutputMediaAssets 可以使用 assetName 属性来命名。如果该属性不存在，则 OutputMediaAsset 的名称将为 <outputAsset> 元素的任意内部文本值，并以作业名称值或作业 ID 值（在没有定义名称属性的情况下）为后缀。例如，如果将 assetName 的值设置为“Sample”，则会将 OutputMediaAsset 名称属性设置为“Sample”。但是，如果未设置 assetName 的值，但已将作业名称设置为“NewJob”，则 OutputMediaAsset 名称将为“JobOutputAsset(value)\_NewJob”。


##创建包含连锁任务的作业

在许多应用程序方案中，开发人员希望创建一系列处理任务。在媒体服务中，可以创建一系列连锁任务。每个任务执行不同的处理步骤，并且可以使用不同的媒体处理器。连锁任务可以将资产从一个任务转给另一个任务，从而对资产执行线性序列的任务。但是，在作业中执行的任务不需要处于序列中。创建连锁任务时，连锁 **ITask **对象在单个 **IJob** 对象中创建。

>[AZURE.NOTE]每个作业当前有 30 个任务的限制。如果需要链接超过 30 个的任务，请创建多个作业以包含任务。


	POST https://media.chinacloudapi.cn/api/Jobs HTTP/1.1
	Content-Type: application/json;odata=verbose
	Accept: application/json;odata=verbose
	DataServiceVersion: 3.0
	MaxDataServiceVersion: 3.0
	x-ms-version: 2.11
	Authorization: Bearer <token value>
	x-ms-client-request-id: 00000000-0000-0000-0000-000000000000

	{  
	   "Name":"NewTestJob",
	   "InputMediaAssets":[  
	      {  
	         "__metadata":{  
	            "uri":"https://testrest.chinacloudapp.cn/api/Assets('nb%3Acid%3AUUID%3A910ffdc1-2e25-4b17-8a42-61ffd4b8914c')"
	         }
	      }
	   ],
	   "Tasks":[  
	      {  
	         "Configuration":"H264 Adaptive Bitrate MP4 Set 720p",
	         "MediaProcessorId":"nb:mpid:UUID:ff4df607-d419-42f0-bc17-a481b1331e56",
	         "TaskBody":"<?xml version=\"1.0\" encoding=\"utf-8\"?><taskBody><inputAsset>JobInputAsset(0)</inputAsset><outputAsset>JobOutputAsset(0)</outputAsset></taskBody>"
	      },
	      {  
	         "Configuration":"H264 Smooth Streaming 720p",
	         "MediaProcessorId":"nb:mpid:UUID:ff4df607-d419-42f0-bc17-a481b1331e56",
	         "TaskBody":"<?xml version=\"1.0\" encoding=\"utf-16\"?><taskBody><inputAsset>JobOutputAsset(0)</inputAsset><outputAsset>JobOutputAsset(1)</outputAsset></taskBody>"
	      }
	   ]
	}


###注意事项

若要启用任务链，必须满足以下条件：

- 作业必须至少具有两个任务。
- 必须至少有一个任务的输入是作业中另一个任务的输出。

## 使用 OData 批处理 

以下示例演示如何使用 OData 批处理来创建作业和任务。有关批处理的信息，请参阅[开放数据协议 (OData) 批处理](http://www.odata.org/documentation/odata-version-3-0/batch-processing/)。
 
	POST https://media.chinacloudapi.cn/api/$batch HTTP/1.1
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Content-Type: multipart/mixed; boundary=batch_a01a5ec4-ba0f-4536-84b5-66c5a5a6d34e
	Accept: multipart/mixed
	Accept-Charset: UTF-8
	Authorization: Bearer <token>
	x-ms-version: 2.11
	x-ms-client-request-id: 00000000-0000-0000-0000-000000000000
	Host: media.chinacloudapi.cn
	
	
	--batch_a01a5ec4-ba0f-4536-84b5-66c5a5a6d34e
	Content-Type: multipart/mixed; boundary=changeset_122fb0a4-cd80-4958-820f-346309967e4d
	
	--changeset_122fb0a4-cd80-4958-820f-346309967e4d
	Content-Type: application/http
	Content-Transfer-Encoding: binary
	
	POST https://media.chinacloudapi.cn/api/Jobs HTTP/1.1
	Content-ID: 1
	Content-Type: application/json
	Accept: application/json
	DataServiceVersion: 3.0
	MaxDataServiceVersion: 3.0
	Accept-Charset: UTF-8
	Authorization: Bearer <token>
	x-ms-version: 2.11
	x-ms-client-request-id: 00000000-0000-0000-0000-000000000000
	
	{"Name" : "NewTestJob", "InputMediaAssets@odata.bind":["https://media.chinacloudapi.cn/api/Assets('nb%3Acid%3AUUID%3A2a22445d-1500-80c6-4b34-f1e5190d33c6')"]}
	
	--changeset_122fb0a4-cd80-4958-820f-346309967e4d
	Content-Type: application/http
	Content-Transfer-Encoding: binary
	
	POST https://media.chinacloudapi.cn/api/$1/Tasks HTTP/1.1
	Content-ID: 2
	Content-Type: application/json;odata=verbose
	Accept: application/json;odata=verbose
	DataServiceVersion: 3.0
	MaxDataServiceVersion: 3.0
	Accept-Charset: UTF-8
	Authorization: Bearer <token>
	x-ms-version: 2.11
	x-ms-client-request-id: 00000000-0000-0000-0000-000000000000
	
	{  
	   "Configuration":"H264 Adaptive Bitrate MP4 Set 720p",
	   "MediaProcessorId":"nb:mpid:UUID:ff4df607-d419-42f0-bc17-a481b1331e56",
	   "TaskBody":"<?xml version=\"1.0\" encoding=\"utf-8\"?><taskBody><inputAsset>JobInputAsset(0)</inputAsset><outputAsset assetName=\"Custom output name\">JobOutputAsset(0)</outputAsset></taskBody>"
	}

	--changeset_122fb0a4-cd80-4958-820f-346309967e4d--
	--batch_a01a5ec4-ba0f-4536-84b5-66c5a5a6d34e--
 


## 使用 JobTemplate 创建作业


使用一组常用任务处理多个资产时，JobTemplate 可用于指定默认任务预设、任务顺序等。

以下示例演示如何使用以内联方式定义的 TaskTemplate 创建 JobTemplate。TaskTemplate 将媒体编码器标准版用作 MediaProcessor 来编码资产文件；但是，也可使用其他 Mediaprocessor。


	POST https://media.chinacloudapi.cn/API/JobTemplates HTTP/1.1
	Content-Type: application/json;odata=verbose
	Accept: application/json;odata=verbose
	DataServiceVersion: 3.0
	MaxDataServiceVersion: 3.0
	x-ms-version: 2.11
	Authorization: Bearer <token value>
	Host: media.chinacloudapi.cn

	
	{"Name" : "NewJobTemplate25", "JobTemplateBody" : "<?xml version=\"1.0\" encoding=\"utf-8\"?><jobTemplate><taskBody taskTemplateId=\"nb:ttid:UUID:071370A3-E63E-4E81-A099-AD66BCAC3789\"><inputAsset>JobInputAsset(0)</inputAsset><outputAsset>JobOutputAsset(0)</outputAsset></taskBody></jobTemplate>", "TaskTemplates" : [{"Id" : "nb:ttid:UUID:071370A3-E63E-4E81-A099-AD66BCAC3789", "Configuration" : "H264 Smooth Streaming 720p", "MediaProcessorId" : "nb:mpid:UUID:ff4df607-d419-42f0-bc17-a481b1331e56", "Name" : "SampleTaskTemplate2", "NumberofInputAssets" : 1, "NumberofOutputAssets" : 1}] }
 

>[AZURE.NOTE]与其他媒体服务实体不同的是，必须为每个 TaskTemplate 定义一个新的 GUID 标识符并将其放入请求正文中的 taskTemplateId 和 ID 属性中。内容标识方案必须遵循“标识 Azure 媒体服务实体”中所述的方案。此外，不能更新 JobTemplate。而必须创建一个具有已更新的更改的新 JobTemplate。
 

如果成功，将返回以下响应：
	
	HTTP/1.1 201 Created
	
	. . .


以下示例演示如何创建引用 JobTemplate Id 的作业：

	POST https://media.chinacloudapi.cn/API/Jobs HTTP/1.1
	Content-Type: application/json;odata=verbose
	Accept: application/json;odata=verbose
	DataServiceVersion: 3.0
	MaxDataServiceVersion: 3.0
	x-ms-version: 2.11
	Authorization: Bearer <token value>
	Host: media.chinacloudapi.cn

	
	{"Name" : "NewTestJob", "InputMediaAssets" : [{"__metadata" : {"uri" : "https://media.chinacloudapi.cn/api/Assets('nb%3Acid%3AUUID%3A3f1fe4a2-68f5-4190-9557-cd45beccef92')"}}], "TemplateId" : "nb:jtid:UUID:15e6e5e6-ac85-084e-9dc2-db3645fbf0aa"}
	 

如果成功，将返回以下响应：
	
	HTTP/1.1 201 Created
	
	. . . 





##后续步骤
了解如何创建一个对资产进行编码的作业后，请转到[如何使用媒体服务检查作业进度](/documentation/articles/media-services-rest-check-job-progress/)主题。


##另请参阅

[获取媒体处理器](/documentation/articles/media-services-rest-get-media-processor/)
<!---HONumber=Mooncake_0328_2016-->