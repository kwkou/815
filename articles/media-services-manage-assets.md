<properties linkid="develop-media-services-how-to-guides-manage-assets" urlDisplayName="Manage Assets in Media Services" pageTitle="How to Manage Assets in Media Services - Azure" metaKeywords="" description="Learn how to manage assets on Media Services. You can also manage jobs, tasks, access policies, locators, and more. Code samples are written in C# and use the Media Services SDK for .NET." metaCanonical="" services="media-services" documentationCenter="" title="How to: Manage Assets in storage" authors="migree" solutions="" manager="" editor="" />

如何：管理存储中的资产
======================

本文是介绍 Azure Media Services 编程的系列主题中的一篇。前一个主题是[如何：保护资产](http://go.microsoft.com/fwlink/?LinkID=301813&clcid=0x409)。

在创建媒体资产并将其上载到 Media Services 后，你可以在服务器上访问和管理这些资产。还可以在服务器上管理属于 Media Services 的其他对象，包括作业、任务、访问策略、定位器，等等。

以下示例演示如何按 assetId 查询资产。

``` {}
static IAsset GetAsset(string assetId)
{
// Use a LINQ Select query to get an asset.
var assetInstance =
from a in _context.Assets
where a.Id == assetId
select a;
// Reference the asset as an IAsset.
IAsset asset = assetInstance.FirstOrDefault();

return asset;
}
```

若要列出服务器上的所有可用资产，你可以使用以下方法，该方法将循环访问资产集合并显示有关每个资产的详细信息。

``` {}
 
static void ListAssets()
{
string waitMessage = "Building the list.This may take a few "
+ "seconds to a few minutes depending on how many assets "
+ "you have."
+ Environment.NewLine + Environment.NewLine
+ "Please wait..."
+ Environment.NewLine;
Console.Write(waitMessage);

// Create a Stringbuilder to store the list that we build. 
StringBuilder builder = new StringBuilder();

foreach (IAsset asset in _context.Assets)
    {
// Display the collection of assets.
builder.AppendLine("");
builder.AppendLine("******ASSET******");
builder.AppendLine("Asset ID:" + asset.Id);
builder.AppendLine("Name:" + asset.Name);
builder.AppendLine("==============");
builder.AppendLine("******ASSET FILES******");

// Display the files associated with each asset. 
foreach (IAssetFile fileItem in asset.AssetFiles)
        {
builder.AppendLine("Name:" + fileItem.Name);
builder.AppendLine("Size:" + fileItem.ContentFileSize);
builder.AppendLine("==============");
        }
    }

// Display output in console.
Console.Write(builder.ToString());
}
```

以下代码段将从 Media Services 帐户中删除所有资产。

``` {}
foreach (IAsset asset in _context.Assets)
{
asset.Delete();
}
```

有关管理资产的详细信息，请参阅：

-   [使用 Media Services SDK for .NET 管理资产](http://msdn.microsoft.com/zh-cn/library/jj129589.aspx)
-   [使用 Media Services REST API 管理资产](http://msdn.microsoft.com/zh-cn/library/jj129583.aspx)

后续步骤
--------

了解如何管理资产后，请转到[如何通过下载交付资产](http://go.microsoft.com/fwlink/?LinkID=301734&clcid=0x409)主题。

