<properties
    pageTitle="使用 Azure 媒体分析检测面部和情绪 | Azure"
    description="本主题演示如何使用 Azure 媒体分析检测人脸和情感。"
    services="media-services"
    documentationcenter=""
    author="juliako"
    manager="erikre"
    editor="" />
<tags
    ms.assetid="5ca4692c-23f1-451d-9d82-cbc8bf0fd707"
    ms.service="media-services"
    ms.workload="media"
    ms.tgt_pltfrm="na"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.date="02/09/2017"
    wacn.date="03/10/2017"
    ms.author="milanga;juliako;" />  


# 使用 Azure 媒体分析检测面部和情绪
## 概述
**Azure 媒体面部检测器**媒体处理器 \(MP\) 可让你通过面部表情来统计、跟踪动作，甚至计量受众的参与和反应。此服务包含两项功能：

- **面部检测**

	面部检测能够找出并跟踪视频中的人脸。可以同时跟踪多个面部，随着对象移动持续进行跟踪，并将时间和位置的元数据以 JSON 文件的格式返回。跟踪期间，该服务将在人员于屏幕上四处移动时，尝试为他们的面部赋予相同的 ID，即使他们被挡住或暂时离帧。

	>[AZURE.NOTE]此服务并不执行面部识别。面部离帧或被挡住太久的人员，将在回来时赋予新的 ID。

- **情绪检测**
	
	情绪检测是面部检测媒体处理器的可选组件，它根据检测到的面部返回多个情绪属性的分析，包括快乐、悲伤、恐惧、愤怒等等。

**Azure 媒体面部检测器** MP 目前以预览版提供。

本主题提供有关 **Azure Media Face Detector** 的详细信息，并说明如何将其与用于 .NET 的媒体服务 SDK 搭配使用。

## 面部检测器输入文件
视频文件。目前支持以下格式：MP4、MOV 和 WMV。

## 面部检测器输出文件
面部检测器和跟踪 API 可提供高精确度的面部位置检测和跟踪功能，并在单个视频中检测到最多 64 个人脸。正面的面部可提供最佳效果，而侧面的面部和较小的面部（小于或等于 24x24 像素）可能就无法获得相同的精确度。

已检测到并已跟踪的面部将在坐标（左侧、顶部、宽度和高度）中返回，其中会在以像素为单位的图像中指明面部的位置，以及表示正在跟踪该人员的面部 ID 编号。在正面面部长时间于帧中消失或重叠的情况下，面部 ID 编号很容易重置，导致某些人员被分配多个 ID。

### <a id="output_elements"></a>输出 JSON 文件中的元素
对于面部检测和跟踪操作，输出结果以 JSON 格式包含给定文件中面部的元数据。

面部检测和跟踪 JSON 包括以下属性：

元素|说明
---|---
版本|这是指视频 API 的版本。
时间刻度|视频每秒的“刻度”数。
Offset|这是时间戳的时间偏移量。在版本 1.0 的视频 API 中，此属性始终为 0。在我们将来支持的方案中，此值可能会更改。
Framerate|视频的每秒帧数。
Fragments|元数据划分成称为“片段”的不同段。每个片段包含开始时间、持续时间、间隔数字和事件。
开始|第一个事件的开始时间（以“刻度”为单位）。
持续时间|片段的长度（以“刻度”为单位）。
时间间隔|片段中每个事件条目的间隔（以“刻度”为单位）。
事件|每个事件包含在该持续时间内检测到并跟踪的面部。它是包含事件数组的数组。外部数组代表一个时间间隔。内部数组包含在该时间点发生的 0 个或多个事件。空括号代表没有检测到面部。
ID| 正在跟踪的面部的 ID。如果某个面部后来未被检测到，此编号可能会意外更改。给定人员在整个视频中应该拥有相同的 ID，但由于检测算法的限制（例如受到阻挡等情况），我们无法保证这一点。
X, Y|规范化 0.0 到 1.0 比例中面部边框左上角的 X 和 Y 坐标。<br/>-X 和 Y 坐标总是相对于横向方向，因此如果视频是纵向（或使用 iOS 时上下颠倒），便需要相应地变换坐标。
Width, Height|规范化 0.0 到 1.0 比例中面部边框的宽度和高度。
facesDetected|位于 JSON 结果的末尾，汇总在生成视频期间算法所检测到的面部数。由于 ID 可能在面部无法检测时（例如面部离开屏幕、转向别处）意外重置，此数字并不一定与视频中的实际面部数相同。

面部检测器使用分片（元数据可以分解为基于时间的区块，你可以只下载需要的部分）和分段（可以在事件数过于庞大的情况下对事件进行分解）技术。一些简单的计算可帮助你转换数据。例如，如果事件从 6300（刻度）开始，其时间刻度为 2997（刻度/秒），帧速率为 29.97（帧/秒），那么：

* 开始时间/时间刻度 = 2.1 秒
* 秒数 x 帧速率 = 63 帧

## 面部检测输入和输出示例
### 输入视频
[输入视频](http://ampdemo.azureedge.net/azuremediaplayer.html?url=https%3A%2F%2Freferencestream-samplestream.streaming.mediaservices.windows.net%2Fc8834d9f-0b49-4b38-bcaf-ece2746f1972%2FMicrosoft%20Convergence%202015%20%20Keynote%20Highlights.ism%2Fmanifest&amp;autoplay=false)

### 任务配置（预设）
在使用 **Azure 媒体面部检测器**创建任务时，必须指定配置预设。以下配置预设仅适用于面部检测。

    {
      "version":"1.0"
      "options":{
          "TrackingMode": "Faster"
      }
    }

#### 属性说明
| 属性名称 | 说明 |
| --- | --- |
| Mode |<p>速度更快：处理速度更快，但准确性较低（默认设置）。</p><p>质量：跟踪准确性更好，但所需时间更长。</p> |


### JSON 输出
下面是 JSON 输出被截断的示例。

	{
	"version": 1,
	"timescale": 30000,
	"offset": 0,
	"framerate": 29.97,
	"width": 1280,
	"height": 720,
	"fragments": [
	    {
	    "start": 0,
	    "duration": 60060
	    },
	    {
	    "start": 60060,
	    "duration": 60060,
	    "interval": 1001,
	    "events": [
	        [
	        {
	            "id": 0,
	            "x": 0.519531,
	            "y": 0.180556,
	            "width": 0.0867188,
	            "height": 0.154167
	        }
	        ],
	        [
	        {
	            "id": 0,
	            "x": 0.517969,
	            "y": 0.181944,
	            "width": 0.0867188,
	            "height": 0.154167
	        }
	        ],
	        [
	        {
	            "id": 0,
	            "x": 0.517187,
	            "y": 0.183333,
	            "width": 0.0851562,
	            "height": 0.151389
	        }
	        ],

		. . . 

## 情绪检测输入和输出示例
### 输入视频
[输入视频](http://ampdemo.azureedge.net/azuremediaplayer.html?url=https%3A%2F%2Freferencestream-samplestream.streaming.mediaservices.windows.net%2Fc8834d9f-0b49-4b38-bcaf-ece2746f1972%2FMicrosoft%20Convergence%202015%20%20Keynote%20Highlights.ism%2Fmanifest&amp;autoplay=false)

### 任务配置（预设）
在使用 **Azure 媒体面部检测器**创建任务时，必须指定配置预设。以下配置预设指定基于情绪检测创建 JSON。
 	
	{
	  "version": "1.0",
	  "options": {
	    "aggregateEmotionWindowMs": "987",
	    "mode": "aggregateEmotion",
	    "aggregateEmotionIntervalMs": "342"
	  }
	}


#### 属性说明
| 属性名称 | 说明 |
| --- | --- |
| Mode |<p>人脸：仅人脸检测。</p><p>PerFaceEmotion：针对每个人脸检测独立返回表情。</p><p>AggregateEmotion：针对帧中的所有人脸返回平均表情值。</p> |
| AggregateEmotionWindowMs |在已选择 AggregateEmotion 模式时使用。指定用于生成每个聚合结果的视频的长度，以毫秒为单位。 |
| AggregateEmotionIntervalMs |在已选择 AggregateEmotion 模式时使用。指定生成聚合结果的频率。 |

####聚合默认值

下面是聚合窗口和间隔设置的建议值。AggregateEmotionWindowMs 应该超过 AggregateEmotionIntervalMs。

|| 默认值 | 最小值 | 最大值 |
|--- | --- | --- | --- |
| AggregateEmotionWindowMs |0\.5 |2 |0\.25|
| AggregateEmotionIntervalMs |0\.5 |1 |0\.25|

###JSON 输出

聚合情绪的 JSON 输出（已截断）：
 
	
	{
	 "version": 1,
	 "timescale": 30000,
	 "offset": 0,
	 "framerate": 29.97,
	 "width": 1280,
	 "height": 720,
	 "fragments": [
	   {
	     "start": 0,
	     "duration": 60060,
	     "interval": 15015,
	     "events": [
	       [
	         {
	           "windowFaceDistribution": {
	             "neutral": 0,
	             "happiness": 0,
	             "surprise": 0,
	             "sadness": 0,
	             "anger": 0,
	             "disgust": 0,
	             "fear": 0,
	             "contempt": 0
	           },
	           "windowMeanScores": {
	             "neutral": 0,
	             "happiness": 0,
	             "surprise": 0,
	             "sadness": 0,
	             "anger": 0,
	             "disgust": 0,
	             "fear": 0,
	             "contempt": 0
	           }
	         }
	       ],
	       [
	         {
	           "windowFaceDistribution": {
	             "neutral": 0,
	             "happiness": 0,
	             "surprise": 0,
	             "sadness": 0,
	             "anger": 0,
	             "disgust": 0,
	             "fear": 0,
	             "contempt": 0
	           },
	           "windowMeanScores": {
	             "neutral": 0,
	             "happiness": 0,
	             "surprise": 0,
	             "sadness": 0,
	             "anger": 0,
	             "disgust": 0,
	             "fear": 0,
	             "contempt": 0
	           }
	         }
	       ],
	       [
	         {
	           "windowFaceDistribution": {
	             "neutral": 0,
	             "happiness": 0,
	             "surprise": 0,
	             "sadness": 0,
	             "anger": 0,
	             "disgust": 0,
	             "fear": 0,
	             "contempt": 0
	           },
	           "windowMeanScores": {
	             "neutral": 0,
	             "happiness": 0,
	             "surprise": 0,
	             "sadness": 0,
	             "anger": 0,
	             "disgust": 0,
	             "fear": 0,
	             "contempt": 0
	           }
	         }
	       ],
	       [
	         {
	           "windowFaceDistribution": {
	             "neutral": 0,
	             "happiness": 0,
	             "surprise": 0,
	             "sadness": 0,
	             "anger": 0,
	             "disgust": 0,
	             "fear": 0,
	             "contempt": 0
	           },
	           "windowMeanScores": {
	             "neutral": 0,
	             "happiness": 0,
	             "surprise": 0,
	             "sadness": 0,
	             "anger": 0,
	             "disgust": 0,
	             "fear": 0,
	             "contempt": 0
	           }
	         }
	       ]
	     ]
	   },
	   {
	     "start": 60060,
	     "duration": 60060,
	     "interval": 15015,
	     "events": [
	       [
	         {
	           "windowFaceDistribution": {
	             "neutral": 1,
	             "happiness": 0,
	             "surprise": 0,
	             "sadness": 0,
	             "anger": 0,
	             "disgust": 0,
	             "fear": 0,
	             "contempt": 0
	           },
	           "windowMeanScores": {
	             "neutral": 0.688541,
	             "happiness": 0.0586323,
	             "surprise": 0.227184,
	             "sadness": 0.00945675,
	             "anger": 0.00592107,
	             "disgust": 0.00154993,
	             "fear": 0.00450447,
	             "contempt": 0.0042109
	           }
	         }
	       ],
	       [
	         {
	           "windowFaceDistribution": {
	             "neutral": 1,
	             "happiness": 0,
	             "surprise": 0,
	             "sadness": 0,
	             "anger": 0,
	             "disgust": 0,
	             "fear": 0,


## 限制
* 支持的输入视频格式包括 MP4、MOV 和 WMV。
* 可检测的面部大小范围为 24x24 到 2048x2048 像素。无法检测此范围以外的面部。
* 对于每个视频，返回的面部数上限为 64。
* 某些面部可能因技术难题而无法检测，例如非常大的面部角度（头部姿势），以及较大的阻挡物。正面和接近正面的面部可提供最佳效果。

## 代码示例
以下程序演示如何：

1. 创建资产并将媒体文件上传到资产。
2. 使用人脸检测任务创建一个作业，所根据的配置文件包含以下 json 预设。
   
        {
            "version": "1.0"
        }
3. 下载输出 JSON 文件。
   
        using System;
		using System.Configuration;
		using System.IO;
		using System.Linq;
		using Microsoft.WindowsAzure.MediaServices.Client;
		using System.Threading;
		using System.Threading.Tasks;
		
		namespace FaceDetection
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
		            // Used the cached credentials to create CloudMediaContext.
		            _context = new CloudMediaContext(_apiServer, _cachedCredentials);
		
		            // Run the FaceDetection job.
		            var asset = RunFaceDetectionJob(@"C:\supportFiles\FaceDetection\BigBuckBunny.mp4",
		                                        @"C:\supportFiles\FaceDetection\config.json");
		
		            // Download the job output asset.
		            DownloadAsset(asset, @"C:\supportFiles\FaceDetection\Output");
		        }
		
		        static IAsset RunFaceDetectionJob(string inputMediaFilePath, string configurationFile)
		        {
		            // Create an asset and upload the input media file to storage.
		            IAsset asset = CreateAssetAndUploadSingleFile(inputMediaFilePath,
		                "My Face Detection Input Asset",
		                AssetCreationOptions.None);
		
		            // Declare a new job.
		            IJob job = _context.Jobs.Create("My Face Detection Job");
		
		            // Get a reference to Azure Media Face Detector.
		            string MediaProcessorName = "Azure Media Face Detector";
		
		            var processor = GetLatestMediaProcessorByName(MediaProcessorName);
		
		            // Read configuration from the specified file.
		            string configuration = File.ReadAllText(configurationFile);
		
		            // Create a task with the encoding details, using a string preset.
		            ITask task = job.Tasks.AddNew("My Face Detection Task",
		                processor,
		                configuration,
		                TaskOptions.None);
		
		            // Specify the input asset.
		            task.InputAssets.Add(asset);
		
		            // Add an output asset to contain the results of the job.
		            task.OutputAssets.AddNew("My Face Detectoion Output Asset", AssetCreationOptions.None);
		
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




## 相关链接
[Azure Media Services Analytics Overview（Azure 媒体服务分析概述）](/documentation/articles/media-services-analytics-overview/)

[Azure Media Analytics demos（Azure 媒体分析演示）](http://azuremedialabs.azurewebsites.net/demos/Analytics.html)

<!---HONumber=Mooncake_0306_2017-->
<!--Update_Description: update "聚合默认值" table-->