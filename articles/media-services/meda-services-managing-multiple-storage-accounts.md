<properties 
	pageTitle="跨多个存储帐户管理媒体服务资产" 
	description="本文提供有关如何跨多个存储帐户管理媒体服务资产的指导。" 
	services="media-services" 
	documentationCenter="" 
	authors="Juliako" 
	manager="dwrede" 
	editor=""/>

<tags
	ms.service="media-services"
	ms.date="04/18/2016"    
	wacn.date="06/20/2016"/>


#跨多个存储帐户管理媒体服务资产

从 Azure媒体服务2.2 开始，可以将多个存储帐户附加到一个媒体服务帐户。将多个存储帐户附加到一个媒体服务帐户这一功能具有以下优势：

- 使多个存储帐户之间的资产实现负载平衡。
- 缩放媒体服务以处理大量内容（目前，单个存储帐户的上限为 500 TB）。 

本主题演示如何使用 Azure 服务管理 REST API 将多个存储帐户附加到一个媒体服务帐户，此外还说明如何在使用媒体服务 SDK 创建资产时指定不同的存储帐户。

##注意事项

将多个存储帐户附加到媒体服务帐户时，请注意以下事项：

- 附加到媒体服务帐户的所有存储帐户必须与媒体服务帐户位于同一数据中心。
- 目前，存储帐户一旦附加到指定的媒体服务帐户便无法断开。
- 主存储帐户是在创建媒体服务帐户创建时指定的帐户。目前，你无法更改默认存储帐户。 

其他注意事项：

构建流内容的 URL 时，媒体服务会使用 **IAssetFile.Name** 属性的值（如 http://{WAMSAccount}.origin.mediaservices.chinacloudapi.cn/{GUID}/{IAssetFile.Name}/streamingParameters.）。出于这个原因，不允许使用百分号编码。Name 属性的值不能含有任何以下保留的[百分号编码字符](http://zh.wikipedia.org/wiki/百分号编码#.E4.BF.9D.E7.95.99.E5.AD.97.E7.AC.A6.E7.9A.84.E7.99.BE.E5.88.86.E5.8F.B7.E7.BC.96.E7.A0.81)：!*'();:@&=+$,/?%#"。此外，文件扩展名中只能含有一个“.”。

##使用 Azure 服务管理 REST API 附加存储帐户

目前，只能使用 [Azure 服务管理 REST API](http://msdn.microsoft.com/zh-cn/library/azure/dn167014.aspx) 附加多个存储帐户。[如何：使用媒体服务管理 REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn167656.aspx) 主题中的代码示例定义了将存储帐户附加到指定媒体服务帐户的 **AttachStorageAccountToMediaServiceAccount** 方法。此主题中的代码定义了列出已附加到指定媒体服务帐户的所有存储帐户的 **ListStorageAccountDetails** 方法。


##跨多个存储帐户管理媒体服务资产

以下代码使用最新的媒体服务 SDK 执行下列任务：

1. 显示与指定媒体服务帐户关联的所有存储帐户。
1. 检索默认存储帐户的名称。
1. 在默认存储帐户中创建一个新资产。
1. 在指定存储帐户中创建编码作业的输出资产。
	
		using Microsoft.WindowsAzure.MediaServices.Client;
		using System;
		using System.Collections.Generic;
		using System.Configuration;
		using System.IO;
		using System.Linq;
		using System.Text;
		using System.Threading;
		using System.Threading.Tasks;
		
		namespace MultipleStorageAccounts
		{
		    class Program
		    {
		        // Location of the media file that you want to encode. 
		        private static readonly string _singleInputFilePath =
		            Path.GetFullPath(@"../..\supportFiles\multifile\interview2.wmv");
		
		        private static readonly string MediaServicesAccountName = 
		            ConfigurationManager.AppSettings["MediaServicesAccountName"];
		        private static readonly string MediaServicesAccountKey = 
		            ConfigurationManager.AppSettings["MediaServicesAccountKey"];
		
				private static readonly String _defaultScope = "urn:WindowsAzureMediaServices";

				// Azure China uses a different API server and a different ACS Base Address from the Global.
				private static readonly String _chinaApiServerUrl = "https://wamsshaclus001rest-hs.chinacloudapp.cn/API/";
				private static readonly String _chinaAcsBaseAddressUrl = "https://wamsprodglobal001acs.accesscontrol.chinacloudapi.cn";

		        private static CloudMediaContext _context;
		        private static MediaServicesCredentials _cachedCredentials = null;
	
		        static void Main(string[] args)
		        {
	
		            // Create and cache the Media Services credentials in a static class variable.
		            _cachedCredentials = new MediaServicesCredentials(
		                            MediaServicesAccountName,
		                            MediaServicesAccountKey,
									_defaultScope,
									_chinaAcsBaseAddressUrl);

					// Create the API server Uri
					_apiServer = new Uri(_chinaApiServerUrl);

		            // Used the cached credentials to create CloudMediaContext.
		            _context = new CloudMediaContext(_apiServer, _cachedCredentials);
	
		
		            // Display the storage accounts associated with 
		            // the specified Media Services account:
		            foreach (var sa in _context.StorageAccounts)
		                Console.WriteLine(sa.Name);
		
		            // Retrieve the name of the default storage account.
		            var defaultStorageName = _context.StorageAccounts.Where(s => s.IsDefault == true).FirstOrDefault();
		            Console.WriteLine("Name: {0}", defaultStorageName.Name);
		            Console.WriteLine("IsDefault: {0}", defaultStorageName.IsDefault);
		
		            // Retrieve the name of a storage account that is not the default one.
		            var notDefaultStroageName = _context.StorageAccounts.Where(s => s.IsDefault == false).FirstOrDefault();
		            Console.WriteLine("Name: {0}", notDefaultStroageName.Name);
		            Console.WriteLine("IsDefault: {0}", notDefaultStroageName.IsDefault);
		            
		            // Create the original asset in the default storage account.
		            IAsset asset = CreateAssetAndUploadSingleFile(AssetCreationOptions.None, 
		                defaultStorageName.Name, _singleInputFilePath);
		            Console.WriteLine("Created the asset in the {0} storage account", asset.StorageAccountName);
		            
		            // Create an output asset of the encoding job in the other storage account.
		            IAsset outputAsset = CreateEncodingJob(asset, notDefaultStroageName.Name, _singleInputFilePath);
		            if(outputAsset!=null)
		                Console.WriteLine("Created the output asset in the {0} storage account", outputAsset.StorageAccountName);
		
		        }
		
		        static public IAsset CreateAssetAndUploadSingleFile(AssetCreationOptions assetCreationOptions, string storageName, string singleFilePath)
		        {
		            var assetName = "UploadSingleFile_" + DateTime.UtcNow.ToString();
		            
		            // If you are creating an asset in the default storage account, you can omit the StorageName parameter.
		            var asset = _context.Assets.Create(assetName, storageName, assetCreationOptions);
		
		            var fileName = Path.GetFileName(singleFilePath);
		
		            var assetFile = asset.AssetFiles.Create(fileName);
		
		            Console.WriteLine("Created assetFile {0}", assetFile.Name);
		
		            assetFile.Upload(singleFilePath);
		            
		            Console.WriteLine("Done uploading {0}", assetFile.Name);
		
		            return asset;
		        }
		
		        static IAsset CreateEncodingJob(IAsset asset, string storageName, string inputMediaFilePath)
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
		            task.OutputAssets.AddNew("Output asset", storageName,
		                AssetCreationOptions.None);
		
		            // Use the following event handler to check job progress.  
		            job.StateChanged += new
		                    EventHandler<JobStateChangedEventArgs>(StateChanged);
		
		            // Launch the job.
		            job.Submit();
		
		            // Check job execution and wait for job to finish. 
		            Task progressJobTask = job.GetExecutionProgressTask(CancellationToken.None);
		            progressJobTask.Wait();
		
		            // Get an updated job reference.
		            job = GetJob(job.Id);
		
		            // If job state is Error the event handling 
		            // method for job progress should log errors.  Here we check 
		            // for error state and exit if needed.
		            if (job.State == JobState.Error)
		            {
		                Console.WriteLine("\nExiting method due to job error.");
		                return null;
		            }
		
		            // Get a reference to the output asset from the job.
		            IAsset outputAsset = job.OutputMediaAssets[0];
		
		            return outputAsset;
		        }
		
		
		        private static IMediaProcessor GetLatestMediaProcessorByName(string mediaProcessorName)
		        {
		            var processor = _context.MediaProcessors.Where(p => p.Name == mediaProcessorName).
		                ToList().OrderBy(p => new Version(p.Version)).LastOrDefault();
		
		            if (processor == null)
		                throw new ArgumentException(string.Format("Unknown media processor", mediaProcessorName));
		
		            return processor;
		        }
		
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
		                    Console.WriteLine("An error occurred in {0}", job.Id);
		                    break;
		                default:
		                    break;
		            }
		        }
		
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
		    }
		}
 


<!---HONumber=Mooncake_0613_2016-->