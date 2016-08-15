<properties 
	pageTitle="下载媒体资产" 
	description="了解如何将资产下载到计算机。代码示例用 C# 编写且使用 适用于 .NET 的媒体服务 SDK。" 
	services="media-services" 
	documentationCenter="" 
	authors="juliako" 
	manager="dwrede" 
	editor=""/>

<tags
	ms.service="media-services"
	ms.date="04/18/2016"
	wacn.date="06/20/2016"/>

#如何：通过下载交付资产

本主题介绍已上载到媒体服务的媒体资产的交付选项。你可以采用众多的应用程序方案来交付媒体服务内容。你可以下载媒体资产，或使用定位器访问媒体资产。你可以将媒体内容发送到其他应用程序或其他内容提供商。为了提高性能和可缩放性，你还可以使用内容传送网络 (CDN) 来传送内容。

此示例演示如何将媒体资产从媒体服务下载到本地计算机。该代码将按作业 ID 查询与媒体服务帐户关联的作业，并访问其 **OutputMediaAssets** 集合（运行作业后生成的、包含一个或多个输出媒体资产的集）。此示例演示如何通过作业下载输出媒体资产，但你可以运用相同的方法来下载其他资产。

	
	// Download the output asset of the specified job to a local folder.
	static IAsset DownloadAssetToLocal( string jobId, string outputFolder)
	{
	    // This method illustrates how to download a single asset. 
	    // However, you can iterate through the OutputAssets
	    // collection, and download all assets if there are many. 
	
	    // Get a reference to the job. 
	    IJob job = GetJob(jobId);
	
	    // Get a reference to the first output asset. If there were multiple 
	    // output media assets you could iterate and handle each one.
	    IAsset outputAsset = job.OutputMediaAssets[0];
	
		// Create a SAS locator to download the asset
	    IAccessPolicy accessPolicy = _context.AccessPolicies.Create("File Download Policy", TimeSpan.FromDays(30), AccessPermissions.Read);
	    ILocator locator = _context.Locators.CreateLocator(LocatorType.Sas, outputAsset, accessPolicy);
	
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
	
	        Console.WriteLine("File download path:  " + localDownloadPath);
	
	        downloadTasks.Add(outputFile.DownloadAsync(Path.GetFullPath(localDownloadPath), blobTransfer, locator, CancellationToken.None));
	
	        outputFile.DownloadProgressChanged -= DownloadProgress;
	    }
	
	    Task.WaitAll(downloadTasks.ToArray());
	
	    return outputAsset;
	}
	
	static void DownloadProgress(object sender, DownloadProgressChangedEventArgs e)
	{
	    Console.WriteLine(string.Format("{0} % download progress. ", e.Progress));
	}


   
##另请参阅 

[交付流内容](/documentation/articles/media-services-deliver-streaming-content/)


<!---HONumber=Mooncake_0613_2016-->