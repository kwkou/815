<properties
	pageTitle="使用 Azure 媒体分析检测动作 | Azure"
	description="Azure 媒体动作检测器媒体处理器 (MP) 可让你在冗长且平淡的视频中有效识别出你感兴趣的部分。"
	services="media-services"
	documentationCenter=""
	authors="juliako"
	manager="erikre"
	editor=""/>  


<tags
	ms.service="media-services"
	ms.workload="media"
	ms.tgt_pltfrm="na"
	ms.devlang="dotnet"
	ms.topic="article"
	ms.date="10/10/2016"  
	wacn.date="11/25/2016"
	ms.author="milanga;juliako;"/>  

 
# 使用 Azure 媒体分析检测动作

##概述

**Azure 媒体动作检测器**媒体处理器 (MP) 可让你在冗长且平淡的视频中有效识别出你感兴趣的部分。可以对静态相机数据片段使用动作检测，以识别视频中有动作的部分。它会生成 JSON 文件，其中包含带时间戳的元数据，以及发生事件的边界区域。

此技术面向安全视频提要，它可以将动作分类为相关事件和误报（例如阴影或光源变化）。这样，你就可以在无需查看无止境的不相关事件的情况下，从相机输出生成安全警报，并从长时间的监控视频中提取感兴趣的片段。

**Azure 媒体动作检测器** MP 目前以预览版提供。

本主题提供有关 **Azure 媒体动作检测器**的详细信息，并演示如何通过适用于 .NET 的媒体服务 SDK 使用它。


##动作检测器输入文件

视频文件。目前支持以下格式：MP4、MOV 和 WMV。

##任务配置（预设）

在使用 **Azure 媒体动作检测器**创建任务时，必须指定配置预设。

###参数

你可以使用以下参数：

名称|选项|说明|默认
---|---|---|---
sensitivityLevel|字符串：'low'、'medium'、'high'|设置报告动作情况的敏感度级别。调整此项是为了调整误报数量。|'medium'
frameSamplingValue|正整数|设置算法的运行频率。1 等于每个帧，2 是指每隔一个帧，如此类推。|1
detectLightChange|布尔值：'true'、'false'|设置是否在结果中报告轻微的更改|'False'
mergeTimeThreshold|Xs-time: Hh:mm:ss<br/>示例：00:00:03|指定动作事件之间的时间窗口，其中的 2 个事件将组合成 1 个事件进行报告。|00:00:00
detectionZones|检测区域数组：<br/>- 检测区域是包含 3 个或 3 个以上点的数组<br/>- 点是指由从 0 到 1 的 x 和 y 组成的坐标。|描述要使用的多边形检测区域的列表。<br/>报告结果时，将使用区域作为 ID，第一个的格式为 'id':0|单个区域，涵盖整个帧。

###JSON 示例

	
	{
	  'version': '1.0',
	  'options': {
	    'sensitivityLevel': 'medium',
	    'frameSamplingValue': 1,
	    'detectLightChange': 'False',
	    "mergeTimeThreshold":
	    '00:00:02',
	    'detectionZones': [
	      [
	        {'x': 0, 'y': 0},
	        {'x': 0.5, 'y': 0},
	        {'x': 0, 'y': 1}
	       ],
	      [
	        {'x': 0.3, 'y': 0.3},
	        {'x': 0.55, 'y': 0.3},
	        {'x': 0.8, 'y': 0.3},
	        {'x': 0.8, 'y': 0.55},
	        {'x': 0.8, 'y': 0.8},
	        {'x': 0.55, 'y': 0.8},
	        {'x': 0.3, 'y': 0.8},
	        {'x': 0.3, 'y': 0.55}
	      ]
	    ]
	  }
	}


##动作检测器输出文件

动作检测作业将在输出资产中返回 JSON 文件，该文件描述视频中的动作警报和类别。该文件将包含有关在视频中检测到的动作的时间和持续时间的信息。

一旦固定背景视频（例如监控视频）中出现运动对象，动作检测器 API 将提供指示器。动作检测器经过训练可减少误报（例如光源和阴影变化）。当前算法限制包括夜视视频、半透明对象和小对象。

###<a id="output_elements"></a>输出 JSON 文件中的元素

>[AZURE.NOTE]在最新版本中，输出 JSON 格式已更改，对某些客户来说可以说是重大更改。

下表描述了输出 JSON 文件的元素。

元素|说明
---|---
版本|这是指视频 API 的版本。当前版本为 2。
时间刻度|视频每秒的“刻度”数。
Offset|时间戳的时间偏移量（以“刻度”为单位）。在版本 1.0 的视频 API 中，此属性始终为 0。在我们将来支持的方案中，此值可能会更改。
Framerate|视频的每秒帧数。
Width, Height|表示视频的宽度和高度（以像素为单位）。
开始|开始时间戳（以“刻度”为单位）。
持续时间|事件的长度（以“刻度”为单位）。
时间间隔|事件中每个条目的间隔（以“刻度”为单位）。
事件|每个事件片段包含在该持续时间内检测到的动作。
类型|在当前版本中，对于一般动作，该属性始终为“2”。此标签可让视频 API 在将来的版本中灵活地为动作分类。
RegionID|如上所述，在此版本中此属性始终为 0。此标签可让视频 API 在将来的版本中灵活地查找各区域中的动作。
区域|表示你关注的动作在视频中的区域。<br/><br/>-"id" 表示区域 – 这个版本中只有一个，即 ID 0。<br/>-"type" 表示你关注的动作所在区域的形状。目前支持 "rectangle" 和 "polygon"。<br/> 如果指定了 "rectangle"，则区域具有以 X、Y、宽度及高度表示的维。X 和 Y 坐标表示规范化 0.0 到 1.0 比例中的区域的左上角 XY 坐标。宽度和高度表示规范化 0.0 到 1.0 比例中的区域的大小。在当前版本中，X、Y、宽度和高度始终固定为 0、0、1、1。<br/>如果你指定了 "polygon"，则区域的维度以点来表示。<br/>
Fragments|元数据划分成称为“片段”的不同段。每个片段包含开始时间、持续时间、间隔数字和事件。没有事件的片段表示在该开始时间和持续时间内没有检测到任何动作。
括号|每个括号表示事件中的单个间隔。如果该间隔显示空括号，则表示没有检测到动作。
位置|事件下的此新项列出发生动作的位置。这比检测区域更具体。

下面是 JSON 输出示例

	{
	  "version": 2,
	  "timescale": 23976,
	  "offset": 0,
	  "framerate": 24,
	  "width": 1280,
	  "height": 720,
	  "regions": [
	    {
	      "id": 0,
	      "type": "polygon",
	      "points": [{'x': 0, 'y': 0},
	        {'x': 0.5, 'y': 0},
	        {'x': 0, 'y': 1}]
	    }
	  ],
	  "fragments": [
	    {
	      "start": 0,
	      "duration": 226765
	    },
	    {
	      "start": 226765,
	      "duration": 47952,
	      "interval": 999,
	      "events": [
	        [
	          {
	            "type": 2,
	            "typeName": "motion",
	            "locations": [
	              {
	                "x": 0.004184,
	                "y": 0.007463,
	                "width": 0.991667,
	                "height": 0.985185
	              }
	            ],
	            "regionId": 0
	          }
	        ],
	
	…
##限制

- 支持的输入视频格式包括 MP4、MOV 和 WMV。
- 动作检测已针对固定背景视频优化。算法专注于降低误报，例如光源变化和阴影。
- 某些动作可能因技术难题而无法检测到，例如夜视视频、半透明对象和小对象。


## 代码示例

以下程序演示如何：

1. 创建资产并将媒体文件上传到资产。
1. 使用基于包含以下 json 预设的配置文件的视频动作检测任务创建一个作业。
					
		{
		  'Version': '1.0',
		  'Options': {
		    'SensitivityLevel': 'medium',
		    'FrameSamplingValue': 1,
		    'DetectLightChange': 'False',
		    "MergeTimeThreshold":
		    '00:00:02',
		    'DetectionZones': [
		      [
		        {'x': 0, 'y': 0},
		        {'x': 0.5, 'y': 0},
		        {'x': 0, 'y': 1}
		       ],
		      [
		        {'x': 0.3, 'y': 0.3},
		        {'x': 0.55, 'y': 0.3},
		        {'x': 0.8, 'y': 0.3},
		        {'x': 0.8, 'y': 0.55},
		        {'x': 0.8, 'y': 0.8},
		        {'x': 0.55, 'y': 0.8},
		        {'x': 0.3, 'y': 0.8},
		        {'x': 0.3, 'y': 0.55}
		      ]
		    ]
		  }
		}

1. 下载输出 JSON 文件。
		 
        using System;
		using System.Configuration;
		using System.IO;
		using System.Linq;
		using Microsoft.WindowsAzure.MediaServices.Client;
		using System.Threading;
		using System.Threading.Tasks;
		
		namespace VideoMotionDetection
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
				private static readonly String _chinaAcsBaseAddressUrl = "https://wamsprodglobal001acs.accesscontrol.chinacloudapi.cn";
		
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
		
		            // Run the VideoMotionDetection job.
		            var asset = RunVideoMotionDetectionJob(@"C:\supportFiles\VideoMotionDetection\BigBuckBunny.mp4",
		                                        @"C:\supportFiles\VideoMotionDetection\config.json");
		
		            // Download the job output asset.
		            DownloadAsset(asset, @"C:\supportFiles\VideoMotionDetection\Output");
		        }
		
		        static IAsset RunVideoMotionDetectionJob(string inputMediaFilePath, string configurationFile)
		        {
		            // Create an asset and upload the input media file to storage.
		            IAsset asset = CreateAssetAndUploadSingleFile(inputMediaFilePath,
		                "My Video Motion Detection Input Asset",
		                AssetCreationOptions.None);
		
		            // Declare a new job.
		            IJob job = _context.Jobs.Create("My Video Motion Detection Job");
		
		            // Get a reference to Azure Media Motion Detector.
		            string MediaProcessorName = "Azure Media Motion Detector";
		
		            var processor = GetLatestMediaProcessorByName(MediaProcessorName);
		
		            // Read configuration from the specified file.
		            string configuration = File.ReadAllText(configurationFile);
		
		            // Create a task with the encoding details, using a string preset.
		            ITask task = job.Tasks.AddNew("My Video Motion Detection Task",
		                processor,
		                configuration,
		                TaskOptions.None);
		
		            // Specify the input asset.
		            task.InputAssets.Add(asset);
		
		            // Add an output asset to contain the results of the job.
		            task.OutputAssets.AddNew("My Video Motion Detectoion Output Asset", AssetCreationOptions.None);
		
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
[Azure 媒体服务动作检测器博客](https://azure.microsoft.com/blog/motion-detector-update/)

[Azure Media Services Analytics Overview（Azure 媒体服务分析概述）](/documentation/articles/media-services-analytics-overview/)

[Azure Media Analytics demos（Azure 媒体分析演示）](http://azuremedialabs.azurewebsites.net/demos/Analytics.html)

<!---HONumber=Mooncake_1114_2016-->