<properties 
	pageTitle="实现故障转移流式处理方案 | Azure" 
	description="本主题演示如何实现故障转移流式处理方案。" 
	services="media-services" 
	documentationCenter="" 
	authors="Juliako" 
	manager="dwrede" 
	editor=""/>

<tags
	ms.service="media-services"
 	ms.date="04/18/2016"  
	wacn.date="06/27/2016"/>

#实现故障转移流式处理方案

本演练演示如何将内容 (blob) 从一个资产复制到另一个资产，以便处理按需流式处理的冗余。此方案适用于想要将其 CDN 设置为当我们的某个数据中心出现中断时，在两个数据中心之间进行故障转移的客户。本演练使用 Azure 媒体服务 SDK、Azure 媒体服务 REST API 和 Azure 存储空间 SDK 来演示以下任务。

1. 在“数据中心 A”中设置一个媒体服务帐户。
1. 将一个夹层文件上载到源资产中。
1. 将该资产编码成多比特率 MP4 文件。 
1. 为源资产创建一个只读 SAS 定位器，以获取对关联到源资产的存储帐户中的容器的读取权限。
1. 从上一步创建的只读 SAS 定位器中获取源资产的容器名称。我们需要使用这些信息在存储帐户（本主题后面将会介绍）之间复制 BLOB。
1. 为通过编码任务创建的资产创建源定位器。 

然后，若要处理故障转移，请执行以下操作：

1. 在“数据中心 B”中设置一个媒体服务帐户。
1. 在目标媒体服务帐户中创建一个目标空资产。
1. 为目标空资产创建一个写入 SAS 定位器，以获取对关联到目标资产的目标存储帐户中的容器的写入权限。
1. 使用 Azure 存储空间 SDK 在“数据中心 A”中的源存储帐户与“数据中心 B”中的目标存储帐户（这些存储帐户与所需资产关联）之间复制 blob（资产文件）。
1. 将复制到目标 blob 容器的 blob（资产文件）与目标资产关联。 
1. 在“数据中心 B”中为资产创建源定位器，并指定为“数据中心 A”中的资产生成的定位器 ID。 
1. 这样，便会提供 URL 的相对路径相同（只有基本 URL 不同）的流式处理 URL。 
 
然后，若要处理任何中断情况，可以在这些源定位器之上创建 CDN。

请注意以下事项：

- 当前版本的媒体服务 SDK 不支持使用指定的定位器 ID 创建定位器。若要完成此任务，我们将使用媒体服务 REST API。
- 当前版本的媒体服务 SDK 不支持以编程方式生成会将资产与资产文件关联的 IAssetFile 信息。若要完成此任务，我们将使用 CreateFileInfos 媒体服务 REST API。 
- 不支持使用存储加密资产 (AssetCreationOptions.StorageEncrypted) 进行复制（因为两个媒体服务帐户中的加密密钥将会有所不同）。 
- 如果你想要利用动态打包，则必须先获取至少一个按需流式处理保留单位。有关详细信息，请参阅[动态打包资产](/documentation/articles/media-services-dynamic-packaging-overview/)。
 

>[AZURE.NOTE]请考虑将媒体服务[复制器工具](http://replicator.codeplex.com/)用作备用选项，以手动实现故障转移流式处理方案。此工具可用于在两个媒体服务帐户之间复制资产。

##先决条件
 
- 在新的或现有的 Azure 订阅中拥有两个媒体服务帐户。请参阅[如何创建媒体服务帐户](/documentation/articles/media-services-create-account/)。
- 操作系统：Windows 7、Windows 2008 R2 或 Windows 8。
- .NET Framework 4.5 或 .NET Framework 4。
- Visual Studio 2010 SP1 或更高版本（专业版、高级专业版、旗舰版或学习版）。
 
##设置项目

在本部分中，你将创建和设置一个 C# 控制台应用程序项目。

1. 使用 Visual Studio 创建包含 C# 控制台应用程序项目的新解决方案。针对“名称”输入“HandleRedundancyForOnDemandStreaming”，然后单击“确定”。
1. 在与 HandleRedundancyForOnDemandStreaming.csproj 项目文件相同的级别上创建 SupportFiles 文件夹。在 SupportFiles 文件夹下创建 OutputFiles 和 MP4Files 文件夹。将一个 .mp4 文件复制到 MP4Files 文件夹（在本示例中使用 BigBuckBunny.mp4 文件）。 
1. 使用 **Nuget** 添加对媒体服务相关 DLL 的引用。在 Visual Studio 主菜单中，选择“工具”->“库程序包管理器”->“程序包管理器控制台”。在控制台窗口中，键入 Install-Package windowsazure.mediaservices，然后按 Enter。
1. 添加此项目所需的其他引用：System.Configuration、System.Runtime.Serialization 和 System.Web。
1. 将默认添加到 Programs.cs 文件中的 using 语句替换为以下语句：

		using System;
		using System.Configuration;
		using System.Globalization;
		using System.IO;
		using System.Net;
		using System.Runtime.Serialization.Json;
		using System.Text;
		using System.Threading;
		using System.Threading.Tasks;
		using System.Web;
		using System.Xml;
		using System.Linq;
		using Microsoft.WindowsAzure;
		using Microsoft.WindowsAzure.MediaServices.Client;
		using Microsoft.WindowsAzure.Storage;
		using Microsoft.WindowsAzure.Storage.Blob;
		using Microsoft.WindowsAzure.Storage.Auth;



1. 将 appSettings 节添加到 .config 文件，并根据媒体服务和 Storage 密钥与名称值更新值。

		<appSettings>
		  <add key="MediaServicesAccountNameSource" value="Media-Services-Account-Name-Source"/>
		  <add key="MediaServicesAccountKeySource" value="Media-Services-Account-Key-Source"/>
		  <add key="MediaServicesStorageAccountNameSource" value="Media-Services-Storage-Account-Name-Source"/>
		  <add key="MediaServicesStorageAccountKeySource" value="Media-Services-Storage-Account-Key-Source"/>
		  <add key="MediaServicesAccountNameTarget" value="Media-Services-Account-Name-Target" />
		  <add key="MediaServicesAccountKeyTarget" value=" Media-Services-Account-Key-Target" />
		  <add key="MediaServicesStorageAccountNameTarget" value=" Media-Services-Storage-Account-Name-Target" />
		  <add key="MediaServicesStorageAccountKeyTarget" value=" Media-Services-Storage-Account-Key-Target" />
		</appSettings>

##添加用于处理按需流式处理冗余的代码。



1. 将以下类级字段添加到 Program 类。
	
		// Read values from the App.config file.
		private static readonly string MediaServicesAccountNameSource = ConfigurationManager.AppSettings["MediaServicesAccountNameSource"];
		private static readonly string MediaServicesAccountKeySource = ConfigurationManager.AppSettings["MediaServicesAccountKeySource"];
		private static readonly string StorageNameSource = ConfigurationManager.AppSettings["MediaServicesStorageAccountNameSource"];
		private static readonly string StorageKeySource = ConfigurationManager.AppSettings["MediaServicesStorageAccountKeySource"];
		
		private static readonly string MediaServicesAccountNameTarget = ConfigurationManager.AppSettings["MediaServicesAccountNameTarget"];
		private static readonly string MediaServicesAccountKeyTarget = ConfigurationManager.AppSettings["MediaServicesAccountKeyTarget"];
		private static readonly string StorageNameTarget = ConfigurationManager.AppSettings["MediaServicesStorageAccountNameTarget"];
		private static readonly string StorageKeyTarget = ConfigurationManager.AppSettings["MediaServicesStorageAccountKeyTarget"];
		
		// Base support files path.  Update this field to point to the base path  
		// for the local support files folder that you create. 
		private static readonly string SupportFiles = Path.GetFullPath(@"../..\SupportFiles");
		
		// Paths to support files (within the above base path). 
		private static readonly string SingleInputMp4Path = Path.GetFullPath(SupportFiles + @"\MP4Files\BigBuckBunny.mp4");
		private static readonly string OutputFilesFolder = Path.GetFullPath(SupportFiles + @"\OutputFiles");
		
		// Class-level field used to keep a reference to the service context.
		static private CloudMediaContext _contextSource = null;
		static private CloudMediaContext _contextTarget = null;
		static private MediaServicesCredentials _cachedCredentialsSource = null;
		static private MediaServicesCredentials _cachedCredentialsTarget = null;



1. 请使用以下定义替换默认的 Main 方法定义：
		
		static void Main(string[] args)
		{
		    _cachedCredentialsSource = new MediaServicesCredentials(
		                    MediaServicesAccountNameSource,
		                    MediaServicesAccountKeySource);
		
		    _cachedCredentialsTarget = new MediaServicesCredentials(
		                    MediaServicesAccountNameTarget,
		                    MediaServicesAccountKeyTarget);
		
		    // Get server context.    
		    _contextSource = new CloudMediaContext(_cachedCredentialsSource);
		    _contextTarget = new CloudMediaContext(_cachedCredentialsTarget);
		
		
		    IAsset assetSingleFile = CreateAssetAndUploadSingleFile(_contextSource,
		                                AssetCreationOptions.None,
		                                SingleInputMp4Path);
		
		    IJob job = CreateEncodingJob(_contextSource, assetSingleFile);
		
		    if (job.State != JobState.Error)
		    {
		        IAsset sourceOutputAsset = job.OutputMediaAssets[0];
		        // Get the locator for Smooth Streaming
		        var sourceOriginLocator = GetStreamingOriginLocator(_contextSource, sourceOutputAsset);
		
		        Console.WriteLine("Locator Id: {0}", sourceOriginLocator.Id);
		
		
		        // 1.Create a read-only SAS locator for the source asset to have read access to the container in the source Storage account (associated with the source Media Services account)
		        var readSasLocator = GetSasReadLocator(_contextSource, sourceOutputAsset);
		
		
		        // 2.Get the container name of the source asset from the read-only SAS locator created in the previous step
		        string containerName = (new Uri(readSasLocator.Path)).Segments[1];
		
		
		        // 3.Create a target empty asset in the target Media Services account
		        var targetAsset = CreateTargetEmptyAsset(_contextTarget, containerName);
		
		        // 4.Create a write SAS locator for the target empty asset to have write access to the container in the target Storage account (associated with the target Media Services account)
		        ILocator writeSasLocator = CreateSasWriteLocator(_contextTarget, targetAsset);
		
		        // Get asset container name.
		        string targetContainerName = (new Uri(writeSasLocator.Path)).Segments[1];
		
		
		        // 5.Copy the blobs in the source container (source asset) to the target container (target empty asset)
		        CopyBlobsFromDifferentStorage(containerName, targetContainerName, StorageNameSource, StorageKeySource, StorageNameTarget, StorageKeyTarget);
		
		
		        // 6.Use the CreateFileInfos Media Services REST API to automatically generate all the IAssetFile's for the target asset. 
		        //      This API call is not supported in the current Media Services SDK for .NET. 
		        CreateFileInfosForAssetWithRest(_contextTarget, targetAsset, MediaServicesAccountNameTarget, MediaServicesAccountKeyTarget);
		
		        // Check if the AssetFiles are now  associated with the asset.
		        Console.WriteLine("Asset files assocated with the {0} asset:", targetAsset.Name);
		        foreach (var af in targetAsset.AssetFiles)
		        {
		            Console.WriteLine(af.Name);
		        }
		
		        // 7.Copy the Origin locator of the source asset to the target asset by using the same Id
		        var replicatedLocatorPath = CreateOriginLocatorWithRest(_contextTarget,
		                    MediaServicesAccountNameTarget, MediaServicesAccountKeyTarget,
		                    sourceOriginLocator.Id, targetAsset.Id);
		
		        // Create a full URL to the manifest file. Use this for playback
		        // in streaming media clients. 
		        string originalUrlForClientStreaming = sourceOriginLocator.Path + GetPrimaryFile(sourceOutputAsset).Name + "/manifest";
		
		        Console.WriteLine("Original Locator Path: {0}\n", originalUrlForClientStreaming);
		
		        string replicatedUrlForClientStreaming = replicatedLocatorPath + GetPrimaryFile(sourceOutputAsset).Name + "/manifest";
		
		        Console.WriteLine("Replicated Locator Path: {0}", replicatedUrlForClientStreaming);
		
		        readSasLocator.Delete();
		        writeSasLocator.Delete();
		}

1. 下面定义了从 Main 调用的方法定义。
		
		public static IAsset CreateAssetAndUploadSingleFile(CloudMediaContext context,
		                                                AssetCreationOptions assetCreationOptions,
		                                                string singleFilePath)
		{
		    // For the AssetCreationOptions you can specify 
		    // encryption options.
		    //      None:  no encryption. By default, storage encryption is used. If you want to 
		    //        create an unencrypted asset, you must set this option.
		    //      StorageEncrypted:  storage encryption. Encrypts a clear input file 
		    //        before it is uploaded to Azure storage. This is the default if not specified
		    //      CommonEncryptionProtected:  for Common Encryption Protected (CENC) files. An 
		    //        example is a set of files that are already PlayReady encrypted. 
		
		    var assetName = "UploadSingleFile_" + DateTime.UtcNow.ToString();
		
		    var asset = context.Assets.Create(assetName, assetCreationOptions);
		
		    Console.WriteLine("Asset name: " + asset.Name);
		
		    var fileName = Path.GetFileName(singleFilePath);
		
		    var assetFile = asset.AssetFiles.Create(fileName);
		
		    Console.WriteLine("Created assetFile {0}", assetFile.Name);
		
		    Console.WriteLine("Upload {0}", assetFile.Name);
		
		    assetFile.Upload(singleFilePath);
		    Console.WriteLine("Done uploading of {0}", assetFile.Name);
		
		    return asset;
		}
		
		public static IJob CreateEncodingJob(CloudMediaContext context, IAsset asset)
		{
		    // Declare a new job.
		    IJob job = context.Jobs.Create("My encoding job");
		
		    // Get a media processor reference, and pass to it the name of the 
		    // processor to use for the specific task.
		    IMediaProcessor processor = GetLatestMediaProcessorByName(context,
		                                            "Media Encoder Standard");
		
		    // Create a task with the encoding details, using a string preset.
		    // In this case "H264 Multiple Bitrate 720p" preset is used.
		    ITask task = job.Tasks.AddNew("My encoding task",
		        processor,
		        "H264 Multiple Bitrate 720p",
		        TaskOptions.ProtectedConfiguration);
		
		    // Specify the input asset to be encoded.
		    task.InputAssets.Add(asset);
		
		    // Add an output asset to contain the results of the job. 
		    // This output is specified as AssetCreationOptions.None, which 
		    // means the output asset is in the clear (unencrypted). 
		    var outputAssetName = "OutputAsset_" + Guid.NewGuid();
		    task.OutputAssets.AddNew(outputAssetName,
		        AssetCreationOptions.None);
		
		    // Use the following event handler to check job progress.  
		    job.StateChanged += new
		            EventHandler<JobStateChangedEventArgs>(StateChanged);
		
		    // Launch the job.
		    job.Submit();
		
		    // Optionally log job details. This displays basic job details
		    // to the console and saves them to a JobDetails-{JobId}.txt file 
		    // in your output folder.
		    LogJobDetails(context, job.Id);
		
		    // Check job execution and wait for job to finish. 
		    Task progressJobTask = job.GetExecutionProgressTask(CancellationToken.None);
		    progressJobTask.Wait();
		
		    // Get an updated job reference.
		    job = GetJob(context, job.Id);
		
		    // Since we the output asset contains a set of Smooth Streaming files,
		    // set the .ism file to be the primary file
		    if (job.State != JobState.Error)
		        SetPrimaryFile(job.OutputMediaAssets[0]);
		
		    return job;
		}
		
		// Create a locator URL to a streaming media asset 
		// on an origin server.
		public static ILocator GetStreamingOriginLocator(CloudMediaContext context, IAsset assetToStream)
		{
		    // Get a reference to the streaming manifest file from the  
		    // collection of files in the asset. 
		    IAssetFile manifestFile = GetPrimaryFile(assetToStream);
		
		    // Create a 30-day readonly access policy. 
        	// You cannot create a streaming locator using an AccessPolicy that includes write or delete permissions.            
        
		    IAccessPolicy policy = context.AccessPolicies.Create("Streaming policy",
		        TimeSpan.FromDays(30),
		        AccessPermissions.Read);
		
		    // Create a locator to the streaming content on an origin. 
		    ILocator originLocator = context.Locators.CreateLocator(LocatorType.OnDemandOrigin,
		        assetToStream,
		        policy,
		        DateTime.UtcNow.AddMinutes(-5));
		
		    // Return the locator. 
		    return originLocator;
		}
		
		public static ILocator GetSasReadLocator(CloudMediaContext context, IAsset asset)
		{
		    IAccessPolicy accessPolicy = context.AccessPolicies.Create("File Download Policy",
		        TimeSpan.FromDays(30), AccessPermissions.Read);
		
		    ILocator sasLocator = context.Locators.CreateLocator(LocatorType.Sas,
		        asset, accessPolicy);
		
		    return sasLocator;
		}
		
		public static ILocator CreateSasWriteLocator(CloudMediaContext context, IAsset asset)
		{
		
		    IAccessPolicy writePolicy = context.AccessPolicies.Create("Write Policy",
		        TimeSpan.FromDays(30), AccessPermissions.Write);
		
		    ILocator sasLocator = context.Locators.CreateLocator(LocatorType.Sas,
		        asset, writePolicy);
		
		    return sasLocator;
		}
		
		public static IAsset CreateTargetEmptyAsset(CloudMediaContext context, string containerName)
		{
		    // Create a new asset.
		    IAsset assetToBeProcessed = context.Assets.Create(containerName,
		        AssetCreationOptions.None);
		
		    return assetToBeProcessed;
		}
		
		public static void CreateFileInfosForAssetWithRest(CloudMediaContext context, IAsset asset, string mediaServicesAccountNameTarget,
		    string mediaServicesAccountKeyTarget)
		{
		    string apiServer = "";
		    string scope = "";
		    string acsBaseAddress = "";
		
		    string acsToken = GetAcsBearerToken(mediaServicesAccountNameTarget,
		                            mediaServicesAccountKeyTarget, scope, acsBaseAddress);
		
		    if (!string.IsNullOrEmpty(acsToken))
		    {
		        CreateFileInfos(apiServer, acsToken, asset.Id);
		    }
		}
		
		public static string CreateOriginLocatorWithRest(CloudMediaContext context, string mediaServicesAccountNameTarget,
		    string mediaServicesAccountKeyTarget, string locatorIdToReplicate, string targetAssetId)
		{
		    //Make sure we are not asking for a duplicate:
		    var locator = context.Locators.Where(l => l.Id == locatorIdToReplicate).FirstOrDefault();
		    if (locator != null)
		        return "";
		
		    string locatorNewPath = "";
		    string apiServer = "";
		    string scope = "";
		    string acsBaseAddress = "";
		
		    string acsToken = GetAcsBearerToken(mediaServicesAccountNameTarget,
		                            mediaServicesAccountKeyTarget, scope, acsBaseAddress);
		
		    if (!string.IsNullOrEmpty(acsToken))
		    {
		        var asset = context.Assets.Where(a => a.Id == targetAssetId).FirstOrDefault();

            	// You cannot create a streaming locator using an AccessPolicy that includes write or delete permissions.            
		        var accessPolicy = context.AccessPolicies.Create("RestTest", TimeSpan.FromDays(100),
		                                                            AccessPermissions.Read);
		        if (asset != null)
		        {
		            string redirectedServiceUri = null;
		
		            var xmlResponse = CreateLocator(apiServer, out redirectedServiceUri, acsToken,
		                                                        asset.Id, accessPolicy.Id,
		                                                        (int)LocatorType.OnDemandOrigin,
		                                                        DateTime.UtcNow.AddMinutes(-10), locatorIdToReplicate);
		
		            Console.WriteLine("Redirected to: " + redirectedServiceUri);
		            if (xmlResponse != null)
		            {
		                Console.WriteLine(String.Format("Locator Id: {0}",
		                                                xmlResponse.GetElementsByTagName("Id")[0].InnerText));
		                Console.WriteLine(String.Format("Locator Path: {0}",
		                        xmlResponse.GetElementsByTagName("Path")[0].InnerText));
		
		
		                locatorNewPath = xmlResponse.GetElementsByTagName("Path")[0].InnerText;
		            }
		        }
		    }
		
		    return locatorNewPath;
		}
		
		
		public static void SetPrimaryFile(IAsset asset)
		{
		
		    var ismAssetFiles = asset.AssetFiles.ToList().
		                Where(f => f.Name.EndsWith(".ism", StringComparison.OrdinalIgnoreCase))
		                .ToArray();
		
		    if (ismAssetFiles.Count() != 1)
		        throw new ArgumentException("The asset should have only one, .ism file");
		
		    ismAssetFiles.First().IsPrimary = true;
		    ismAssetFiles.First().Update();
		}
		
		public static IAssetFile GetPrimaryFile(IAsset asset)
		{
		    var theManifest =
		            from f in asset.AssetFiles
		            where f.Name.EndsWith(".ism")
		            select f;
		
		    // Cast the reference to a true IAssetFile type. 
		    IAssetFile manifestFile = theManifest.First();
		
		    return manifestFile;
		}
		
		public static IAsset RefreshAsset(CloudMediaContext context, IAsset asset)
		{
		    asset = context.Assets.Where(a => a.Id == asset.Id).FirstOrDefault();
		    return asset;
		}
		
		
		public static void CopyBlobsFromDifferentStorage(string sourceContainerName, string targetContainerName,
		                                    string srcAccountName, string srcAccountKey,
		                                    string destAccountName, string destAccountKey)
		{
		    var srcAccount = new CloudStorageAccount(new StorageCredentials(srcAccountName, srcAccountKey), true);
		    var destAccount = new CloudStorageAccount(new StorageCredentials(destAccountName, destAccountKey), true);
		
		    var cloudBlobClient = srcAccount.CreateCloudBlobClient();
		    var targetBlobClient = destAccount.CreateCloudBlobClient();
		
		    var sourceContainer = cloudBlobClient.GetContainerReference(sourceContainerName);
		    var targetContainer = targetBlobClient.GetContainerReference(targetContainerName);
		    targetContainer.CreateIfNotExists();
		
		
		    string blobToken = sourceContainer.GetSharedAccessSignature(new SharedAccessBlobPolicy()
		    {
		        // Specify the expiration time for the signature.
		        SharedAccessExpiryTime = DateTime.Now.AddDays(1),
		        // Specify the permissions granted by the signature.
		        Permissions = SharedAccessBlobPermissions.Write | SharedAccessBlobPermissions.Read
		    });
		
		
		    foreach (var sourceBlob in sourceContainer.ListBlobs())
		    {
		        string fileName = (sourceBlob as ICloudBlob).Name;
		        var sourceCloudBlob = sourceContainer.GetBlockBlobReference(fileName);
		        sourceCloudBlob.FetchAttributes();
		
		        if (sourceCloudBlob.Properties.Length > 0)
		        {
		            var destinationBlob = targetContainer.GetBlockBlobReference(fileName);
		            destinationBlob.StartCopyFromBlob(new Uri(sourceBlob.Uri.AbsoluteUri + blobToken));
		
		            while (true)
		            {
		                // The StartCopyFromBlob is an async operation, 
		                // so we want to check if the copy operation is completed before proceeding. 
		                // To do that, we call FetchAttributes on the blob and check the CopyStatus. 
		                destinationBlob.FetchAttributes();
		                if (destinationBlob.CopyState.Status != CopyStatus.Pending)
		                {
		                    break;
		                }
		                //It's still not completed. So wait for some time.
		                System.Threading.Thread.Sleep(1000);
		            }
		        }
		
		        Console.WriteLine(fileName);
		    }
		
		    Console.WriteLine("Done copying.");
		}
		private static IMediaProcessor GetLatestMediaProcessorByName(CloudMediaContext context, string mediaProcessorName)
		{
	
		    var processor = context.MediaProcessors.Where(p => p.Name == mediaProcessorName).
		        ToList().OrderBy(p => new Version(p.Version)).LastOrDefault();
		
		    if (processor == null)
		        throw new ArgumentException(string.Format("Unknown media processor", mediaProcessorName));
		
		    return processor;
		}
		
		// This method is a handler for events that track job progress.   
		private static void StateChanged(object sender, JobStateChangedEventArgs e)
		{
		    Console.WriteLine("Job state changed event:");
		    Console.WriteLine("  Previous state: " + e.PreviousState);
		    Console.WriteLine("  Current state: " + e.CurrentState);
		
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
		            LogJobStop(null, job.Id);
		            break;
		        default:
		            break;
		    }
		}
		
		private static void LogJobStop(CloudMediaContext context, string jobId)
		{
		    StringBuilder builder = new StringBuilder();
		    IJob job = GetJob(context, jobId);
		
		    builder.AppendLine("\nThe job stopped due to cancellation or an error.");
		    builder.AppendLine("***************************");
		    builder.AppendLine("Job ID: " + job.Id);
		    builder.AppendLine("Job Name: " + job.Name);
		    builder.AppendLine("Job State: " + job.State.ToString());
		    builder.AppendLine("Job started (server UTC time): " + job.StartTime.ToString());
		    // Log job errors if they exist.  
		    if (job.State == JobState.Error)
		    {
		        builder.Append("Error Details: \n");
		        foreach (ITask task in job.Tasks)
		        {
		            foreach (ErrorDetail detail in task.ErrorDetails)
		            {
		                builder.AppendLine("  Task Id: " + task.Id);
		                builder.AppendLine("    Error Code: " + detail.Code);
		                builder.AppendLine("    Error Message: " + detail.Message + "\n");
		            }
		        }
		    }
		    builder.AppendLine("***************************\n");
		    // Write the output to a local file and to the console. The template 
		    // for an error output file is:  JobStop-{JobId}.txt
		    string outputFile = OutputFilesFolder + @"\JobStop-" + JobIdAsFileName(job.Id) + ".txt";
		    WriteToFile(outputFile, builder.ToString());
		    Console.Write(builder.ToString());
		}
		
		private static void LogJobDetails(CloudMediaContext context, string jobId)
		{
		    StringBuilder builder = new StringBuilder();
		    IJob job = GetJob(context, jobId);
		
		    builder.AppendLine("\nJob ID: " + job.Id);
		    builder.AppendLine("Job Name: " + job.Name);
		    builder.AppendLine("Job submitted (client UTC time): " + DateTime.UtcNow.ToString());
		
		    // Write the output to a local file and to the console. The template 
		    // for an error output file is:  JobDetails-{JobId}.txt
		    string outputFile = OutputFilesFolder + @"\JobDetails-" + JobIdAsFileName(job.Id) + ".txt";
		    WriteToFile(outputFile, builder.ToString());
		    Console.Write(builder.ToString());
		}
		
		// Replace ":" with "_" in Job id values so they can 
		// be used as log file names.  
		private static string JobIdAsFileName(string jobID)
		{
		    return jobID.Replace(":", "_");
		}
		
		// Write method output to the output files folder.
		private static void WriteToFile(string outFilePath, string fileContent)
		{
		    StreamWriter sr = File.CreateText(outFilePath);
		    sr.WriteLine(fileContent);
		    sr.Close();
		}
		
		private static IJob GetJob(CloudMediaContext context, string jobId)
		{
		    // Use a Linq select query to get an updated 
		    // reference by Id. 
		    var jobInstance =
		        from j in context.Jobs
		        where j.Id == jobId
		        select j;
		
		    // Return the job reference as an Ijob. 
		    IJob job = jobInstance.FirstOrDefault();
		
		    return job;
		}
		
		private static IAsset GetAsset(CloudMediaContext context, string assetId)
		{
		    // Use a LINQ Select query to get an asset.
		    var assetInstance =
		        from a in context.Assets
		        where a.Id == assetId
		        select a;
		
		    // Reference the asset as an IAsset.
		    IAsset asset = assetInstance.FirstOrDefault();
		
		    return asset;
		}
		
		public static void DeleteAllAssets(CloudMediaContext context)
		{
		    // Loop through and delete all assets.
		    foreach (IAsset asset in context.Assets)
		    {
		        DeleteLocatorsForAsset(context, asset);
		
		        asset.Delete();
		    }
		}
		
		public static void DeleteLocatorsForAsset(CloudMediaContext context, IAsset asset)
		{
		    string assetId = asset.Id;
		    var locators = from a in context.Locators
		                    where a.AssetId == assetId
		                    select a;
		    foreach (var locator in locators)
		    {
		        Console.WriteLine("Deleting locator {0} for asset {1}", locator.Path, assetId);
		
		        locator.Delete();
		    }
		}
		
		public static void DeleteAccessPolicy(CloudMediaContext context, string existingPolicyId)
		{
		    // To delete a specific access policy, get a reference to the policy.  
		    // based on the policy Id passed to the method.
		    var policyInstance =
		            from p in context.AccessPolicies
		            where p.Id == existingPolicyId
		            select p;
		
		    IAccessPolicy policy = policyInstance.FirstOrDefault();
		
		    policy.Delete();
		
		}
		
		//////////////////////////////////////////////////////
		/// The following methods use REST calls.
		//////////////////////////////////////////////////////
		
		public static string GetAcsBearerToken(string clientId, string clientSecret, string scope, string accessControlServiceUri)
		{
		    if (string.IsNullOrEmpty(clientId))
		        throw new ArgumentNullException("clientId");
		
		    if (string.IsNullOrEmpty(clientSecret))
		        throw new ArgumentNullException("clientSecret");
		
		    if (string.IsNullOrEmpty(scope))
		    {
		        scope = "urn:WindowsAzureMediaServices";
		    }
		    else if (!scope.ToLower().StartsWith("urn:"))
		    {
		        scope = "urn:" + scope;
		    }
		
		    if (string.IsNullOrEmpty(accessControlServiceUri))
		    {
		        accessControlServiceUri = "https://wamsprodglobal001acs.accesscontrol.chinacloudapi.cn/v2/OAuth2-13";
		    }
		    else if (!accessControlServiceUri.ToLower().EndsWith("/v2/oauth2-13"))
		    {
		        accessControlServiceUri = accessControlServiceUri.TrimEnd('/') + "/v2/OAuth2-13";
		    }
		
		    HttpWebRequest request = (HttpWebRequest)HttpWebRequest.Create(accessControlServiceUri);
		    request.Method = "POST";
		    request.ContentType = "application/x-www-form-urlencoded";
		    request.KeepAlive = true;
		    string acsBearerToken = null;
		
		    var requestBytes = Encoding.ASCII.GetBytes("grant_type=client_credentials&client_id=" +
		        clientId + "&client_secret=" + HttpUtility.UrlEncode(clientSecret) +
		        "&scope=" + HttpUtility.UrlEncode(scope));
		    request.ContentLength = requestBytes.Length;
		
		    var requestStream = request.GetRequestStream();
		    requestStream.Write(requestBytes, 0, requestBytes.Length);
		    requestStream.Close();
		
		    var response = (HttpWebResponse)request.GetResponse();
		
		    if (response.StatusCode == HttpStatusCode.OK)
		    {
		        using (Stream responseStream = response.GetResponseStream())
		        {
		            using (StreamReader stream = new StreamReader(responseStream))
		            {
		                string responseString = stream.ReadToEnd();
		                var reader = JsonReaderWriterFactory.CreateJsonReader(Encoding.UTF8.GetBytes(responseString),
		                    new XmlDictionaryReaderQuotas());
		
		                while (reader.Read())
		                {
		                    if ((reader.Name == "access_token") && (reader.NodeType == XmlNodeType.Element))
		                    {
		                        if (reader.Read())
		                        {
		                            acsBearerToken = reader.Value;
		                            break;
		                        }
		                    }
		                }
		            }
		        }
		    }
		
		    return acsBearerToken;
		}
		
		public static XmlDocument CreateLocator(string mediaServicesApiServerUri,
		                                        out string redirectedMediaServicesApiServerUri,
		                                        string acsBearerToken, string assetId,
		                                        string accessPolicyId, int locatorType,
		                                        DateTime startTime, string locatorIdToReplicate = null,
		                                        bool autoRedirect = true)
		{
		    if (string.IsNullOrEmpty(mediaServicesApiServerUri))
		    {
		        mediaServicesApiServerUri = "https://media.chinacloudapi.cn/api/";
		    }
		    if (!mediaServicesApiServerUri.EndsWith("/"))
		        mediaServicesApiServerUri = mediaServicesApiServerUri + "/";
		
		    if (string.IsNullOrEmpty(acsBearerToken)) throw new ArgumentNullException("acsBearerToken");
		    if (string.IsNullOrEmpty(assetId)) throw new ArgumentNullException("assetId");
		    if (string.IsNullOrEmpty(accessPolicyId)) throw new ArgumentNullException("accessPolicyId");
		
		    redirectedMediaServicesApiServerUri = null;
		    XmlDocument xmlResponse = null;
		
		    StringBuilder sb = new StringBuilder();
		    sb.Append("{ "AssetId" : "" + assetId + """);
		    sb.Append(", "AccessPolicyId" : "" + accessPolicyId + """);
		    sb.Append(", "Type" : "" + locatorType + """);
		    if (startTime != DateTime.MinValue)
		        sb.Append(", "StartTime" : "" + startTime.ToString("G", CultureInfo.CreateSpecificCulture("zh-cn")) + """);
		    if (!string.IsNullOrEmpty(locatorIdToReplicate))
		        sb.Append(", "Id" : "" + locatorIdToReplicate + """);
		    sb.Append("}");
		
		    string requestbody = sb.ToString();
		
		    try
		    {
		        var request = GenerateRequest("POST", mediaServicesApiServerUri, "Locators",
		            null, acsBearerToken, requestbody);
		        var response = (HttpWebResponse)request.GetResponse();
		
		        switch (response.StatusCode)
		        {
		            case HttpStatusCode.MovedPermanently:
		                //Recurse once with the mediaServicesApiServerUri redirect Location:
		                if (autoRedirect)
		                {
		                    redirectedMediaServicesApiServerUri = response.Headers["Location"];
		                    string secondRedirection = null;
		                    xmlResponse = CreateLocator(redirectedMediaServicesApiServerUri,
		                                                out secondRedirection, acsBearerToken,
		                                                assetId, accessPolicyId, locatorType,
		                                                startTime, locatorIdToReplicate, false);
		                }
		                else
		                {
		                    Console.WriteLine("Redirection to {0} failed.",
		                        mediaServicesApiServerUri);
		                    return null;
		                }
		                break;
		            case HttpStatusCode.Created:
		                using (Stream responseStream = response.GetResponseStream())
		                {
		                    using (StreamReader stream = new StreamReader(responseStream))
		                    {
		                        string responseString = stream.ReadToEnd();
		                        var reader = JsonReaderWriterFactory.
		                            CreateJsonReader(Encoding.UTF8.GetBytes(responseString),
		                                new XmlDictionaryReaderQuotas());
		
		                        xmlResponse = new XmlDocument();
		                        reader.Read();
		                        xmlResponse.LoadXml(reader.ReadInnerXml());
		                    }
		                }
		                break;
		
		            default:
		                Console.WriteLine(response.StatusDescription);
		                break;
		        }
		    }
		    catch (WebException ex)
		    {
		        Console.WriteLine(ex.Message);
		    }
		
		    return xmlResponse;
		}
		
		public static void CreateFileInfos(string mediaServicesApiServerUri,
		                            string acsBearerToken,
		                            string assetId
		                            )
		{
		    if (String.IsNullOrEmpty(mediaServicesApiServerUri))
		    {
		        mediaServicesApiServerUri = "https://media.chinacloudapi.cn/api/";
		    }
		    if (!mediaServicesApiServerUri.EndsWith("/"))
		        mediaServicesApiServerUri = mediaServicesApiServerUri + "/";
		
		    if (String.IsNullOrEmpty(acsBearerToken)) throw new ArgumentNullException("acsBearerToken");
		    if (String.IsNullOrEmpty(assetId)) throw new ArgumentNullException("assetId");
		
		
		    string id = assetId.Replace(":", "%");
		
		    UriBuilder builder = new UriBuilder(mediaServicesApiServerUri);
		    builder.Path = Path.Combine(builder.Path, "CreateFileInfos");
		    builder.Query = String.Format(CultureInfo.InvariantCulture, "assetid='{0}'", assetId);
		
		    try
		    {
		        var request = GenerateRequest("GET", mediaServicesApiServerUri, "CreateFileInfos",
		            String.Format(CultureInfo.InvariantCulture, "assetid='{0}'", assetId), acsBearerToken, null);
		
		        using (HttpWebResponse response = (HttpWebResponse)request.GetResponse())
		        {
		            if (response.StatusCode == HttpStatusCode.MovedPermanently)
		            {
		                string redirectedMediaServicesApiUrl = response.Headers["Location"];
		
		                CreateFileInfos(redirectedMediaServicesApiUrl, acsBearerToken, assetId);
		            }
		            else if ((response.StatusCode != HttpStatusCode.OK) &&
		                (response.StatusCode != HttpStatusCode.Accepted) &&
		                (response.StatusCode != HttpStatusCode.Created) &&
		                (response.StatusCode != HttpStatusCode.NoContent))
		            {
		                // TODO: Throw a more specific exception.
		                throw new Exception("Invalid response received ");
		            }
		        }
		    }
		    catch (WebException ex)
		    {
		        Console.WriteLine(ex.Message);
		    }
		}
		
		
		private static HttpWebRequest GenerateRequest(string verb,
		                                                string mediaServicesApiServerUri,
		                                                string resourcePath, string query,
		                                                string acsBearerToken, string requestbody)
		{
		    var uriBuilder = new UriBuilder(mediaServicesApiServerUri);
		    uriBuilder.Path += resourcePath;
		    if (query != null)
		    {
		        uriBuilder.Query = query;
		    }
		    HttpWebRequest request = (HttpWebRequest)HttpWebRequest.Create(uriBuilder.Uri);
		    request.AllowAutoRedirect = false; //We manage our own redirects.
		    request.Method = verb;
		
		    if (resourcePath == "$metadata")
		        request.MediaType = "application/xml";
		    else
		    {
		        request.ContentType = "application/json;odata=verbose";
		        request.Accept = "application/json;odata=verbose";
		    }
		
		    request.Headers.Add("DataServiceVersion", "3.0");
		    request.Headers.Add("MaxDataServiceVersion", "3.0");
		    request.Headers.Add("x-ms-version", "2.1");
		    request.Headers.Add(HttpRequestHeader.Authorization, "Bearer " + acsBearerToken);
		
		    if (requestbody != null)
		    {
		        var requestBytes = Encoding.ASCII.GetBytes(requestbody);
		        request.ContentLength = requestBytes.Length;
		
		        var requestStream = request.GetRequestStream();
		        requestStream.Write(requestBytes, 0, requestBytes.Length);
		        requestStream.Close();
		    }
		    else
		    {
		        request.ContentLength = 0;
		    }
		    return request;
		}
		

##后续步骤

现在，你可以使用流量管理器在两个数据中心之间路由请求，因此可在任何中断情况下进行故障转移。

<!---HONumber=Mooncake_0620_2016-->