<properties 
	pageTitle="常见问题" 
	description="常见问题 (FAQ)" 
	services="media-services" 
	documentationCenter="" 
	authors="Juliako" 
	manager="dwrede" 
	editor=""/>

<tags
	ms.service="media-services"
 	ms.date="03/02/2016"  
	wacn.date="04/05/2016"/>


#常见问题  

##一般性的 AMS 常见问题 

问：如何缩放索引？

答：编码任务和索引任务的保留单位相同。请遵循[如何缩放编码保留单位](/documentation/articles/media-services-how-to-scale/)中的说明。**请注意**，保留单位类型不影响索引器性能。

问：我已经上载、编码并发布了视频。为什么在我尝试对视频进行流式处理时，它不播放？

答：最常见的一种原因是，你没有在尝试播放的流式处理终结点上分配至少一个保留流式处理单位。请遵循[如何缩放流式处理保留单位](/documentation/articles/media-services-how-to-scale/)中的说明。

问：我是否可以在实时流上进行合成操作？

答：Azure 媒体服务当前不提供实时流上的合成操作，因此你需要在计算机上进行预合成。

问：我是否可以将 Azure CDN 与实时流式处理一起使用？

答：媒体服务支持与 Azure CDN 集成（有关详细信息，请参阅[如何在媒体服务帐户中管理流式处理终结点](/documentation/articles/media-services-manage-origins/#enable_cdn)）。可以将实时流式处理与 CDN 结合使用。Azure 媒体服务提供平滑流式处理、HLS 和 MPEG-DASH 输出。所有这些格式均使用 HTTP 来传输数据并从 HTTP 缓存中获益。在实时流式处理中，实际视频/音频数据被分为数个片段，并且单个片段缓存于 CDN 中。仅需要刷新的数据是清单数据。CDN 定期刷新清单数据。

问：Azure 媒体服务是否支持存储图像？

答：如果需要存储 JPEG 或 PNG 图像，应将其存储在 Azure Blob 存储中。除非你想要将图像与你的视频或音频资产相关联，否则将图像放入媒体服务帐户毫无益处。或者，你可能需要在视频编码器中将图像用作叠加。媒体编码器标准版支持在视频上叠加图像，且它将 JPEG 和 PNG 列为支持的输入格式。有关详细信息，请参阅[创建覆盖](/documentation/articles/media-services-custom-mes-presets-with-dotnet/#overlay)。

问：如何将资产从一个媒体服务帐户复制到另一个媒体服务帐户？

答：要使用 .NET 将资产从一个媒体服务帐户复制到另一个帐户，可以使用 [Azure 媒体服务 .NET SDK 扩展](https://github.com/Azure/azure-sdk-for-media-services-extensions/)存储库中提供的 [IAsset.Copy](https://github.com/Azure/azure-sdk-for-media-services-extensions/blob/dev/MediaServices.Client.Extensions/IAssetExtensions.cs#L354) 扩展方法。有关详细信息，请参阅[此](https://social.msdn.microsoft.com/Forums/zh-cn/28912d5d-6733-41c1-b27d-5d5dff2695ca/migrate-media-services-across-subscription?forum=MediaServices)论坛线程。

问：AMS 支持使用哪些字符来为文件命名？

答：生成流式处理内容的 URL（例如 http://{AMSAccount}.origin.mediaservices.chinacloudapi.cn/{GUID}/{IAssetFile.Name}/streamingParameters.）时，媒体服务会使用 IAssetFile.Name 属性的值。出于这个原因，不允许使用百分号编码。**Name** 属性的值不能含有任何以下保留的[百分号编码字符](http://en.wikipedia.org/wiki/Percent-encoding#Percent-encoding_reserved_characters)：!*'();:@&=+$,/?%#"。此外，文件扩展名中只能含有一个“¡®.¡¯”。


问：如何使用 REST 进行连接？

答：在成功连接到 https://media.chinacloudapi.cn 之后，你将接收到指定另一个媒体服务 URI 的 301 重定向。必须按[使用 REST API 连接到媒体服务](/documentation/articles/media-services-rest-connect_programmatically/)中所述对新的 URI 执行后续调用。


问：如何在编码过程中旋转视频。

答：[媒体编码器标准](/documentation/articles/media-services-dotnet-encode-with-media-encoder-standard/)支持旋转 90/180/270 度。默认行为是“自动”，即尝试在传入的 MP4/MOV 文件中检测旋转元数据并对其进行补偿。包含[此处](http://msdn.microsoft.com/zh-cn/library/azure/mt269960.aspx)定义的 json 预设之一的以下 **Sources** 元素：
	
	"Version": 1.0,
	"Sources": [
	{
	  "Streams": [],
	  "Filters": {
	    "Rotation": "90"
	  }
	}
	],
	"Codecs": [
	
	...

<!---HONumber=Mooncake_0328_2016-->