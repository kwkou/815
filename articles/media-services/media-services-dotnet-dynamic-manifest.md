<properties 
	pageTitle="使用 Azure 媒体服务 .NET SDK 创建筛选器" 
	description="本主题介绍如何创建筛选器，以便客户端能够使用它们来流式传输流的特定部分。媒体服务将创建动态清单来存档此选择性流。" 
	services="media-services" 
	documentationCenter="" 
	authors="Juliako" 
	manager="dwrede,cenkdin" 
	editor=""/>

<tags
	ms.service="media-services"
	ms.date="04/18/2016"
	wacn.date="06/20/2016"/>


#使用 Azure 媒体服务 .NET SDK 创建筛选器

> [AZURE.SELECTOR]
- [.NET](/documentation/articles/media-services-dotnet-dynamic-manifest/)
- [REST](/documentation/articles/media-services-rest-dynamic-manifest/)

从 2.11 版开始，媒体服务可让你为资产定义筛选器。这些筛选器是服务器端规则，可让你的客户选择运行如下操作：只播放一段视频（而非播放完整视频），或只指定客户设备可以处理的一部分音频和视频再现内容（而非与该资产相关的所有再现内容）。通过按客户请求创建的**动态清单**可以实现对资产进行这种筛选，并基于指定的筛选器流式传输视频。

有关与筛选器和动态清单相关的更多详细信息，请参阅[动态清单概述](/documentation/articles/media-services-dynamic-manifest-overview/)。

本主题说明如何使用媒体服务 .NET SDK 创建、更新和删除筛选器。


请注意，如果你更新筛选器，则流式处理终结点需要 2 分钟的时间来刷新规则。如果内容是通过使用此筛选器提供的（并在代理和 CDN 缓存中缓存），则更新此筛选器会导致播放器失败。建议在更新筛选器之后清除缓存。如果此选项不可用，请考虑使用其他筛选器。

##用于创建筛选器的类型

创建筛选器时将使用以下类型：

- **IStreamingFilter**。此类型基于以下 REST API [Filter](http://msdn.microsoft.com/zh-cn/library/azure/mt149056.aspx)
- **IStreamingAssetFilter**。此类型基于以下 REST API [AssetFilter](http://msdn.microsoft.com/zh-cn/library/azure/mt149053.aspx)
- **PresentationTimeRange**此类型基于以下 REST API [PresentationTimeRange](http://msdn.microsoft.com/zh-cn/library/azure/mt149052.aspx)
- **FilterTrackSelectStatement** 和 **IFilterTrackPropertyCondition**。这些类型基于以下 REST API [FilterTrackSelect 和 FilterTrackPropertyCondition](http://msdn.microsoft.com/zh-cn/library/azure/mt149055.aspx)


##创建/更新/读取/删除全局筛选器

下面的代码演示如何使用 .NET 创建、更新、读取和删除资产筛选器。
	
	string filterName = "GlobalFilter_" + Guid.NewGuid().ToString();
	            
	List<FilterTrackSelectStatement> filterTrackSelectStatements = new List<FilterTrackSelectStatement>();
	
	FilterTrackSelectStatement filterTrackSelectStatement = new FilterTrackSelectStatement();
	filterTrackSelectStatement.PropertyConditions = new List<IFilterTrackPropertyCondition>();
	filterTrackSelectStatement.PropertyConditions.Add(new FilterTrackNameCondition("Track Name", FilterTrackCompareOperator.NotEqual));
	filterTrackSelectStatement.PropertyConditions.Add(new FilterTrackBitrateRangeCondition(new FilterTrackBitrateRange(0, 1), FilterTrackCompareOperator.NotEqual));
	filterTrackSelectStatement.PropertyConditions.Add(new FilterTrackTypeCondition(FilterTrackType.Audio, FilterTrackCompareOperator.NotEqual));
	filterTrackSelectStatements.Add(filterTrackSelectStatement);
	
	// Create
	IStreamingFilter filter = _context.Filters.Create(filterName, new PresentationTimeRange(), filterTrackSelectStatements);
	
	// Update
	filter.PresentationTimeRange = new PresentationTimeRange(timescale: 500);
	filter.Update();
	
	// Read
	var filterUpdated = _context.Filters.FirstOrDefault();
	Console.WriteLine(filterUpdated.Name);

	// Delete
	filter.Delete();


##创建/更新/读取/删除资产筛选器

下面的代码演示如何使用 .NET 创建、更新、读取和删除资产筛选器。

	
	string assetName = "AssetFilter_" + Guid.NewGuid().ToString();
	var asset = _context.Assets.Create(assetName, AssetCreationOptions.None);
	
	string filterName = "AssetFilter_" + Guid.NewGuid().ToString();
	
	    
	// Create
	IStreamingAssetFilter filter = asset.AssetFilters.Create(filterName,
	                                    new PresentationTimeRange(), 
	                                    new List<FilterTrackSelectStatement>());
	
	// Update
	filter.PresentationTimeRange = 
	        new PresentationTimeRange(start: 6000000000, end: 72000000000);
	
	filter.Update();
	
	// Read
	asset = _context.Assets.Where(c => c.Id == asset.Id).FirstOrDefault();
	var filterUpdated = asset.AssetFilters.FirstOrDefault();
	Console.WriteLine(filterUpdated.Name);
	
	// Delete
	filterUpdated.Delete();
	



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
 


<!---HONumber=Mooncake_0613_2016-->