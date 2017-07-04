<properties
    pageTitle="使用 .NET 通过 Media Encoder Standard 对资产进行编码 | Azure"
    description="本主题介绍如何使用 .NET 通过 Media Encoder Standard 对资产进行编码。"
    services="media-services"
    documentationcenter=""
    author="juliako"
    manager="erikre"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="03431b64-5518-478a-a1c2-1de345999274"
    ms.service="media-services"
    ms.workload="media"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/21/2017"
    wacn.date="05/08/2017"
    ms.author="juliako;anilmur"
    ms.sourcegitcommit="2c4ee90387d280f15b2f2ed656f7d4862ad80901"
    ms.openlocfilehash="c335c192ee833007363ae8a8e6607072441f1222"
    ms.lasthandoff="04/28/2017" />

# <a name="encode-an-asset-with-media-encoder-standard-using-net"></a>使用 .NET 通过媒体编码器标准版对资产进行编码
编码作业是媒体服务中最常见的处理操作之一。 可通过创建编码作业将媒体文件从一种编码转换为另一种编码。 进行编码时，可以使用媒体服务内置的 Media Encoder。 你也可以使用媒体服务合作伙伴提供的编码器；可通过 Azure 应用商店获取第三方编码器。 

本主题介绍如何使用 .NET 通过媒体编码器标准 (MES) 对资产进行编码。 Media Encoder Standard 使用[此处](/documentation/articles/media-services-mes-presets-overview/)所述的编码器预设之一进行配置。

建议始终将夹层文件编码为自适应比特率 MP4 集，然后使用[动态打包](/documentation/articles/media-services-dynamic-packaging-overview/)将该集转换为所需的格式。 

如果输出资产已经过存储加密，则必须配置资产传送策略。 有关详细信息，请参阅[配置资产传送策略](/documentation/articles/media-services-dotnet-configure-asset-delivery-policy/)。

> [AZURE.NOTE]
> MES 将生成一个输出文件，其名称包含输入文件名的前 32 个字符。 该名称基于预设文件中指定的内容。 例如，"FileName": "{Basename}_{Index}{Extension}"。 {Basename} 替换为输入文件名的前 32 个字符。
> 
> 

### <a name="mes-formats"></a>MES 格式
[格式和编解码器](/documentation/articles/media-services-media-encoder-standard-formats/)

### <a name="mes-presets"></a>MES 预设
Media Encoder Standard 使用[此处](/documentation/articles/media-services-mes-presets-overview/)所述的编码器预设之一进行配置。

### <a name="input-and-output-metadata"></a>输入和输出元数据
如果你使用 MES 为输入资产（或资产）编码，在该编码任务成功完成时，你便能获取输出资产。 输出资产包含视频、音频、缩略图、清单等等，具体视你使用的编码预设而定。

输出资产还包含提供输入资产相关元数据的文件。 元数据 XML 文件的名称采用下列格式：<asset_id>_metadata.xml（例如，41114ad3-eb5e-4c57-8d92-5354e2b7d4a4_metadata.xml），其中 <asset_id> 是输入资产的 AssetId 值。 [此处](/documentation/articles/media-services-input-metadata-schema/)描述了此输入元数据 XML 的架构。

输出资产还包含提供输出资产相关元数据的文件。 元数据 XML 文件的名称采用下列格式：<source_file_name>_manifest.xml（例如，BigBuckBunny_manifest.xml）。 [此处](/documentation/articles/media-services-output-metadata-schema/)描述了此输出元数据 XML 的架构。

如果想要检查这两个元数据文件中的任意一个，可以创建 SAS 定位器并将文件下载到本地计算机。 你可以就如何创建 SAS 定位器并下载使用媒体服务 .NET SDK 扩展的文件找到相关示例。

## <a name="example"></a>示例
以下代码示例使用媒体服务 .NET SDK 执行下列任务：

* 创建编码作业。
* 获取对媒体编码器标准版编码器的引用。
* 指定使用[自适应流式处理](/documentation/articles/media-services-autogen-bitrate-ladder-with-mes/)预设。 
* 将一个编码任务添加到该作业。 
* 指定要编码的输入资产。
* 创建将包含所编码资产的输出资产。
* 添加事件处理程序以检查作业进度。
* 提交作业。
		
		static public IAsset EncodeToAdaptiveBitrateMP4Set(IAsset asset)
		{
		    // Declare a new job.
		    IJob job = _context.Jobs.Create("Media Encoder Standard Job");
		    // Get a media processor reference, and pass to it the name of the 
		    // processor to use for the specific task.
		    IMediaProcessor processor = GetLatestMediaProcessorByName("Media Encoder Standard");
		

		    // Create a task with the encoding details, using a string preset.
		    // In this case "Adaptive Streaming" preset is used.
		    ITask task = job.Tasks.AddNew("My encoding task",
		        processor,
		        "Adaptive Streaming",
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



## <a name="see-also"></a>另请参阅
- [如何使用 Media Encoder Standard 通过 .NET 来生成缩略图](/documentation/articles/media-services-dotnet-generate-thumbnail-with-mes/)
- [媒体服务编码概述](/documentation/articles/media-services-encode-asset/)
<!--Update_Description:update MSDN links to A.cn links;add anchors to subtitles-->
