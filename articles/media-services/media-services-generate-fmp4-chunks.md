<properties
    pageTitle="创建生成 fMP4 区块的 Azure 媒体服务编码任务 | Azure"
    description="本主题介绍如何创建生成 fMP4 区块的编码任务。 将此任务用于 Media Encoder Standard 编码器时，输出资产会包含 fMP4 区块而非 ISO MP4 文件。"
    services="media-services"
    documentationcenter=""
    author="juliako"
    manager="erikre"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="b7029ac5-eadd-4a2f-8111-1fc460828981"
    ms.service="media-services"
    ms.workload="media"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="11/29/2016"
    wacn.date="04/24/2017"
    ms.author="juliako"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="3015c1722a657f7b59dac944677193a6ea538570"
    ms.lasthandoff="04/14/2017" />

#  <a name="create-an-encoding-task-that-generates-fmp4-chunks"></a>创建生成 fMP4 区块的编码任务

## <a name="overview"></a>概述

本主题介绍如何创建一个编码任务，以便生成分片的 MP4 (fMP4) 区块而非 ISO MP4 文件。 若要生成 fMP4 区块，请使用 **Media Encoder Standard** 编码器创建一个编码任务，并请指定 **AssetFormatOption.AdaptiveStreaming** 选项，如以下代码片段所示：  
    
    task.OutputAssets.AddNew(@"Output Asset containing fMP4 chunks", 
            options: AssetCreationOptions.None, 
            formatOption: AssetFormatOption.AdaptiveStreaming);


## <a id="encoding_with_dotnet"></a>使用媒体服务 .NET SDK 进行编码

以下代码示例使用媒体服务 .NET SDK 执行下列任务：

- 创建编码作业。
- 获取对 **Media Encoder Standard** 编码器的引用。
- 向作业添加一个编码任务，指定使用“自适应流式处理”预设。 
- 创建一个输出资产，其中将包含 fMP4 区块和 .ism 文件。
- 添加事件处理程序以检查作业进度。
- 提交作业。

        using System;
        using System.Configuration;
        using System.IO;
        using System.Linq;
        using Microsoft.WindowsAzure.MediaServices.Client;
        using System.Threading;

        namespace AdaptiveStreaming
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

            private static Uri _apiServer = null;
            
            // Field for service context.
            private static CloudMediaContext _context = null;
            private static MediaServicesCredentials _cachedCredentials = null;
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

                // Encode and generate the output using the "Adaptive Streaming" preset.
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

                // Create a task
                ITask task = job.Tasks.AddNew("Media Encoder Standard encoding task",
                processor,
                "Adaptive Streaming",
                TaskOptions.None);

                // Specify the input asset to be encoded.
                task.InputAssets.Add(asset);

                // Add an output asset to contain the results of the job. 

                // This output is specified as AssetCreationOptions.None, which 
                // means the output asset is not encrypted. 
                // It is also specified to use AssetFormatOption.AdaptiveStreaming, 
                // which means the output asset will contain fMP4 chunks.

                task.OutputAssets.AddNew(@"Output Asset containing fMP4 chunks",
                    options: AssetCreationOptions.None,
                    formatOption: AssetFormatOption.AdaptiveStreaming);

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



## <a name="see-also"></a>另请参阅
[媒体服务编码概述](/documentation/articles/media-services-encode-asset/)