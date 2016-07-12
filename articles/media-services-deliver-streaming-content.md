<properties 
	pageTitle="如何从媒体服务传送流内容" 
	description="了解如何创建用于生成流 URL 的定位符。代码示例用 C# 编写且使用适用于 .NET 的媒体服务 SDK。" 
	authors="juliako" 
	manager="dwrede" 
	editor="" 
	services="media-services" 
	documentationCenter=""/>

<tags
	ms.service="media-services"
	ms.date="04/18/2016"
	wacn.date="06/20/2016"/>


#如何：传送流内容
 
> [AZURE.SELECTOR]
- [REST](/documentation/articles/media-services-rest-deliver-streaming-content/)
- [.NET](/documentation/articles/media-services-deliver-streaming-content/)
- [管理门户](/documentation/articles/media-services-manage-content/#publish)

##概述

你可以通过创建 OnDemand 流式处理定位符并生成流 URL 来流式传输自适应比特率 MP4 集。[对资产进行编码](/documentation/articles/media-services-encode-asset/)主题说明了如何编码成自适应比特率 MP4 集。

>[AZURE.NOTE]如果内容已加密，则在创建定位符之前配置资产传送策略（如[本](/documentation/articles/media-services-dotnet-configure-asset-delivery-policy/)主题中所述）。

你也可以使用 OnDemand 流式处理定位符生成指向可渐进式下载的 MP4 文件的 URL。

本主题说明如何创建 OnDemand 流式处理定位符，以发布资产及生成平滑流、MPEG DASH 和 HLS 流 URL。此外，还将示范如何生成渐进式下载 URL。
  	 
##创建 OnDemand 流式处理定位符

若要创建 OnDemand 流式处理定位符并获取 URL，你需要执行以下操作：

   1. 如果内容已加密，则定义访问策略。
   2. 创建 OnDemand 流式处理定位符。
   3. 如果你想要流式处理，请获取资产中的流式处理清单文件 (.ism)。 
   		
	如果你想要渐进式下载，请获取资产中的 MP4 文件名。  
   4. 生成清单文件或 MP4 文件的 URL。 
   

###使用媒体服务 .NET SDK 

生成流 URL

	private static void BuildStreamingURLs(IAsset asset)
	{
	
	    // Create a 30-day readonly access policy. 
      	// You cannot create a streaming locator using an AccessPolicy that includes write or delete permissions.
	    IAccessPolicy policy = _context.AccessPolicies.Create("Streaming policy",
	        TimeSpan.FromDays(30),
	        AccessPermissions.Read);
	
	    // Create a locator to the streaming content on an origin. 
	    ILocator originLocator = _context.Locators.CreateLocator(LocatorType.OnDemandOrigin, asset,
	        policy,
	        DateTime.UtcNow.AddMinutes(-5));
	
	    // Display some useful values based on the locator.
	    Console.WriteLine("Streaming asset base path on origin: ");
	    Console.WriteLine(originLocator.Path);
	    Console.WriteLine();
	
	    // Get a reference to the streaming manifest file from the  
	    // collection of files in the asset. 
	    var manifestFile = asset.AssetFiles.Where(f => f.Name.ToLower().
	                                EndsWith(".ism")).
	                                FirstOrDefault();
	    
	    // Create a full URL to the manifest file. Use this for playback
	    // in streaming media clients. 
	    string urlForClientStreaming = originLocator.Path + manifestFile.Name + "/manifest";
	    Console.WriteLine("URL to manifest for client streaming using Smooth Streaming protocol: ");
	    Console.WriteLine(urlForClientStreaming);
	    Console.WriteLine("URL to manifest for client streaming using HLS protocol: ");
	    Console.WriteLine(urlForClientStreaming + "(format=m3u8-aapl)");
	    Console.WriteLine("URL to manifest for client streaming using MPEG DASH protocol: ");
	    Console.WriteLine(urlForClientStreaming + "(format=mpd-time-csf)"); 
	    Console.WriteLine();
	}

代码将输出：
	
	URL to manifest for client streaming using Smooth Streaming protocol:
	http://amstest1.streaming.mediaservices.chinacloudapi.cn/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny.ism/manifest
	URL to manifest for client streaming using HLS protocol:
	http://amstest1.streaming.mediaservices.chinacloudapi.cn/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny.ism/manifest(format=m3u8-aapl)
	URL to manifest for client streaming using MPEG DASH protocol:
	http://amstest1.streaming.mediaservices.chinacloudapi.cn/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny.ism/manifest(format=mpd-time-csf)
	

>[AZURE.NOTE]你也可以通过 SSL 连接流式传输内容。为此，请确保流 URL 以 HTTPS 开头。

生成渐进式下载 URL

	private static void BuildProgressiveDownloadURLs(IAsset asset)
	{
	    // Create a 30-day readonly access policy. 
	    IAccessPolicy policy = _context.AccessPolicies.Create("Streaming policy",
	        TimeSpan.FromDays(30),
	        AccessPermissions.Read);
	
	    // Create an OnDemandOrigin locator to the asset. 
	    ILocator originLocator = _context.Locators.CreateLocator(LocatorType.OnDemandOrigin, asset,
	        policy,
	        DateTime.UtcNow.AddMinutes(-5));
	
	    // Display some useful values based on the locator.
	    Console.WriteLine("Streaming asset base path on origin: ");
	    Console.WriteLine(originLocator.Path);
	    Console.WriteLine();
	
	    // Get MP4 files.
	    IEnumerable<IAssetFile> mp4AssetFiles = asset
	        .AssetFiles
	        .ToList()
	        .Where(af => af.Name.EndsWith(".mp4", StringComparison.OrdinalIgnoreCase));
	            
	    // Create a full URL to the MP4 files. Use this to progressively download files.
	    foreach (var pd in mp4AssetFiles)
	        Console.WriteLine(originLocator.Path + pd.Name);
	}

代码将输出：
	
	http://amstest1.streaming.mediaservices.chinacloudapi.cn/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny_H264_650kbps_AAC_und_ch2_96kbps.mp4
	http://amstest1.streaming.mediaservices.chinacloudapi.cn/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny_H264_400kbps_AAC_und_ch2_96kbps.mp4
	http://amstest1.streaming.mediaservices.chinacloudapi.cn/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny_H264_3400kbps_AAC_und_ch2_96kbps.mp4
	http://amstest1.streaming.mediaservices.chinacloudapi.cn/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny_H264_2250kbps_AAC_und_ch2_96kbps.mp4
	
	. . . 

###使用 Azure 媒体服务 .NET SDK 扩展

以下代码将调用 .NET SDK 扩展方法，用于创建定位符，并为自适应流生成平滑流、HLS 和 MPEG-DASH URL。

	// Create a loctor.
	_context.Locators.Create(
	    LocatorType.OnDemandOrigin,
	    inputAsset,
	    AccessPermissions.Read,
	    TimeSpan.FromDays(30));
	
	// Get the streaming URLs.
	Uri smoothStreamingUri = inputAsset.GetSmoothStreamingUri();
	Uri hlsUri = inputAsset.GetHlsUri();
	Uri mpegDashUri = inputAsset.GetMpegDashUri();
	
	Console.WriteLine(smoothStreamingUri);
	Console.WriteLine(hlsUri);
	Console.WriteLine(mpegDashUri);



##另请参阅

[下载资产](/documentation/articles/media-services-deliver-asset-download/) 
[配置资产传送策略](/documentation/articles/media-services-dotnet-configure-asset-delivery-policy/)
<!---HONumber=Mooncake_0613_2016-->