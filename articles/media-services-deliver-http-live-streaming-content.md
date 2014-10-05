<properties linkid="develop-media-services-how-to-guides-deliver-apple-live-streaming" urlDisplayName="Deliver Apple HTTP Live Streaming (HLS)" pageTitle="How to Deliver Apple HTTP Live Streaming (HLS) - Azure" metaKeywords="" description="Learn how to create a locator to Apple HTTP Live Stream (HLS) content on Media Services origin server. Code samples are written in C# and use the Media Services SDK for .NET." metaCanonical="" services="media-services" documentationCenter="" title="How to: Deliver Apple HLS streaming content" authors="migree" solutions="" manager="" editor="" />

如何：交付 Apple HLS 流内容
===========================

本文是介绍 Azure Media Services 编程的系列主题中的一篇。前一个主题是[如何：交付流内容](http://go.microsoft.com/fwlink/?LinkID=301942&clcid=0x409)。

本主题说明如何在 Media Services 来源服务器上创建一个定位器，以指向 Apple HTTP 实时流 (HLS) 内容。使用此方法可以生成 Apple HLS 内容的 URL，并将该 URL 提供给 Apple iOS 设备进行播放。生成定位器 URL 的基本思路与此相同。在来源服务器上生成指向 Apple HLS 流资产路径的定位器，然后生成链接到流内容清单的完整 URL。

以下代码示例假设你已获取对 HLS 流资产的引用；代码中引用的是名为 **assetToStream** 的变量。运行此代码以在资产上生成来源定位器后，你可以使用生成的 URL 在 iPad 或 iPhone 等 iOS 设备中播放流内容。

若要生成指向 Apple HLS 流内容的定位器，请执行以下操作：

1.  获取对资产中清单文件的引用
2.  定义访问策略
3.  通过调用 CreateLocator 创建来源定位器
4.  生成清单文件的 URL

以下代码演示如何执行这些步骤：

``` {}
static ILocator GetStreamingHLSOriginLocator( string targetAssetID)
{
// Get a reference to the asset you want to stream.
IAsset assetToStream = GetAsset(targetAssetID);

// Get a reference to the HLS streaming manifest file from the  
// collection of files in the asset. 
var theManifest =
from f in assetToStream.AssetFiles
where f.Name.EndsWith(".ism")
select f;

// Cast the reference to a true IAssetFile type. 
IAssetFile manifestFile = theManifest.First();
IAccessPolicy policy = null;
ILocator originLocator = null;

// Create a 30-day readonly access policy. 
policy = _context.AccessPolicies.Create("Streaming HLS Policy",
TimeSpan.FromDays(30),
AccessPermissions.Read);

originLocator = _context.Locators.CreateLocator(LocatorType.OnDemandOrigin, assetToStream,
policy,
DateTime.UtcNow.AddMinutes(-5));

// Create a URL to the HLS streaming manifest file.Use this for playback
// in Apple iOS streaming clients.
string urlForClientStreaming = originLocator.Path
+ manifestFile.Name + "/manifest(format=m3u8-aapl)";
Console.WriteLine("URL to manifest for client streaming: ");
Console.WriteLine(urlForClientStreaming);
Console.WriteLine();

// Return the locator. 
return originLocator;
}
```

有关交付资产的详细信息，请参阅：

-   [使用 Media Services for .NET 交付资产](http://msdn.microsoft.com/zh-cn/library/jj129575.aspx)
-   [使用 Media Services REST API 交付资产](http://msdn.microsoft.com/zh-cn/library/jj129578.aspx)

后续步骤
--------

本主题是“使用 Azure Media Services”系列主题中的最后一篇。我们介绍了如何设置计算机以进行 Media Services 开发，以及如何执行典型的编程任务。有关 Media Services 编程的详细信息，请参阅以下资源：

-   [Azure Media Services 文档](http://go.microsoft.com/fwlink/?linkid=245437)
-   [Media Services SDK for .NET 入门](http://go.microsoft.com/fwlink/?linkid=252966)
-   [使用 Media Services SDK for .NET 生成应用程序](http://go.microsoft.com/fwlink/?linkid=247821)
-   [使用 Azure Media Services REST API 生成应用程序](http://go.microsoft.com/fwlink/?linkid=252967)
-   [Media Services 论坛](http://social.msdn.microsoft.com/Forums/zh-cn/MediaServices/threads)
-   [如何监视 Media Services 帐户](http://www.windowsazure.cn/zh-cn/manage/services/media-services/how-to-monitor-a-media-services-account/)
-   [如何管理 Media Services 中的内容](http://www.windowsazure.cn/zh-cn/manage/services/media-services/how-to-manage-content-in-media-services/)

