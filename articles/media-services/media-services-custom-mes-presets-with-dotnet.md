<properties
    pageTitle="自定义 Media Encoder Standard 预设 | Azure"
    description="本主题说明如何通过自定义 Media Encoder Standard 任务预设执行高级编码。本主题说明如何使用媒体服务 .NET SDK 创建编码任务和作业。此外，还说明如何向编码作业提供自定义预设。"
    services="media-services"
    documentationcenter=""
    author="juliako"
    manager="erikre"
    editor="" />
<tags
    ms.assetid="ec95392f-d34a-4c22-a6df-5274eaac445b"
    ms.service="media-services"
    ms.workload="media"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="11/29/2016"
    wacn.date="01/13/2017"
    ms.author="juliako" />  


# 自定义 Media Encoder Standard 预设

##概述

本主题演示如何通过使用自定义预设的 Media Encoder Standard \(MES\) 执行高级编码。本主题使用 .NET 创建编码任务和执行此任务的作业。

本主题介绍如何使用 [H264 多比特率 720p](/documentation/articles/media-services-mes-preset-H264-Multiple-Bitrate-720p/) 预设并减少层数，从而自定义预设。[自定义 Media Encoder Standard 预设](/documentation/articles/media-services-advanced-encoding-with-mes/)主题演示了可用于执行高级编码任务的自定义预设。

## <a id="customizing_presets"></a> 自定义 MES 预设

### 原始预设

将在 [H264 多比特率 720p](/documentation/articles/media-services-mes-preset-H264-Multiple-Bitrate-720p/) 主题中定义的 JSON 保存到具有 .json 扩展名的文件。例如，**CustomPreset\_JSON.json**。

### 自定义的预设

打开 **CustomPreset\_JSON.json** 文件，删除 **H264Layers** 中的前三层，使文件如下所示。

	
	{  
	  "Version": 1.0,  
	  "Codecs": [  
	    {  
	      "KeyFrameInterval": "00:00:02",  
	      "H264Layers": [  
	        {  
	          "Profile": "Auto",  
	          "Level": "auto",  
	          "Bitrate": 1000,  
	          "MaxBitrate": 1000,  
	          "BufferWindow": "00:00:05",  
	          "Width": 640,  
	          "Height": 360,  
	          "BFrames": 3,  
	          "ReferenceFrames": 3,  
	          "AdaptiveBFrame": true,  
	          "Type": "H264Layer",  
	          "FrameRate": "0/1"  
	        },  
	        {  
	          "Profile": "Auto",  
	          "Level": "auto",  
	          "Bitrate": 650,  
	          "MaxBitrate": 650,  
	          "BufferWindow": "00:00:05",  
	          "Width": 640,  
	          "Height": 360,  
	          "BFrames": 3,  
	          "ReferenceFrames": 3,  
	          "AdaptiveBFrame": true,  
	          "Type": "H264Layer",  
	          "FrameRate": "0/1"  
	        },  
	        {  
	          "Profile": "Auto",  
	          "Level": "auto",  
	          "Bitrate": 400,  
	          "MaxBitrate": 400,  
	          "BufferWindow": "00:00:05",  
	          "Width": 320,  
	          "Height": 180,  
	          "BFrames": 3,  
	          "ReferenceFrames": 3,  
	          "AdaptiveBFrame": true,  
	          "Type": "H264Layer",  
	          "FrameRate": "0/1"  
	        }  
	      ],  
	      "Type": "H264Video"  
	    },  
	    {  
	      "Profile": "AACLC",  
	      "Channels": 2,  
	      "SamplingRate": 48000,  
	      "Bitrate": 128,  
	      "Type": "AACAudio"  
	    }  
	  ],  
	  "Outputs": [  
	    {  
	      "FileName": "{Basename}_{Width}x{Height}_{VideoBitrate}.mp4",  
	      "Format": {  
	        "Type": "MP4Format"  
	      }  
	    }  
	  ]  
	}  
	

## <a id="encoding_with_dotnet"></a>使用媒体服务 .NET SDK 进行编码

以下代码示例使用媒体服务 .NET SDK 执行下列任务：

- 创建编码作业。
- 获取对 Media Encoder Standard 编码器的引用。
- 加载前面部分中创建的自定义 JSON 预设。
  
		// Load the JSON from the local file.
		string configuration = File.ReadAllText(fileName);  

- 将编码任务添加到作业。
- 指定要编码的输入资产。
- 创建将包含所编码资产的输出资产。
- 添加事件处理程序以检查作业进度。
- 提交作业。
   
        using System;
        using System.Collections.Generic;
        using System.Configuration;
        using System.IO;
        using System.Linq;
        using System.Net;
        using System.Security.Cryptography;
        using System.Text;
        using System.Threading.Tasks;
        using Microsoft.WindowsAzure.MediaServices.Client;
        using Newtonsoft.Json.Linq;
        using System.Threading;
        using Microsoft.WindowsAzure.MediaServices.Client.ContentKeyAuthorization;
        using Microsoft.WindowsAzure.MediaServices.Client.DynamicEncryption;
        using System.Web;
        using System.Globalization;
  
        namespace CustomizeMESPresests
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
  
                private static readonly string _mediaFiles =
                    Path.GetFullPath(@"../..\Media");
  
                private static readonly string _singleMP4File =
                    Path.Combine(_mediaFiles, @"BigBuckBunny.mp4");
  
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

  
                    // Get an uploaded asset.
                    var asset = _context.Assets.FirstOrDefault();
  
                    // Encode and generate the output using custom presets.
                    EncodeToAdaptiveBitrateMP4Set(asset);
  
                    Console.ReadLine();
                }
  
                static public IAsset EncodeToAdaptiveBitrateMP4Set(IAsset asset)
                {
                    // Declare a new job.
                    IJob job = _context.Jobs.Create("Media Encoder Standard Job");
                    // Get a media processor reference, and pass to it the name of the 
                    // processor to use for the specific task.
                    IMediaProcessor processor = GetLatestMediaProcessorByName("Media Encoder Standard");

                    // Load the XML (or JSON) from the local file.
                    string configuration = File.ReadAllText("CustomPreset_JSON.json");

                    // Create a task
                    ITask task = job.Tasks.AddNew("Media Encoder Standard encoding task",
                        processor,
                        configuration,
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

            }
        }




## 另请参阅
[媒体服务编码概述](/documentation/articles/media-services-encode-asset/)

<!---HONumber=Mooncake_0109_2017-->
<!--Update_Description: main content update, the subject is updated from "advanced encoding" to "custom preset"-->