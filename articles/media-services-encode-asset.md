<properties linkid="develop-media-services-how-to-guides-encode-an-asset" urlDisplayName="How to Encode an Asset" pageTitle="How to Encode an Asset for Media Services - Azure" metaKeywords="" description="Learn how to use the Azure Media Encoder to encode media content on Media Services. Code samples are written in C# and use the Media Services SDK for .NET." metaCanonical="" services="media-services" documentationCenter="" title="How to: Encode an Asset" authors="migree" solutions="" manager="" editor="" />

如何：对资产进行编码
====================

本文是介绍 Azure Media Services 编程的系列主题中的一篇。前一个主题是[如何：获取媒体处理器](http://go.microsoft.com/fwlink/?LinkID=301732&ampclcid=0x409)。

对于服务器上的媒体内容，你可以通过 Azure 媒体编码器使用许多的媒体编码和格式对这些内容进行编码。你也可以使用 Media Services 合作伙伴提供的编码器；可通过 [Azure Marketplace](https://datamarket.azure.com/) 获取第三方编码器。可以使用[编码器预设](http://msdn.microsoft.com/zh-cn/library/hh973610.aspx)字符串或配置文件指定编码任务的详细信息。

编码为 MP4
----------

以下方法将上载单个资产，并创建一个作业以使用“H264 Broadband 720p”预设将该资产编码为 MP4，该预设将创建编码为 H264、分辨率为 720p 的单个 MP4 媒体。

``` {}
    static IJob CreateEncodingJob(string inputMediaFilePath, string outputFolder)
    {
    //Create an encrypted asset and upload to storage.
        IAsset asset = CreateAssetAndUploadSingleFile(AssetCreationOptions.StorageEncrypted, 
            inputMediaFilePath);

        // Declare a new job.

    IJob job = _context.Jobs.Create("My encoding job");
    
        // Get a reference to the Azure Media Encoder
        IMediaProcessor processor = GetLatestMediaProcessorByName("Azure Media Encoder");
    
        // Create a task with the encoding details, using a string preset.
    ITask task = job.Tasks.AddNew("My encoding task",
    processor,
            "H264 Broadband 720p",
    _protectedConfig);
    
        // Specify the input asset to be encoded.
    task.InputAssets.Add(asset);
    
        // Add an output asset to contain the results of the job. 
    // This output is specified as AssetCreationOptions.None, which 
    // means the output asset is in the clear (unencrypted). 
    task.OutputAssets.AddNew("Output asset", AssetCreationOptions.None);
    
        // Use the following event handler to check job progress.  
    job.StateChanged += new EventHandler&ltJobStateChangedEventArgs>(StateChanged);
    
        // Launch the job.
    job.Submit();
    
        // Optionally log job details.This displays basic job details
    // to the console and saves them to a JobDetails-JobId.txt file 
    // in your output folder.
    LogJobDetails(job.Id);
    
        // Check job execution and wait for job to finish. 
    Task progressJobTask = job.GetExecutionProgressTask(CancellationToken.None);
    progressJobTask.Wait();
    
        // If job state is Error, the event handling 
    // method for job progress should log errors.Here we check 
    // for error state and exit if needed.
    if (job.State == JobState.Error)
        {
            Console.WriteLine("\nExiting method due to job error.");
    return job;
        }
    
        // Perform other tasks.For example, access the assets that are the output of a job, 
    // either by creating URLs to the asset on the server, or by downloading. 
    return job;
    }

    private static void StateChanged(object sender, JobStateChangedEventArgs e)
    {
        Console.WriteLine("Job state changed event:");
        Console.WriteLine("  Previous state:" + e.PreviousState);
        Console.WriteLine("  Current state:" + e.CurrentState);
        switch (e.CurrentState)
        {
    case JobState.Finished:
    Console.WriteLine();
    Console.WriteLine("Job is finished.Please wait while local tasks or downloads complete...");
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
    LogJobStop(job.Id);
    break;
    default:
                break;
        }
    }
```

编码为平滑流式处理
------------------

如果你想要将视频编码为平滑流式处理，可以使用两个选项：

-   直接编码为平滑流式处理
-   编码为 MP4，然后转换为平滑流式处理

若要直接编码为平滑流式处理，请使用上面所示的代码，不过，需要在其中改用一个平滑流式处理编码器预设。有关编码器预设的完整列表，请参阅 [Azure 媒体编码器的任务预设字符串](http://msdn.microsoft.com/zh-cn/library/jj129582.aspx)。

若要将 MP4 转换为平滑流式处理，请使用 Azure 媒体包装器。Azure 媒体包装器不支持字符串预设，因此你必须在 XML 中指定配置选项。在 [Azure 媒体包装器的任务预设](http://msdn.microsoft.com/zh-cn/library/windowsazure/hh973635.aspx)中可以找到将 MP4 转换为平滑流式处理所需的 XML。将该 XML 复制并粘贴到项目中名为 MediaPackager\_MP4ToSmooth.xml 的文件。以下代码演示了如何将 MP4 资产转换为平滑流式处理。下面的方法采用现有某个资产并对其进行转换。

``` {}
private static IJob ConvertMP4toSmooth(IAsset assetToConvert, string configFilePath)
 {
    // Declare a new job to contain the tasks
IJob job = _context.Jobs.Create("Convert to Smooth Streaming job");
// Set up the first Task to convert from MP4 to Smooth Streaming. 
// Read in task configuration XML
string configMp4ToSmooth = File.ReadAllText(Path.GetFullPath(configFilePath + @"\MediaPackager_MP4ToSmooth.xml"));
// Get a media packager reference
IMediaProcessor processor = GetLatestMediaProcessorByName("Azure Media Packager");
// Create a task with the conversion details, using the configuration data
ITask task = job.Tasks.AddNew("My Mp4 to Smooth Task",
processor,
configMp4ToSmooth,
TaskOptions.None);
// Specify the input asset to be converted.
task.InputAssets.Add(assetToConvert);
// Add an output asset to contain the results of the job.
task.OutputAssets.AddNew("Streaming output asset", AssetCreationOptions.None);
// Use the following event handler to check job progress. 
    // The StateChange method is the same as the one in the previous sample
job.StateChanged += new EventHandler&ltJobStateChangedEventArgs>(StateChanged);
// Launch the job.
job.Submit();
// Check job execution and wait for job to finish. 
Task progressJobTask = job.GetExecutionProgressTask(CancellationToken.None);
progressJobTask.Wait();
// Get a refreshed job reference after waiting on a thread.
job = GetJob(job.Id);
// Check for errors
if (job.State == JobState.Error)
    {
Console.WriteLine("\nExiting method due to job error.");
    }
return job;
}
```

有关处理资产的详细信息，请参阅：

-   [使用 Media Services SDK for .NET 处理资产](http://msdn.microsoft.com/zh-cn/library/jj129580.aspx)
-   [使用 Media Services REST API 处理资产](http://msdn.microsoft.com/zh-cn/library/jj129574.aspx)

后续步骤
--------

了解如何创建一个对资产进行编码的作业后，请转到[如何使用 Media Services 检查作业进度](http://go.microsoft.com/fwlink/?LinkID=301737&ampclcid=0x409)主题。

