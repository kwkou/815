<properties 
	pageTitle="如何使用媒体编码器标准版对资产进行编码" 
	description="本主题介绍如何使用 .NET 通过媒体编码器标准版对资产进行编码。" 
	services="media-services" 
	documentationCenter="" 
	authors="juliako,anilmur" 
	manager="dwrede" 
	editor=""/>

<tags
	ms.service="media-services"
	ms.date="12/02/2015"
	wacn.date=""/>


#如何使用媒体编码器标准版对资产进行编码

编码作业是媒体服务中最常见的处理操作之一。可通过创建编码作业将媒体文件从一种编码转换为另一种编码。进行编码时，可以使用媒体服务内置的 Media Encoder。你也可以使用媒体服务合作伙伴提供的编码器；可通过 Azure 应用商店获取第三方编码器。

本主题介绍如何使用 .NET 通过媒体编码器标准 (MES) 对资产进行编码。媒体编码器标准使用[此处](https://msdn.microsoft.com/zh-cn/library/azure/mt269960.aspx)所述的编码器预设之一进行配置。

建议始终将夹层文件编码为自适应比特率 MP4 集，然后使用[动态打包](/documentation/articles/media-services-dynamic-packaging-overview)将该集转换为所需的格式。若要利用动态打包，首先必须获取你计划从中传送内容的流式处理终结点的至少一个点播流单元。有关详细信息，请参阅[如何缩放媒体服务](/documentation/articles/media-services-manage-origins#scale_streaming_endpoints)。

如果你的输出资产已经过存储加密，则必须配置资产传送策略。有关详细信息，请参阅[配置资产传送策略](/documentation/articles/media-services-dotnet-configure-asset-delivery-policy)。

###MES 格式

[格式和编解码器](/documentation/articles/media-services-media-encoder-standard-formats)

###MES 预设

媒体编码器标准使用[此处](https://msdn.microsoft.com/zh-cn/library/azure/mt269960.aspx)所述的编码器预设之一进行配置。

###MES 输入和输出元数据

[此处](http://msdn.microsoft.com/zh-cn/library/azure/dn783120.aspx)说明了编码器输入元数据。

[此处](http://msdn.microsoft.com/zh-cn/library/azure/dn783217.aspx)说明了编码器输出元数据。


##示例

以下代码示例使用媒体服务 .NET SDK 执行下列任务：

- 创建编码作业。
- 获取对媒体编码器标准版编码器的引用。
- 指定使用“H264 多比特率 720p”预设。你可以在[此处](https://msdn.microsoft.com/zh-cn/library/azure/mt269960.aspx)看到所有预设。你还可以在[此处](https://msdn.microsoft.com/zh-cn/library/mt269962.aspx)查看相关主题，了解这些预设必须遵循的架构。
- 将一个编码任务添加到该作业。 
- 指定要编码的输入资产。
- 创建将包含所编码资产的输出资产。
- 添加事件处理程序以检查作业进度。
- 提交作业。
		
		static public IAsset EncodeToAdaptiveBitrateMP4Set(IAsset asset)
		{
		    // Declare a new job.
		    IJob job = _context.Jobs.Create("Media Encoder Standard Job");
		    // Get a media processor reference, and pass to it the name of the 
		    // processor to use for the specific task.
		    IMediaProcessor processor = GetLatestMediaProcessorByName("Media Encoder Standard");
		

		    // Create a task with the encoding details, using a string preset.
		    // In this case "H264 Multiple Bitrate 720p" preset is used.
		    ITask task = job.Tasks.AddNew("My encoding task",
		        processor,
		        "H264 Multiple Bitrate 720p",
		        TaskOptions.None);
		
		    // Specify the input asset to be encoded.
		    task.InputAssets.Add(asset);
		    // Add an output asset to contain the results of the job. 
		    // This output is specified as AssetCreationOptions.None, which 
		    // means the output asset is not encrypted. 
		    task.OutputAssets.AddNew("Output asset",
		        AssetCreationOptions.None);
		
		    job.StateChanged += new EventHandler<JobStateChangedEventArgs>(JobStateChanged);
		    job.Submit();
		    job.GetExecutionProgressTask(CancellationToken.None).Wait();
		
		    return job.OutputMediaAssets[0];
		}
		
		private static void JobStateChanged(object sender, JobStateChangedEventArgs e)
		{
		    Console.WriteLine("Job state changed event:");
		    Console.WriteLine("  Previous state: " + e.PreviousState);
		    Console.WriteLine("  Current state: " + e.CurrentState);
		    switch (e.CurrentState)
		    {
		        case JobState.Finished:
		            Console.WriteLine();
		            Console.WriteLine("Job is finished. Please wait while local tasks or downloads complete...");
		            break;
		        case JobState.Canceling:
		        case JobState.Queued:
		        case JobState.Scheduled:
		        case JobState.Processing:
		            Console.WriteLine("Please wait...\n");
		            break;
		        case JobState.Canceled:
		        case JobState.Error:
		
		            // Cast sender as a job.
		            IJob job = (IJob)sender;
		
		            // Display or log error details as needed.
		            break;
		        default:
		            break;
		    }
		}
		
		
		private static IMediaProcessor GetLatestMediaProcessorByName(string mediaProcessorName)
		{
		    var processor = _context.MediaProcessors.Where(p => p.Name == mediaProcessorName).
		    ToList().OrderBy(p => new Version(p.Version)).LastOrDefault();
		
		    if (processor == null)
		        throw new ArgumentException(string.Format("Unknown media processor", mediaProcessorName));
		
		    return processor;
		}


[AZURE.INCLUDE [media-services-user-voice-include](../includes/media-services-user-voice-include.md)]

##另请参阅 

[如何使用媒体编码器标准通过 .NET 来生成缩略图](/documentation/articles/media-services-dotnet-generate-thumbnail-with-mes)
[媒体服务编码概述](/documentation/articles/media-services-encode-asset)

<!---HONumber=Mooncake_0104_2016-->