<properties linkid="develop-media-services-how-to-guides-deliver-media-assets" urlDisplayName="Delivering Media Assets" pageTitle="How to Deliver Media Assets - Azure" metaKeywords="" description="Learn about options for delivering media assets that have been uploaded to Media Services in Azure. Code samples are written in C# and use the Media Services SDK for .NET." metaCanonical="" services="media-services" documentationCenter="" title="How to: Deliver an Asset by Download" authors="migree" solutions="" manager="" editor="" />

如何：通过下载交付资产
======================

本文是介绍 Azure Media Services 编程的系列主题中的一篇。前一个主题是[如何：管理资产](http://go.microsoft.com/fwlink/?LinkID=301815&clcid=0x409)。

本主题介绍已上载到 Media Services 的媒体资产的交付选项。你可以采用众多的应用程序方案来交付 Media Services 内容。你可以下载媒体资产，或使用定位器访问媒体资产。你可以将媒体内容发送到其他应用程序或其他内容提供商。为了提高性能和可伸缩性，你还可以使用 Azure CDN 等内容交付网络 (CDN) 来交付内容。

此示例演示如何从 Media Services 下载媒体资产。该代码将按作业 ID 查询与 Media Services 帐户关联的作业，并访问其 **OutputMediaAssets** 集合（运行作业后生成的、包含一个或多个输出媒体资产的集）。此示例演示如何通过作业下载输出媒体资产，但你可以运用相同的方法来下载其他资产。

``` {}
 
// Download the output asset of the specified job to a local folder.
static IAsset DownloadAssetToLocal(string jobId, string outputFolder)
{
// This method illustrates how to download a single asset. 
// However, you can iterate through the OutputAssets
// collection, and download all assets if there are many. 

// Get a reference to the job. 
IJob job = GetJob(jobId);

// Get a reference to the first output asset.If there were multiple 
// output media assets you could iterate and handle each one.
IAsset outputAsset = job.OutputMediaAssets[0];

    // Create a SAS locator to download the asset
IAccessPolicy accessPolicy = _context.AccessPolicies.Create("File Download Policy", TimeSpan.FromDays(30), AccessPermissions.Read);
ILocator locator = _context.Locators.CreateSasLocator(outputAsset, accessPolicy);

BlobTransferClient blobTransfer = new BlobTransferClient
    {
NumberOfConcurrentTransfers = 20,
ParallelTransferThreadCount = 20
    };

var downloadTasks = new List<Task>();
foreach (IAssetFile outputFile in outputAsset.AssetFiles)
    {
// Use the following event handler to check download progress.
outputFile.DownloadProgressChanged += DownloadProgress;

string localDownloadPath = Path.Combine(outputFolder, outputFile.Name);

Console.WriteLine("File download path:" + localDownloadPath);

downloadTasks.Add(outputFile.DownloadAsync(Path.GetFullPath(localDownloadPath), blobTransfer, locator, CancellationToken.None));

outputFile.DownloadProgressChanged -= DownloadProgress;
    }

Task.WaitAll(downloadTasks.ToArray());

return outputAsset;
}

static void DownloadProgress(object sender, DownloadProgressChangedEventArgs e)
{
Console.WriteLine(string.Format("{0} % download progress.", e.Progress));
}
```

有关交付资产的详细信息，请参阅：

-   [使用 Media Services for .NET 交付资产](http://msdn.microsoft.com/zh-cn/library/jj129575.aspx)
-   [使用 Media Services REST API 交付资产](http://msdn.microsoft.com/zh-cn/library/jj129578.aspx)

后续步骤
--------

本主题介绍了如何从 Azure 存储空间下载资产。有关其他资产交付方式的信息，请转到[如何交付流内容](http://go.microsoft.com/fwlink/?LinkID=301942)主题。

