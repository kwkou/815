<properties linkid="develop-media-services-how-to-guides-deliver-streaming-content" urlDisplayName="Deliver Streaming Content from Media Services" pageTitle="How to Deliver Streaming Content from Media Services a€“ Azure" metaKeywords="" description="Learn how to deliver streaming content from Media Services using a direct URL. Code samples are written in C# and use the Media Services SDK for .NET." metaCanonical="" disqusComments="1" umbracoNaviHide="0" title="How to: Deliver streaming content" authors="" />

如何：交付流内容
================

本文是介绍 Azure Media Services 编程的系列主题中的一篇。前一个主题是[如何：通过下载交付资产](http://go.microsoft.com/fwlink/?LinkID=301734&clcid=0x409)。

除了可从 Media Services 下载媒体内容外，你还可以使用自适应位速率流式处理来交付内容。例如，你可以创建一个定向 URL（称为定位器），以流式传输 Media Services 来源服务器上的内容。Microsoft Silverlight 等客户端应用程序可以直接从定位器播放流内容。

以下示例演示如何为作业生成的输出资产创建一个来源定位器。该示例假设你已获取对包含平滑流式处理文件的资产的引用；代码中引用的是名为 **assetToStream** 的变量。

若要创建指向流内容的来源定位器，请执行以下操作：

1.  获取对资产中流式处理清单文件 (.ism) 的引用
2.  定义访问策略
3.  通过调用 CreateLocator 方法创建来源定位器
4.  生成清单文件的 URL

以下代码演示如何执行这些步骤：

``` {}
private static ILocator GetStreamingOriginLocator( string targetAssetID)
{
// Get a reference to the asset you want to stream.
IAsset assetToStream = GetAsset(targetAssetID);

// Get a reference to the streaming manifest file from the  
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
policy = _context.AccessPolicies.Create("Streaming policy",
TimeSpan.FromDays(30),
AccessPermissions.Read);

// Create an OnDemandOrigin locator to the asset. 
originLocator = _context.Locators.CreateLocator(LocatorType.OnDemandOrigin, assetToStream,
policy,
DateTime.UtcNow.AddMinutes(-5));
            
// Display some useful values based on the locator.
Console.WriteLine("Streaming asset base path on origin: ");
Console.WriteLine(originLocator.Path);
Console.WriteLine();
    
// Create a full URL to the manifest file.Use this for playback
// in streaming media clients. 
string urlForClientStreaming = originLocator.Path + manifestFile.Name + "/manifest";
Console.WriteLine("URL to manifest for client streaming: ");
Console.WriteLine(urlForClientStreaming);
Console.WriteLine();
    
// Display the ID of the origin locator, the access policy, and the asset.
Console.WriteLine("Origin locator Id:" + originLocator.Id);
Console.WriteLine("Access policy Id:" + policy.Id);
Console.WriteLine("Streaming asset Id:" + assetToStream.Id);

// Return the locator. 
return originLocator;
}
```

有关交付资产的详细信息，请参阅：

-   [使用 Media Services for .NET 交付资产](http://msdn.microsoft.com/zh-cn/library/jj129575.aspx)
-   [使用 Media Services REST API 交付资产](http://msdn.microsoft.com/zh-cn/library/jj129578.aspx)

后续步骤
--------

到目前为止，我们已介绍如何通过从 Azure 存储空间下载内容以及使用平滑流式处理来交付媒体。下一主题[如何交付 HLS 内容](http://go.microsoft.com/fwlink/?LinkId=301817)将介绍如何使用 Apple HTTP 实时流 (HLS) 交付流内容。

