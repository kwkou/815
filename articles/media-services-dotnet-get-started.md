<properties linkid="develop-media-services-tutorials-get-started" urlDisplayName="Get Started with Media Services" pageTitle="Get Started with Media Services - Azure" metaKeywords="Azure media services" description="An introduction to using Media Services with Azure." metaCanonical="" services="media-services" documentationCenter="" title="Get started with Media Services" authors="" solutions="" manager="" editor="" />

Media Services 入门
===================

本教程说明如何开始使用 Azure Media Services 进行开发。其中介绍了基本的 Media Services 工作流，以及进行 Media Services 开发需要用到的最常见编程对象和任务。完成本教程后，你就能够播放你上载、编码和下载的示例媒体文件。你还可以通过浏览找到编码的资产并在服务器上播放。

可从以下位置获取包含本教程中所述代码的 C\# Visual Studio 项目：[下载](http://go.microsoft.com/fwlink/?linkid=253275)。

本教程将指导你完成以下基本步骤：

-   [设置项目](#Step1)
-   [获取 Media Services 服务器上下文](#Step2)
-   [创建资产并将其关联文件上载到 Media Services](#Step3)
-   [为资产编码并下载输出资产](#Step4)

先决条件
--------

若要完成演练并基于 Azure Media Services SDK 进行开发，必须满足以下先决条件。

-   在新的或现有的 Azure 订阅中拥有一个 Media Services 帐户。有关详细信息，请参阅[如何创建 Media Services 帐户](http://go.microsoft.com/fwlink/?LinkId=256662)。
-   操作系统：Windows 7、Windows 2008 R2 或 Windows 8。
-   .NET Framework 4.5 或 .NET Framework 4。
-   Visual Studio 2012 或 Visual Studio 2010 SP1（专业版、高级专业版、旗舰版或学习版）。
-   安装 **Azure SDK for .NET.**、**Azure Media Services SDK for .NET** 和 **WCF Data Services 5.0 for OData V3 库**，并使用 [windowsazure.mediaservices Nuget](http://nuget.org/packages/windowsazure.mediaservices) 程序包添加对你的项目的引用。以下部分演示了如何安装以及添加这些引用。

**说明**

若要完成本教程，你需要一个 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 [Azure 免费试用](http://www.windowsazure.com/en-us/pricing/free-trial/?WT.mc_id=A8A8397B5)。

设置项目
--------

1.  在 Visual Studio 2012 或 Visual Studio 2010 SP1 中创建一个新的 C\# 控制台应用程序。输入“名称”、“位置”和“解决方案名称”，然后单击**“确定”**。

2.  添加对 System.Configuration 程序集的引用。

    若要使用“管理引用”对话框添加引用，**请执行以下操作**。右键单击 “解决方案资源管理器”中的“引用”节点并选择**“添加引用”**。在 “管理引用”对话框中，**选择相应的程序集**（在本例中为 System.Configuration）。

3.  （如果尚未这样做）使用 [windowsazure.mediaservices Nuget](http://nuget.org/packages/windowsazure.mediaservices) 程序包添加对 **Azure SDK for .NET.** (Microsoft.WindowsAzure.StorageClient.dll)、**Azure Media Services SDK for .NET** (Microsoft.WindowsAzure.MediaServices.Client.dll) 和 **WCF Data Services 5.0 for OData V3** (Microsoft.Data.OData.dll) 库的引用。

    若要使用 Nuget 添加引用，请执行以下操作。在 Visual Studio 主菜单中，选择“工具”-\>“库程序包管理器”-\>“程序包管理器控制台”。在控制台窗口中，键入 *Install-Package [程序包名称]*，然后按 Enter（在本例中，应使用以下命令：*Install-Package windowsazure.mediaservices*。）

4.  在 **app.config** 文件中添加一个 *appSettings* 部分，并设置 Azure Media Services 帐户名和帐户密钥的值。在设置帐户期间，你已获取 Media Services 帐户名和帐户密钥。在 Visual Studio 项目中，将这些值添加到 app.config 文件中每项设置的值属性。

    > [WACOM.NOTE] 在 Visual Studio 2012 中，已按默认添加 App.config 文件。在 Visual Studio 2010 中，必须手动添加应用程序配置文件。

    ``` {}
    <configuration>
        . . . 
        <appSettings>
        <add key="accountName" value="Add-Media-Services-Account-Name" />
        <add key="accountKey" value="Add-Media-Services-Account-Key" />
        </appSettings>
    </configuration>
     
    ```

5.  在本地计算机上创建一个新的文件夹并将其命名为 supportFiles（在本例中，supportFiles 在 MediaServicesGettingStarted 项目目录的下面。）本演练随附的[项目](http://go.microsoft.com/fwlink/?linkid=253275)包含 supportFiles 目录。你可以将此目录的内容复制到你的 supportFiles 文件夹中。

6.  使用以下代码覆盖位于 Program.cs 文件开头的现有 using 语句。

         using System;
        using System.Linq;
        using System.Configuration;
        using System.IO;
        using System.Text;
        using System.Threading;
        using System.Threading.Tasks;
        using System.Collections.Generic;
        using Microsoft.WindowsAzure;
        using Microsoft.WindowsAzure.MediaServices.Client;

7.  添加以下类级路径变量。**\_supportFiles** 路径应指向你在上一步创建的文件夹。

         // Base support files path.Update this field to point to the base path  
        // for the local support files folder that you create. 
        private static readonly string _supportFiles =
        Path.GetFullPath(@"../..\supportFiles");
            
        // Paths to support files (within the above base path).You can use 
        // the provided sample media files from the "supportFiles" folder, or 
        // provide paths to your own media files below to run these samples.
        private static readonly string _singleInputFilePath =
        Path.GetFullPath(_supportFiles + @"\multifile\interview2.wmv");
        private static readonly string _outputFilesFolder =
        Path.GetFullPath(_supportFiles + @"\outputfiles");

8.  添加以下类级变量，以检索身份验证和连接设置。这些设置是从 App.Config 文件中提取的，当你连接到 Media Services、进行身份验证以及获取用于访问服务器上下文的令牌时，需要用到这些设置。项目中的代码将引用这些变量来创建服务器上下文的实例。

         private static readonly string _accountKey = ConfigurationManager.AppSettings["accountKey"];
        private static readonly string _accountName = ConfigurationManager.AppSettings["accountName"];

9.  添加以下类级变量，用作对服务器上下文的静态引用。

         // Field for service context.
        private static CloudMediaContext _context = null;

获取 Media Services 上下文
--------------------------

Media Services 上下文对象包含 Media Services 编程时需要访问的所有基本对象和集合。该上下文包含对重要集合（包括作业、资产、文件、访问策略、定位器和其他对象）的引用。要完成大多数的 Media Services 编程任务，你必须获取服务器上下文。

在 Program.cs 文件中，添加以下代码作为 **Main** 方法中的第一个项。此代码使用 app.config 文件中你的 Media Services 帐户名和帐户密钥值来创建服务器上下文的实例。该实例将分配到你在类级别创建的 **\_context** 变量。

    // Get the service context.
    _context = new CloudMediaContext(_accountName, _accountKey);

创建资产并上载文件
------------------

本部分中的代码将执行以下操作：

1.  创建一个空资产
    创建资产时，你可以指定三个不同的用于加密资产的选项。

    -   **AssetCreationOptions.None**：不加密。如果你想要创建不加密的资产，则必须设置此选项。
    -   **AssetCreationOptions.CommonEncryptionProtected**：适用于通用加密保护 (CENC) 文件，例如，已进行 PlayReady 加密的一组文件。
    -   **AssetCreationOptions.StorageEncrypted**：存储加密。将明文输入文件上载到 Azure 存储空间之前对其进行加密。

        **说明**

        Media Services 提供磁盘存储加密，而不通过数字版权管理器 (DRM) 等途径加密。

 

2.  创建要与资产关联的 AssetFile 实例。
3.  创建用于定义权限以及资产访问持续时间的 AccessPolicy 实例。
4.  创建用于提供资产访问权限的 Locator 实例。
5.  将单个媒体文件上载到 Media Services。创建和上载过程也称为引入资产。

将以下方法添加到类。

``` {}
static private IAsset CreateEmptyAsset(string assetName, AssetCreationOptions assetCreationOptions)
{
var asset = _context.Assets.Create(assetName, assetCreationOptions);

Console.WriteLine("Asset name:" + asset.Name);
Console.WriteLine("Time created:" + asset.Created.Date.ToString());

return asset;
}

static public IAsset CreateAssetAndUploadSingleFile(AssetCreationOptions assetCreationOptions, string singleFilePath)
{
var assetName = "UploadSingleFile_" + DateTime.UtcNow.ToString();
var asset = CreateEmptyAsset(assetName, assetCreationOptions);

var fileName = Path.GetFileName(singleFilePath);

var assetFile = asset.AssetFiles.Create(fileName);

Console.WriteLine("Created assetFile {0}", assetFile.Name);

var accessPolicy = _context.AccessPolicies.Create(assetName, TimeSpan.FromDays(3),
AccessPermissions.Write | AccessPermissions.List);

var locator = _context.Locators.CreateLocator(LocatorType.Sas, asset, accessPolicy);

Console.WriteLine("Upload {0}", assetFile.Name);

assetFile.Upload(singleFilePath);
Console.WriteLine("Done uploading of {0} using Upload()", assetFile.Name);

locator.Delete();
accessPolicy.Delete();

return asset;
}
```

在 Main 方法中 **\_context = new CloudMediaContext(\_accountName, \_accountKey);** 行的后面添加对方法的调用。

    IAsset asset = CreateAssetAndUploadSingleFile(AssetCreationOptions.None, _singleInputFilePath)

在服务器上为资产编码并下载输出资产
----------------------------------

在 Media Services 中，可通过多种方式创建用于处理媒体内容的作业：编码、加密、执行格式转换，等等。一个 Media Services 作业始终包含一个或多个用于指定处理工作详细信息的任务。在本部分中，你将要创建一个基本的编码任务，然后运行使用 Azure 媒体编码器执行该任务的作业。该任务使用预设的字符串来指定要执行的编码类型。若要查看可用的预设编码值，请参阅 [Azure 媒体编码器的任务预设字符串](http://msdn.microsoft.com/zh-cn/library/windowsazure/jj129582.aspx)。Media Services 支持使用与 Microsoft Expression Encoder 相同的媒体文件输入和输出格式。有关支持的格式列表，请参阅 [Media Services 支持的文件类型](http://msdn.microsoft.com/zh-cn/library/windowsazure/hh973634.aspx)。

1.  将以下 **CreateEncodingJob** 方法定义添加到类。此方法演示如何完成执行某个编码作业而需要完成的多个任务：

    -   声明新作业。
    -   声明用于处理该作业的媒体处理器。媒体处理器是处理编码、加密、格式转换和其他相关处理作业的组件。有多种类型的媒体处理器可用（你可以使用 \_context.MediaProcessors 逐一查看所有这些处理器）。本演练稍后所示的 GetLatestMediaProcessorByName 方法将返回 Azure 媒体编码器处理器。
    -   声明新任务。每个作业有一个或多个任务。请注意，对于任务，可为其指定一个友好名称、媒体处理器实例、任务配置字符串和任务创建选项。配置字符串指定编码设置。本示例使用 **H264 Broadband 720p** 设置。此预设将生成单个 MP4 文件。有关此预设和其他预设的详细信息，请参阅 [Azure 媒体编码器的任务预设字符串](http://msdn.microsoft.com/library/windowsazure/jj129582.aspx)。
    -   将输入资产添加到任务。在本例中，输入资产是你在前一部分中创建的资产。
    -   将输出资产添加到任务。为输出资产指定一个友好名称、一个布尔值（指示是否在完成作业后将输出保存在服务器上）和一个 **AssetCreationOptions.None** 值（指示不加密要存储和传输的输出）。
    -   提交作业。
        提交作业是执行编码作业所要完成的最后一个步骤。

    该方法还演示了如何执行其他有用的任务（但这些任务是可选的），例如，跟踪作业进度，以及访问编码作业创建的资产。

    ``` {}
    static IJob CreateEncodingJob(IAsset asset, string inputMediaFilePath, string outputFolder)
    {
    // Declare a new job.
    IJob job = _context.Jobs.Create("My encoding job");
    // Get a media processor reference, and pass to it the name of the 
    // processor to use for the specific task.
    IMediaProcessor processor = GetLatestMediaProcessorByName("Azure Media Encoder");

    // Create a task with the encoding details, using a string preset.
    ITask task = job.Tasks.AddNew("My encoding task",
    processor,
    "H264 Broadband 720p",
    Microsoft.WindowsAzure.MediaServices.Client.TaskOptions.ProtectedConfiguration);

    // Specify the input asset to be encoded.
    task.InputAssets.Add(asset);
    // Add an output asset to contain the results of the job. 
    // This output is specified as AssetCreationOptions.None, which 
    // means the output asset is not encrypted. 
    task.OutputAssets.AddNew("Output asset",
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

        // **********
    // Optional code.Code after this point is not required for 
    // an encoding job, but shows how to access the assets that 
    // are the output of a job, either by creating URLs to the 
    // asset on the server, or by downloading. 
        // **********

    // Get an updated job reference.
    job = GetJob(job.Id);

    // If job state is Error the event handling 
    // method for job progress should log errors.Here we check 
    // for error state and exit if needed.
    if (job.State == JobState.Error)
        {
    Console.WriteLine("\nExiting method due to job error.");
    return job;
        }

    // Get a reference to the output asset from the job.
    IAsset outputAsset = job.OutputMediaAssets[0];
    IAccessPolicy policy = null;
    ILocator locator = null;

    // Declare an access policy for permissions on the asset. 
    // You can call an async or sync create method. 
    policy =
    _context.AccessPolicies.Create("My 30 days readonly policy",
    TimeSpan.FromDays(30),
    AccessPermissions.Read);

    // Create a SAS locator to enable direct access to the asset 
    // in blob storage.You can call a sync or async create method.  
    // You can set the optional startTime param as 5 minutes 
    // earlier than Now to compensate for differences in time  
    // between the client and server clocks. 

    locator = _context.Locators.CreateLocator(LocatorType.Sas, outputAsset,
    policy,
    DateTime.UtcNow.AddMinutes(-5));

    // Build a list of SAS URLs to each file in the asset. 
    List<String> sasUrlList = GetAssetSasUrlList(outputAsset, locator);

    // Write the URL list to a local file.You can use the saved 
    // SAS URLs to browse directly to the files in the asset.
    if (sasUrlList != null)
        {
    string outFilePath = Path.GetFullPath(outputFolder + @"\" + "FileSasUrlList.txt");
    StringBuilder fileList = new StringBuilder();
    foreach (string url in sasUrlList)
            {
    fileList.AppendLine(url);
    fileList.AppendLine();
            }
    WriteToFile(outFilePath, fileList.ToString());

    // Optionally download the output to the local machine.
    DownloadAssetToLocal(job.Id, outputFolder);
        }

        
    return job;
    }
    ```

2.  在 **Main** 方法中，在你前面添加的行后面添加对 **CreateEncodingJob** 方法的调用。

    ``` {}
    CreateEncodingJob(asset, _singleInputFilePath, _outputFilesFolder);
    ```

3.  将以下帮助器方法添加到类。需要使用这些方法来支持 **CreateEncodingJob** 方法。以下是帮助器方法的摘要。
    -   **GetLatestMediaProcessorByName** 方法返回相应的媒体处理器，用于处理编码、加密和其他相关处理任务。可以使用要创建的处理器的相应字符串名称来创建媒体处理器。可传入 mediaProcessor 参数方法中的可能字符串包括：**Azure Media Encoder**、**Azure Media Packager**、**Azure Media Encryptor** 和 **Storage Decryption**。

        ``` {}
        private static IMediaProcessor GetLatestMediaProcessorByName(string mediaProcessorName)
        {
        // The possible strings that can be passed into the 
        // method for the mediaProcessor parameter:
        //   Azure Media Encoder
        //   Azure Media Packager
        //   Azure Media Encryptor
        //   Storage Decryption

        var processor = _context.MediaProcessors.Where(p => p.Name == mediaProcessorName).
        ToList().OrderBy(p => new Version(p.Version)).LastOrDefault();

        if (processor == null)
        throw new ArgumentException(string.Format("Unknown media processor", mediaProcessorName));

        return processor;
        }
        ```

    -   当你运行作业时，通常需要采用某种方式来跟踪作业进度。以下代码示例定义了 StateChanged 事件处理程序。此事件处理程序将跟踪作业进度，并根据现状提供更新的状态。该代码还定义了 LogJobStop 方法。此帮助器方法将记录错误详细信息。

        ``` {}
        private static void StateChanged(object sender, JobStateChangedEventArgs e)
        {
        Console.WriteLine("Job state changed event:");
        Console.WriteLine("  Previous state:" + e.PreviousState);
        Console.WriteLine("  Current state:" + e.CurrentState);

        switch (e.CurrentState)
            {
        case JobState.Finished:
        Console.WriteLine();
        Console.WriteLine("********************");
        Console.WriteLine("Job is finished.");
        Console.WriteLine("Please wait while local tasks or downloads complete...");
        Console.WriteLine("********************");
        Console.WriteLine();
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
        LogJobStop(job.Id);
        break;
        default:
        break;
            }
        }

        private static void LogJobStop(string jobId)
        {
        StringBuilder builder = new StringBuilder();
        IJob job = GetJob(jobId);

        builder.AppendLine("\nThe job stopped due to cancellation or an error.");
        builder.AppendLine("***************************");
        builder.AppendLine("Job ID:" + job.Id);
        builder.AppendLine("Job Name:" + job.Name);
        builder.AppendLine("Job State:" + job.State.ToString());
        builder.AppendLine("Job started (server UTC time):" + job.StartTime.ToString());
        builder.AppendLine("Media Services account name:" + _accountName);
        // Log job errors if they exist.  
        if (job.State == JobState.Error)
            {
        builder.Append("Error Details:\n");
        foreach (ITask task in job.Tasks)
                {
        foreach (ErrorDetail detail in task.ErrorDetails)
                    {
        builder.AppendLine("  Task Id:" + task.Id);
        builder.AppendLine("    Error Code:" + detail.Code);
        builder.AppendLine("    Error Message:" + detail.Message + "\n");
                    }
                }
            }
        builder.AppendLine("***************************\n");
        // Write the output to a local file and to the console.The template 
        // for an error output file is:JobStop-{JobId}.txt
        string outputFile = _outputFilesFolder + @"\JobStop-" + JobIdAsFileName(job.Id) + ".txt";
        WriteToFile(outputFile, builder.ToString());
        Console.Write(builder.ToString());
        }

        private static void LogJobDetails(string jobId)
        {
        StringBuilder builder = new StringBuilder();
        IJob job = GetJob(jobId);

        builder.AppendLine("\nJob ID:" + job.Id);
        builder.AppendLine("Job Name:" + job.Name);
        builder.AppendLine("Job submitted (client UTC time):" + DateTime.UtcNow.ToString());
        builder.AppendLine("Media Services account name:" + _accountName);

        // Write the output to a local file and to the console.The template 
        // for an error output file is:JobDetails-{JobId}.txt
        string outputFile = _outputFilesFolder + @"\JobDetails-" + JobIdAsFileName(job.Id) + ".txt";
        WriteToFile(outputFile, builder.ToString());
        Console.Write(builder.ToString());
        }
                
        private static string JobIdAsFileName(string jobID)
        {
        return jobID.Replace(":", "_");
        }
        ```

    -   WriteToFile 方法将一个文件写入到指定的输出文件夹。

        ``` {}
        static void WriteToFile(string outFilePath, string fileContent)
        {
        StreamWriter sr = File.CreateText(outFilePath);
        sr.Write(fileContent);
        sr.Close();
        }
        ```

    -   在 Media Services 中为资产编码后，可以访问执行编码作业后生成的输出资产。本演练演示了访问编码作业输出的两种方式：

        -   在服务器上创建资产的 SAS URL。
        -   从服务器下载输出资产。

        GetAssetSasUrlList 方法将创建资产中所有文件的 SAS URL 列表。

        ``` {}
        static List<String> GetAssetSasUrlList(IAsset asset, ILocator locator)
        {
        // Declare a list to contain all the SAS URLs.
        List<String> fileSasUrlList = new List<String>();

        // If the asset has files, build a list of URLs to 
        // each file in the asset and return. 
        foreach (IAssetFile file in asset.AssetFiles)
            {
        string sasUrl = BuildFileSasUrl(file, locator);
        fileSasUrlList.Add(sasUrl);
            }

        // Return the list of SAS URLs.
        return fileSasUrlList;
        }

        // Create and return a SAS URL to a single file in an asset. 
        static string BuildFileSasUrl(IAssetFile file, ILocator locator)
        {
        // Take the locator path, add the file name, and build 
        // a full SAS URL to access this file.This is the only 
        // code required to build the full URL.
        var uriBuilder = new UriBuilder(locator.Path);
        uriBuilder.Path += "/" + file.Name;

        // Optional:print the locator.Path to the asset, and 
        // the full SAS URL to the file
        Console.WriteLine("Locator path: ");
        Console.WriteLine(locator.Path);
        Console.WriteLine();
        Console.WriteLine("Full URL to file: ");
        Console.WriteLine(uriBuilder.Uri.AbsoluteUri);
        Console.WriteLine();


        //Return the SAS URL.
        return uriBuilder.Uri.AbsoluteUri;
        }
        ```

    -   **DownloadAssetToLocal** 方法将资产中的每个文件下载到本地文件夹。在本例中，由于资产是使用一个输入媒体文件创建的，因此，输出资产文件集合包含两个文件：一个 .mp4 文件（编码的媒体文件）和一个 .xml 文件（包含有关资产的元数据）。该方法将下载这两个文件。

        ``` {}
        static IAsset DownloadAssetToLocal(string jobId, string outputFolder)
        {
        // This method illustrates how to download a single asset. 
        // However, you can iterate through the OutputAssets
        // collection, and download all assets if there are many. 

        // Get a reference to the job. 
        IJob job = GetJob(jobId);
        // Get a reference to the first output asset.If there were multiple 
        // output media assets you could iterate and handle each one.
        IAsset outputAsset = job.OutputMediaAssets[0];

        IAccessPolicy accessPolicy = _context.AccessPolicies.Create("File Download Policy", TimeSpan.FromDays(30), AccessPermissions.Read);
        ILocator locator = _context.Locators.CreateSasLocator(outputAsset, accessPolicy);
        BlobTransferClient blobTransfer = new BlobTransferClient
            {
        NumberOfConcurrentTransfers = 10,
        ParallelTransferThreadCount = 10
            };

        var downloadTasks = new List<Task>();
        foreach (IAssetFile outputFile in outputAsset.AssetFiles)
            {
        // Use the following event handler to check download progress.
        outputFile.DownloadProgressChanged += DownloadProgress;

        string localDownloadPath = Path.Combine(outputFolder, outputFile.Name);

        Console.WriteLine("File download path:" + localDownloadPath);

        downloadTasks.Add(outputFile.DownloadAsync(Path.GetFullPath(localDownloadPath), blobTransfer, locator, CancellationToken.None));

        outputFile.DownloadProgressChanged -= DownloadProgress;
            }

        Task.WaitAll(downloadTasks.ToArray());

        return outputAsset;
        }

        static void DownloadProgress(object sender, DownloadProgressChangedEventArgs e)
        {
        Console.WriteLine(string.Format("{0} % download progress.", e.Progress));
        }
        ```

    -   GetJob 和 GetAsset 帮助器方法将查询并返回对作业对象和资产对象的引用，这些对象都具有给定的 ID。你可以使用类似的 LINQ 查询来返回对服务器中其他 Media Services 对象的引用。

        ``` {}
        static IJob GetJob(string jobId)
        {
        // Use a Linq select query to get an updated 
        // reference by Id. 
        var jobInstance =
        from j in _context.Jobs
        where j.Id == jobId
        select j;
        // Return the job reference as an Ijob. 
        IJob job = jobInstance.FirstOrDefault();

        return job;
        }
        static IAsset GetAsset(string assetId)
        {
        // Use a LINQ Select query to get an asset.
        var assetInstance =
        from a in _context.Assets
        where a.Id == assetId
        select a;
        // Reference the asset as an IAsset.
        IAsset asset = assetInstance.FirstOrDefault();

        return asset;
        }
        ```

测试代码
--------

运行程序（按 F5）。控制台将显示类似于下面的输出：

``` {}
Asset name:UploadSingleFile_11/14/2012 10:09:11 PM
Time created:11/14/2012 12:00:00 AM
Created assetFile interview2.wmv
Upload interview2.wmv
Done uploading of interview2.wmv using Upload()

Job ID:nb:jid:UUID:ea8d5a66-86b8-9b4d-84bc-6d406259acb8
Job Name:My encoding job
Job submitted (client UTC time):11/14/2012 10:09:39 PM
Media Services account name:Add-Media-Services-Account-Name
Media Services account location:Add-Media-Services-account-location-name

Job(My encoding job) state:Queued.
Please wait...

Job(My encoding job) state:Processing.
Please wait...

********************
Job(My encoding job) is finished.
Please wait while local tasks or downloads complete...
********************

Locator path:
https://mediasvcd08mtz29tcpws.blob.core.windows-int.net/asset-4f5b42f4-3ade-4c2c
-9d48-44900d4f6b62?st=2012-11-14T22%3A07%3A01Z&se=2012-11-14T23%3A07%3A01Z&sr=c&
si=d07ec40c-02d7-4642-8e54-443b79f3ba3c&sig=XKMo0qJI5w8Fod3NsV%2FBxERnav8Jb6hL7f
xylq3oESc%3D

Full URL to file:
https://mediasvcd08mtz29tcpws.blob.core.windows-int.net/asset-4f5b42f4-3ade-4c2c
-9d48-44900d4f6b62/interview2.mp4?st=2012-11-14T22%3A07%3A01Z&se=2012-11-14T23%3
A07%3A01Z&sr=c&si=d07ec40c-02d7-4642-8e54-443b79f3ba3c&sig=XKMo0qJI5w8Fod3NsV%2F
BxERnav8Jb6hL7fxylq3oESc%3D

Locator path:
https://mediasvcd08mtz29tcpws.blob.core.windows-int.net/asset-4f5b42f4-3ade-4c2c
-9d48-44900d4f6b62?st=2012-11-14T22%3A07%3A01Z&se=2012-11-14T23%3A07%3A01Z&sr=c&
si=d07ec40c-02d7-4642-8e54-443b79f3ba3c&sig=XKMo0qJI5w8Fod3NsV%2FBxERnav8Jb6hL7f
xylq3oESc%3D

Full URL to file:
https://mediasvcd08mtz29tcpws.blob.core.windows-int.net/asset-4f5b42f4-3ade-4c2c
-9d48-44900d4f6b62/interview2_metadata.xml?st=2012-11-14T22%3A07%3A01Z&se=2012-1
1-14T23%3A07%3A01Z&sr=c&si=d07ec40c-02d7-4642-8e54-443b79f3ba3c&sig=XKMo0qJI5w8F
od3NsV%2FBxERnav8Jb6hL7fxylq3oESc%3D

Downloads are in progress, please wait.

File download path:C:\supportFiles\outputfiles\interview2.mp4
1.70952185308162 % download progress.
3.685088 % download progress.
6.488704 % download progress.
6.838087 % download progress.
. . . 
99.076374 % download progress.
99.152267 % download progress.
100 % download progress.
File download path:C:\supportFiles\outputfiles\interview2_metadata.xml
100 % download progress.
```

1.  运行此应用程序后，将发生以下情况：

2.  将一个 .wmv 文件上载到 Media Services。

3.  然后，使用 **Azure 媒体编码器**的 **H264 Broadband 720p** 预设来为该文件编码。

4.  FileSasUrlList.txt 文件在 \\supportFiles\\outputFiles 文件夹中创建。该文件包含所编码资产的 URL。

    若要播放媒体文件，请从文本文件中复制资产的 URL，然后将它粘贴到浏览器中。

5.  .mp4 媒体文件和 \_metadata.xml 文件将下载到 outputFiles 文件夹中。

**说明**

在 Media Services 对象模型中，资产是代表一个或多个文件的 Media Services 内容集合对象。定位器路径提供 Azure Blob URL，该 URL 是此资产在 Azure 存储空间中的基路径。若要访问资产中的特定文件，请在定位器基路径中添加一个文件名。

后续步骤
--------

本演练演示了生成简单 Media Services 应用程序所要执行的编程任务序列。你已学习了基本的 Media Services 编程任务，包括获取服务器上下文、创建资产、为资产编码，以及下载或访问服务器上的资产。有关后续步骤和其他高级开发任务，请参阅以下主题：

-   [如何使用 Media Services](http://www.windowsazure.cn/zh-cn/develop/net/how-to-guides/media-services/)
-   [使用 Media Services REST API 生成应用程序](http://msdn.microsoft.com/zh-cn/library/windowsazure/hh973618.aspx)

