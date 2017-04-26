<properties
    pageTitle="使用 Azure Media Encoder Standard 自动生成比特率阶梯 | Azure"
    description="本主题介绍如何使用 Media Encoder Standard (MES) 根据输入分辨率和比特率自动生成比特率阶梯。 不会超过输入分辨率和比特率。 例如，如果输入在 3Mbps 时为 720p，则输出最多会保持 720p，并且会以低于 3Mbps 的速率开始。"
    services="media-services"
    documentationcenter=""
    author="juliako"
    manager="erikre"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="63ed95da-1b82-44b0-b8ff-eebd535bc5c7"
    ms.service="media-services"
    ms.workload="media"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="11/29/2016"
    wacn.date="04/24/2017"
    ms.author="juliako"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="f74a7e5af7173c164623d21e3c8aef8872569bd7"
    ms.lasthandoff="04/14/2017" />

#  <a name="use-azure-media-encoder-standard-to-auto-generate-a-bitrate-ladder"></a>使用 Azure Media Encoder Standard 自动生成比特率阶梯

## <a name="overview"></a>概述

本主题介绍如何使用 Media Encoder Standard (MES) 根据输入分辨率和比特率自动生成比特率阶梯（比特率-分辨率对）。 自动生成的预设不会超过输入分辨率和比特率。 例如，如果输入在 3Mbps 时为 720p，则输出最多会保持 720p，并且会以低于 3Mbps 的速率开始。

若要使用此功能，需在创建编码任务时指定“自适应流式处理”预设。 使用“自适应流式处理”预设时，MES 编码器会以智能方式为比特率阶梯设置一个上限。 但是，用户无法控制编码成本，因为是由服务确定要使用的层数和分辨率。 可以在本主题[末尾](#output)看到 MES 生成的输出层的示例，是使用“自适应流式处理”预设进行编码得来的。

## <a id="encoding_with_dotnet"></a>使用媒体服务 .NET SDK 进行编码

以下代码示例使用媒体服务 .NET SDK 执行下列任务：

- 创建编码作业。
- 获取对媒体编码器标准版编码器的引用。
- 向作业添加一个编码任务，指定使用“自适应流式处理”预设。 
- 创建将包含所编码资产的输出资产。
- 添加事件处理程序以检查作业进度。
- 提交作业。

        using System;
        using System.Configuration;
        using System.IO;
        using System.Linq;
        using Microsoft.WindowsAzure.MediaServices.Client;
        using System.Threading;

        namespace AdaptiveStreamingMESPresest
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

## <a id="output"></a>输出

此部分显示 MES 生成的输出层的三个示例，是使用“自适应流式处理”预设进行编码得来的。 

### <a name="example-1"></a>示例 1
高度为“1080”，帧速率为“29.970”的源生成 6 个视频层：

|层|高度|宽度|比特率 (kbps)|
|---|---|---|---|
|1|1080|1920|6780|
|2|720|1280|3520|
|3|540|960|2210|
|4|360|640|1150|
|5|270|480|720|
|6|180|320|380|

### <a name="example-2"></a>示例 2
高度为“720”，帧速率为“23.970”的源生成 5 个视频层：

|层|高度|宽度|比特率 (kbps)|
|---|---|---|---|
|1|720|1280|2940|
|2|540|960|1850|
|3|360|640|960|
|4|270|480|600|
|5|180|320|320|

### <a name="example-3"></a>示例 3
高度为“360”，帧速率为“29.970”的源生成 3 个视频层：

|层|高度|宽度|比特率 (kbps)|
|---|---|---|---|
|1|360|640|700|
|2|270|480|440|
|3|180|320|230|


## <a name="see-also"></a>另请参阅
[媒体服务编码概述](/documentation/articles/media-services-encode-asset/)