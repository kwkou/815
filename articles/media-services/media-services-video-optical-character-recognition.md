<properties
    pageTitle="使用 Azure 媒体分析 OCR 将文本数字化 | Azure"
    description="Azure 媒体分析 OCR（光学字符识别）可让你将视频文件中的文本内容转换成可编辑、可搜索的数字文本。这可让你从媒体的视频信号中自动提取有意义的元数据。"
    services="media-services"
    documentationcenter=""
    author="juliako"
    manager="erikre"
    editor="" />
<tags
    ms.assetid="307c196e-3a50-4f4b-b982-51585448ffc6"
    ms.service="media-services"
    ms.workload="media"
    ms.tgt_pltfrm="na"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.date="02/01/2017"
    wacn.date="03/10/2017"
    ms.author="juliako" />  


# 使用 Azure 媒体分析将视频文件中的文本内容转换为数字文本
## 概述
如果你需要提取视频文件的文本内容，并生成可编辑、可搜索的数字文本，则应该使用 Azure 媒体分析 OCR（光学字符识别）。此 Azure 媒体处理器可检测视频文件的文本内容并生成文本文件供你使用。OCR 可让你从媒体的视频信号中自动提取有意义的元数据。

与搜索引擎配合使用时，可以根据文本轻松编制媒体的索引，并增强发现内容的能力。这在包含大量文本的视频（例如视频录制或者幻灯片演示屏幕截图）中非常有用。Azure OCR 媒体处理器已针对数字文本进行优化。

**Azure 媒体 OCR** 媒体处理器目前以预览版提供。

本主题提供有关 **Azure 媒体 OCR** 的详细信息，并演示如何将其与适用于 .NET 的媒体服务 SDK 结合使用。有关其他信息和示例，请参阅[这篇博客](https://azure.microsoft.com/blog/announcing-video-ocr-public-preview-new-config/)。

##OCR 输入文件

视频文件。目前支持以下格式：MP4、MOV 和 WMV。

##任务配置 

任务配置（预设）在使用 **Azure 媒体 OCR** 创建任务时，必须使用 JSON 或 XML 指定配置预设。

>[AZURE.NOTE]
在高度/宽度上，OCR 引擎仅将最小 40 像素到最大 32000 像素的图像区域视为有效输入。
>

### 属性说明
| 属性名称 | 说明 |
| --- | --- |
| 语言 |（可选）描述要查找的文本的语言。下列其中一项：AutoDetect（默认值）、Arabic、ChineseSimplified、ChineseTraditional、Czech Danish、Dutch、English、Finnish、French、German、Greek、Hungarian、Italian、Japanese、Korean、Norwegian、Polish、Portuguese、Romanian、Russian、SerbianCyrillic、SerbianLatin、Slovak、Spanish、Swedish、Turkish。 |
| TextOrientation |（可选）描述要查找的文本的方向。“Left”表示所有字母顶部朝向左侧。默认文本（例如书籍中出现的文本）的方向为“Up”。下列其中一项：AutoDetect（默认值）、Up、Right、Down、Left。 |
| TimeInterval |<p>（可选）描述采样率。默认值为每隔 1/2 秒。</p><p>JSON 格式 – HH:mm:ss.SSS（默认值 00:00:00.500）</p><p>XML 格式 – W3C XSD 持续时间基元（默认值 PT0.5） </p>|
| DetectRegions |<p>（可选）DetectRegion 对象的数组，指定视频帧内要检测文本的区域。</p><p>DetectRegion 对象由以下四个整数值构成：</p><p>Left - 从左边缘算起的像素</p><p>Top - 从上边缘算起的像素</p><p>Width – 以像素为单位的区域宽度</p><p>Height – 以像素为单位的区域高度</p> |

####JSON 预设示例
	 	
	{
	    'Version':'1.0', 
	    'Options': 
	    {
	    'Language':'English', 
	        'TimeInterval':'00:00:01.5',
	        'DetectRegions': 
	         [
	            {'Left':'1','Top':'1','Width':'1','Height':'1'},
	            {'Left':'2','Top':'2','Width':'2','Height':'2'}
	         ],
	        'TextOrientation':'Up'
	    }
	}

####XML 预设示例

	<?xml version=""1.0"" encoding=""utf-16""?>
	<VideoOcrPreset xmlns:xsi=""http://www.w3.org/2001/XMLSchema-instance"" xmlns:xsd=""http://www.w3.org/2001/XMLSchema"" Version=""1.0"" xmlns=""http://www.windowsazure.com/media/encoding/Preset/2014/03"">
	  <Options>
	     <Language>English</Language>
	     <TimeInterval>PT1.5S</TimeInterval>
	     <DetectRegions>
	         <DetectRegion>
	               <Left>1</Left>
	               <Top>1</Top>
	               <Width>1</Width>
	               <Height>1</Height>
	        </DetectRegion>
	   </DetectRegions>
	   <TextOrientation>Up</TextOrientation>
	  </Options>
	</VideoOcrPreset>

##OCR 输出文件

OCR 媒体处理器的输出是一个 JSON 文件。

###输出 JSON 文件中的元素

视频 OCR 输出针对视频中的字符提供时间分段数据。你可以使用属性（例如语言或方向）来锁定你想要分析的文本。

输出包含以下属性：

元素|说明
---|---
时间刻度|视频每秒的“刻度”数
Offset|时间戳的时间偏移量。在版本 1.0 的视频 API 中，此属性始终为 0。
Framerate|视频的每秒帧数
width|以像素为单位的视频宽度
height|以像素为单位的视频高度
Fragments|将元数据堆积成的基于时间的视频块数组
start|片段的开始时间（以“刻度”为单位）
duration|片段的长度（以“刻度”为单位）
interval|给定片段中每个事件的间隔
events|包含区域的数组
region|表示检测到的单词或短语的对象
语言|区域中检测到的文本的语言
orientation|区域中检测到的文本的方向
lines|区域中检测到的文本的行数组
text|实际文本

###JSON 输出示例

以下输出示例包含常规视频信息和多个视频片段。每个视频片段包含 OCR MP 检测到的每个区域及其语言和文本方向。区域还包含此区域中的每个单词行，以及该行的文本、位置及其中每个单词的信息（单词内容、位置和置信度）。下面是一个示例，我在其中嵌入了一些注释。

	{
		"version": 1, 
		"timescale": 90000, 
		"offset": 0, 
		"framerate": 30, 
		"width": 640, 
		"height": 480,  // general video information
		"fragments": [
		    {
		        "start": 0, 
		        "duration": 180000, 
		        "interval": 90000,  // the time information about this fragment
		        "events": [
		            [
		               { 
		                    "region": { // the detected region array in this fragment 
		                        "language": "English",  // region language
		                        "orientation": "Up",  // text orientation
		                        "lines": [  // line information array in this region, including the text and the position
		                            {
		                                "text": "One Two", 
		                                "left": 10, 
		                                "top": 10, 
		                                "right": 210, 
		                                "bottom": 110, 
		                                "word": [  // word information array in this line
		                                    {
		                                        "text": "One", 
		                                        "left": 10, 
		                                        "top": 10, 
		                                        "right": 110, 
		                                        "bottom": 110, 
		                                        "confidence": 900
		                                    }, 
		                                    {
		                                        "text": "Two", 
		                                        "left": 110, 
		                                        "top": 10, 
		                                        "right": 210, 
		                                        "bottom": 110, 
		                                        "confidence": 910
		                                    }
		                                ]
		                            }
		                        ]
		                    }
		                }
		            ]
		        ]
		    }
		]
	}

## 代码示例

以下程序演示如何：

1. 创建资产并将媒体文件上传到资产。
1. 使用 OCR 配置/预设文件创建作业。
1. 下载输出 JSON 文件。
		 
        using System;
		using System.Configuration;
		using System.IO;
		using System.Linq;
		using Microsoft.WindowsAzure.MediaServices.Client;
		using System.Threading;
		using System.Threading.Tasks;
		
		namespace OCR
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
		
		            // Run the OCR job.
		            var asset = RunOCRJob(@"C:\supportFiles\OCR\presentation.mp4",
		                                        @"C:\supportFiles\OCR\config.json");
		
		            // Download the job output asset.
		            DownloadAsset(asset, @"C:\supportFiles\OCR\Output");
		        }
		
		        static IAsset RunOCRJob(string inputMediaFilePath, string configurationFile)
		        {
		            // Create an asset and upload the input media file to storage.
		            IAsset asset = CreateAssetAndUploadSingleFile(inputMediaFilePath,
		                "My OCR Input Asset",
		                AssetCreationOptions.None);
		
		            // Declare a new job.
		            IJob job = _context.Jobs.Create("My OCR Job");
		
		            // Get a reference to Azure Media OCR.
		            string MediaProcessorName = "Azure Media OCR";
		
		            var processor = GetLatestMediaProcessorByName(MediaProcessorName);
		
		            // Read configuration from the specified file.
		            string configuration = File.ReadAllText(configurationFile);
		
		            // Create a task with the encoding details, using a string preset.
		            ITask task = job.Tasks.AddNew("My OCR Task",
		                processor,
		                configuration,
		                TaskOptions.None);
		
		            // Specify the input asset.
		            task.InputAssets.Add(asset);
		
		            // Add an output asset to contain the results of the job.
		            task.OutputAssets.AddNew("My OCR Output Asset", AssetCreationOptions.None);
		
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




##相关链接

[Azure 媒体服务分析概述](/documentation/articles/media-services-analytics-overview/)

<!---HONumber=Mooncake_0306_2017-->
<!--Update_Description: add link reference to media service analytics-->