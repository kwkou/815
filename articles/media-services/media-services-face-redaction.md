<properties
    pageTitle="使用 Azure 媒体分析进行面部修订 | Azure"
    description="本主题演示如何使用 Azure 媒体分析检测面部。"
    services="media-services"
    documentationcenter=""
    author="juliako"
    manager="erikre"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="5b6d8b8c-5f4d-4fef-b3d6-dc22c6b5a0f5"
    ms.service="media-services"
    ms.workload="media"
    ms.tgt_pltfrm="na"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.date="04/16/2017"
    wacn.date="05/08/2017"
    ms.author="juliako;"
    ms.sourcegitcommit="2c4ee90387d280f15b2f2ed656f7d4862ad80901"
    ms.openlocfilehash="8c0a1ed057de41dc4ea7f6fd526c1c87bf8271de"
    ms.lasthandoff="04/28/2017" />

# <a name="redact-faces-with-azure-media-analytics"></a>使用 Azure 媒体分析进行面部修订
## <a name="overview"></a>概述
**Azure 媒体修订器**是一种 [Azure 媒体分析](/documentation/articles/media-services-analytics-overview/)媒体处理器 (MP)，可用于在云中进行可缩放的面部修订。 使用面部修订，可对视频进行修改，使所选个人的面部模糊显示。 用户可能想要在公共安全和新闻媒体场景中使用面部修订服务。 对于时长仅几分钟但包含多张面孔的镜头，进行手动面部修订可能需要几个小时，但使用此服务仅需几个简单步骤即可完成该过程。 有关详细信息，请参阅[此](https://azure.microsoft.com/zh-cn/blog/azure-media-redactor/)博客。

本主题提供有关 **Azure 媒体修订器**的详细信息，并演示如何通过适用于 .NET 的媒体服务 SDK 使用它。

**Azure 媒体修订器** MP 目前以预览版提供。 它在所有公共 Azure 区域以及美国政府和中国数据中心中可用。 当前，此预览版免费。 

## <a name="face-redaction-modes"></a>面部修订模式
面部修订的工作方式是：检测每一帧视频中的面部，并跟踪之前和之后的面部对象，以便同一个人在其他角度也模糊显示。 自动修订过程非常复杂，并且无法始终产生 100% 符合要求的输出，因此，媒体分析提供了几种修改最终输出的方式。

除了完全自动模式外，还可使用双步工作流通过 ID 列表选择/取消选找到的面部。 此外，为了对每一帧进行任意调整，MP 使用 JSON 格式的元数据文件。 此工作流拆分为“分析”和“修订”模式。 可将这两个模式组合为在一个作业中运行两项任务的单个过程；此模式称为“组合”。

### <a name="combined-mode"></a>组合模式
这将自动生成经过修订的 mp4，而无需任何手动输入。

| 阶段 | 文件名 | 说明 |
| --- | --- | --- |
| 输入资产 |foo.bar |WMV、MOV 或 MP4 格式的视频 |
| 输入配置 |作业配置预设 |{'version':'1.0', 'options': {'mode':'combined'}} |
| 输出资产 |foo_redacted.mp4 |进行了模糊处理的视频 |

#### <a name="input-example"></a>输入示例：
[观看此视频](http://ampdemo.azureedge.net/?url=http%3A%2F%2Freferencestream-samplestream.streaming.mediaservices.windows.net%2Fed99001d-72ee-4f91-9fc0-cd530d0adbbc%2FDancing.mp4)

#### <a name="output-example"></a>输出示例：
[观看此视频](http://ampdemo.azureedge.net/?url=http%3A%2F%2Freferencestream-samplestream.streaming.mediaservices.windows.net%2Fc6608001-e5da-429b-9ec8-d69d8f3bfc79%2Fdance_redacted.mp4)

### <a name="analyze-mode"></a>分析模式
双步工作流的 **分析** 步骤使用视频输入，并生成表示面部位置的 JSON 文件，以及显示每个检测到的面部的 jpg 图像。

| 阶段 | 文件名 | 说明 |
| --- | --- | --- |
| 输入资产 |foo.bar |WMV、MPV 或 MP4 格式的视频 |
| 输入配置 |作业配置预设 |{'version':'1.0', 'options': {'mode':'analyze'}} |
| 输出资产 |foo_annotations.json |JSON 格式的面部位置批注数据。 用户可编辑此数据，以修改模糊边界框。 请查看以下示例。 |
| 输出资产 |foo_thumb%06d.jpg [foo_thumb000001.jpg, foo_thumb000002.jpg] |裁剪后的 jpg 文件，显示每个检测到的面部，其中的数字指示面部的标签 ID |

#### <a name="output-example"></a>输出示例：
	{
	  "version": 1,
	  "timescale": 50,
	  "offset": 0,
	  "framerate": 25.0,
	  "width": 1280,
	  "height": 720,
	  "fragments": [
	    {
	      "start": 0,
	      "duration": 2,
	      "interval": 2,
	      "events": [
	        [  
	          {
	            "id": 1,
	            "x": 0.306415737,
	            "y": 0.03199235,
	            "width": 0.15357475,
	            "height": 0.322126418
	          },
	          {
	            "id": 2,
	            "x": 0.5625317,
	            "y": 0.0868245438,
	            "width": 0.149155334,
	            "height": 0.355517566
	          }
	        ]
	      ]
	    },
    	… truncated

### <a name="redact-mode"></a>修订模式
工作流的第二步使用更大数量的输入，这些输入必须合并为单个资产。

这包括要模糊处理的 ID 的列表、原始视频和批注 JSON。 此模式使用批注来对输入视频进行模糊处理。

“分析”步骤的输出不包括原始视频。 需要将该视频上传到“修订”模式任务的输入资产中，并将其选作主文件。

| 阶段 | 文件名 | 说明 |
| --- | --- | --- |
| 输入资产 |foo.bar |WMV、MPV 或 MP4 格式的视频。 与步骤 1 中相同的视频。 |
| 输入资产 |foo_annotations.json |第一阶段中的批注元数据文件，包含可选的修改。 |
| 输入资产 |foo_IDList.txt（可选） |要进行修订的可选面部 ID 列表，以新行进行分隔。 如果留空，则模糊所有面部。 |
| 输入配置 |作业配置预设 |{'version':'1.0', 'options': {'mode':'redact'}} |
| 输出资产 |foo_redacted.mp4 |基于批注进行了模糊处理的视频 |

#### <a name="example-output"></a>示例输出
这是来自选择了一个 ID 的 ID 列表的输出。

[观看此视频](http://ampdemo.azureedge.net/?url=http%3A%2F%2Freferencestream-samplestream.streaming.mediaservices.windows.net%2Fad6e24a2-4f9c-46ee-9fa7-bf05e20d19ac%2Fdance_redacted1.mp4)

示例 foo_IDList.txt
 
     1
     2
     3
 
## <a name="attribute-descriptions"></a>属性说明
修订 MP 提供高精确度的面部位置检测和跟踪功能，可在一个视频帧中检测到最多 64 张人脸。 正面的面部可提供最佳效果，而检测和跟踪侧面的面部和较小的面部（小于或等于 24x24 像素）可能具有一定难度。

对于已检测并跟踪到的面部，将返回其坐标，以指示面部位置，还将返回面部 ID 编号，以表示正在跟踪该人员的面部。 在正面面部长时间于帧中消失或重叠的情况下，面部 ID 编号很容易重置，导致某些人员被分配多个 ID。

有关属性的详细说明，请参阅[使用 Azure 媒体分析检测面部和情感](/documentation/articles/media-services-face-and-emotion-detection/)主题。

## <a name="sample-code"></a>代码示例
以下程序演示如何：

1. 创建资产并将媒体文件上传到资产。
2. 基于包含以下 json 预设的配置文件创建含有面部修订任务的作业。 
   
        {'version':'1.0', 'options': {'mode':'combined'}}
    
3. 下载输出 JSON 文件。 

	        using System;
	        using System.Configuration;
	        using System.IO;
	        using System.Linq;
	        using Microsoft.WindowsAzure.MediaServices.Client;
	        using System.Threading;
	        using System.Threading.Tasks;

	        namespace FaceRedaction
	        {
	            class Program
	            {
	                // Read values from the App.config file.
	                private static readonly string _mediaServicesAccountName =
	                    ConfigurationManager.AppSettings["MediaServicesAccountName"];
	                private static readonly string _mediaServicesAccountKey =
	                    ConfigurationManager.AppSettings["MediaServicesAccountKey"];
	            private static readonly String _defaultScope = "urn:WindowsAzureMediaServices";

	            // Azure China uses a different API server and a different ACS Base Address from the Global.
	            private static readonly String _chinaApiServerUrl = "https://wamsshaclus001rest-hs.chinacloudapp.cn/API/";
	            private static readonly String _chinaAcsBaseAddressUrl = "https://wamsprodglobal001acs.accesscontrol.chinacloudapi.cn";

	                // Field for service context.
	                private static CloudMediaContext _context = null;
	                private static MediaServicesCredentials _cachedCredentials = null;
	            private static Uri _apiServer = null;

	                static void Main(string[] args)
	                {

	                    // Create and cache the Media Services credentials in a static class variable.
	                        _cachedCredentials = new MediaServicesCredentials(
	                                _mediaServicesAccountName,
	                                _mediaServicesAccountKey,
	                                _defaultScope,
	                                _chinaAcsBaseAddressUrl);

	                // Create the API server Uri
	                _apiServer = new Uri(_chinaApiServerUrl);

	                        // Used the chached credentials to create CloudMediaContext.
	                        _context = new CloudMediaContext(_apiServer, _cachedCredentials);

	                    // Run the FaceRedaction job.
	                    var asset = RunFaceRedactionJob(@"C:\supportFiles\FaceRedaction\SomeFootage.mp4",
	                                                @"C:\supportFiles\FaceRedaction\config.json");

	                    // Download the job output asset.
	                    DownloadAsset(asset, @"C:\supportFiles\FaceRedaction\Output");
	                }

	                static IAsset RunFaceRedactionJob(string inputMediaFilePath, string configurationFile)
	                {
	                    // Create an asset and upload the input media file to storage.
	                    IAsset asset = CreateAssetAndUploadSingleFile(inputMediaFilePath,
	                        "My Face Redaction Input Asset",
	                        AssetCreationOptions.None);

	                    // Declare a new job.
	                    IJob job = _context.Jobs.Create("My Face Redaction Job");

	                    // Get a reference to Azure Media Redactor.
	                    string MediaProcessorName = "Azure Media Redactor";

	                    var processor = GetLatestMediaProcessorByName(MediaProcessorName);

	                    // Read configuration from the specified file.
	                    string configuration = File.ReadAllText(configurationFile);

	                    // Create a task with the encoding details, using a string preset.
	                    ITask task = job.Tasks.AddNew("My Face Redaction Task",
	                        processor,
	                        configuration,
	                        TaskOptions.None);

	                    // Specify the input asset.
	                    task.InputAssets.Add(asset);

	                    // Add an output asset to contain the results of the job.
	                    task.OutputAssets.AddNew("My Face Redaction Output Asset", AssetCreationOptions.None);

	                    // Use the following event handler to check job progress.  
	                    job.StateChanged += new EventHandler<JobStateChangedEventArgs>(StateChanged);

	                    // Launch the job.
	                    job.Submit();

	                    // Check job execution and wait for job to finish.
	                    Task progressJobTask = job.GetExecutionProgressTask(CancellationToken.None);

	                    progressJobTask.Wait();

	                    // If job state is Error, the event handling
	                    // method for job progress should log errors.  Here we check
	                    // for error state and exit if needed.
	                    if (job.State == JobState.Error)
	                    {
	                        ErrorDetail error = job.Tasks.First().ErrorDetails.First();
	                        Console.WriteLine(string.Format("Error: {0}. {1}",
	                                                        error.Code,
	                                                        error.Message));
	                        return null;
	                    }

	                    return job.OutputMediaAssets[0];
	                }

	                static IAsset CreateAssetAndUploadSingleFile(string filePath, string assetName, AssetCreationOptions options)
	                {
	                    IAsset asset = _context.Assets.Create(assetName, options);

	                    var assetFile = asset.AssetFiles.Create(Path.GetFileName(filePath));
	                    assetFile.Upload(filePath);

	                    return asset;
	                }

	                static void DownloadAsset(IAsset asset, string outputDirectory)
	                {
	                    foreach (IAssetFile file in asset.AssetFiles)
	                    {
	                        file.Download(Path.Combine(outputDirectory, file.Name));
	                    }
	                }

	                static IMediaProcessor GetLatestMediaProcessorByName(string mediaProcessorName)
	                {
	                    var processor = _context.MediaProcessors
	                        .Where(p => p.Name == mediaProcessorName)
	                        .ToList()
	                        .OrderBy(p => new Version(p.Version))
	                        .LastOrDefault();

	                    if (processor == null)
	                        throw new ArgumentException(string.Format("Unknown media processor",
	                                                                   mediaProcessorName));

	                    return processor;
	                }

	                static private void StateChanged(object sender, JobStateChangedEventArgs e)
	                {
	                    Console.WriteLine("Job state changed event:");
	                    Console.WriteLine("  Previous state: " + e.PreviousState);
	                    Console.WriteLine("  Current state: " + e.CurrentState);

	                    switch (e.CurrentState)
	                    {
	                        case JobState.Finished:
	                            Console.WriteLine();
	                            Console.WriteLine("Job is finished.");
	                            Console.WriteLine();
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
	                            // LogJobStop(job.Id);
	                            break;
	                        default:
	                            break;
	                    }
	                }

	            }
	        }

## <a name="related-links"></a>相关链接
[Azure 媒体服务分析概述](/documentation/articles/media-services-analytics-overview/)

[Azure 媒体分析演示](http://azuremedialabs.azurewebsites.net/demos/Analytics.html)
<!--Update_Description:add output sample;add anchors to subtitles-->