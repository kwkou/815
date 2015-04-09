<properties linkid="develop-media-services-how-to-guides-encrypt-assets" urlDisplayName="Encrypt Assets in Media Services" pageTitle="How to Encrypt Assets in Media Services - Azure" metaKeywords="" description="Learn how to use Microsoft PlayReady Protection to encrypt an asset in Media Services. Code samples are written in C# and use the Media Services SDK for .NET. Code samples are written in C# and use the Media Services SDK for .NET." metaCanonical="" services="media-services" documentationCenter="" title="How to: Protect an Asset with PlayReady Protection" authors="migree" solutions="" manager="" editor="" />

如何：使用 PlayReady 保护功能保护资产
=====================================

本文是介绍 Azure Media Services 编程的系列主题中的一篇。前一个主题是[如何：检查作业进度](http://go.microsoft.com/fwlink/?LinkID=301737&clcid=0x409)。

在 Azure Media Services 中，你可以提交一个与 Microsoft PlayReady 保护集成的作业来加密资产。本部分中的代码使用输入文件夹中的多个流文件，在创建一个任务后，将使用 PlayReady 保护将这些文件加密。

以下示例演示如何创建一个简单的作业来提供 PlayReady 保护。

1.  检索配置数据。你可以从 [Azure 媒体加密器的任务预设](http://msdn.microsoft.com/zh-cn/library/hh973610.aspx)主题中获取示例配置文件。
2.  上载 MP4 输入文件
3.  将 MP4 文件转换为平滑流式处理资产
4.  使用 PlayReady 加密资产
5.  提交作业

以下代码示例演示如何执行这些步骤：

``` {}
private static IJob CreatePlayReadyProtectionJob(string inputMediaFilePath, string primaryFilePath, string configFilePath)
{
// Create a storage-encrypted asset and upload the MP4. 
IAsset asset = CreateAssetAndUploadSingleFile(AssetCreationOptions.StorageEncrypted, inputMediaFilePath);

// Declare a new job to contain the tasks.
IJob job = _context.Jobs.Create("My PlayReady Protection job");

// Set up the first task. 

// Read the task configuration data into a string. 
string configMp4ToSmooth = File.ReadAllText(Path.GetFullPath(configFilePath + @"\MediaPackager_MP4ToSmooth.xml"));

// Get a media processor instance
IMediaProcessor processor = GetLatestMediaProcessorByName("Azure Media Packager");

// Create a task with the conversion details, using the configuration data 
ITask task = job.Tasks.AddNew("My Mp4 to Smooth Task",
processor,
configMp4ToSmooth,
_clearConfig);

// Specify the input asset to be converted.
task.InputAssets.Add(asset);

// Add an output asset to contain the results of the job.We do not need 
// to persist the output asset to storage, so set the shouldPersistOutputOnCompletion
// parameter to false. 
task.OutputAssets.AddNew("Streaming output asset", AssetCreationOptions.None);
IAsset smoothOutputAsset = task.OutputAssets[0];

// Set up the encryption task. 

// Read the configuration data into a string. 
string configPlayReady = File.ReadAllText(Path.GetFullPath(configFilePath + @"\MediaEncryptor_PlayReadyProtection.xml"));

// Get a media processor instance
IMediaProcessor playreadyProcessor = GetLatestMediaProcessorByName("Azure Media Encryptor");

// Create a second task, specifying a task name, the media processor, and configuration
ITask playreadyTask = job.Tasks.AddNew("My PlayReady Task",
playreadyProcessor,
configPlayReady,
_protectedConfig);

// Add the input asset, which is the smooth streaming output asset from the first task. 
playreadyTask.InputAssets.Add(smoothOutputAsset);

// Add an output asset to contain the results of the job.Persist the output by setting 
// the shouldPersistOutputOnCompletion param to true.
playreadyTask.OutputAssets.AddNew("PlayReady protected output asset",
AssetCreationOptions.None);

// Use the following event handler to check job progress. 
job.StateChanged += new
EventHandler<JobStateChangedEventArgs>(StateChanged);

// Launch the job.
job.Submit();

// Optionally log job details.This displays basic job details
// to the console and saves them to a JobDetails-{JobId}.txt file 
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
```

有关 PlayReady 保护的详细信息，请参阅：

-   [使用 Microsoft PlayReady 保护资产](http://msdn.microsoft.com/zh-cn/library/dn189154.aspx)
-   [Microsoft PlayReady](http://www.microsoft.com/PlayReady/)

后续步骤
--------

了解如何使用 Media Services 保护资产后，请转到[如何管理资产](http://go.microsoft.com/fwlink/?LinkID=301943&clcid=0x409)主题。

